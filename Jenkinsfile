pipeline {
    agent any
    environment {
        ACR_DOMAIN = 'crpi-k5gg1rbjrocgkfs3.cn-shenzhen.personal.cr.aliyuncs.com'
        NAMESPACE  = 'byl-test'
        IMAGE_NAME = 'maven-demo'
        TAG        = "${env.BUILD_ID}"
        FULL_IMAGE = "${ACR_DOMAIN}/${NAMESPACE}/${IMAGE_NAME}:${TAG}"   // 拼一次后面复用
        HOST       = '172.16.88.141'
        USER       = 'root'
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('IdeaProjects/untitled/') {
            steps {
                dir('maven-demo') {
                    sh '''
                      export MAVEN_HOME=/opt/maven/apache-maven-3.8.8
                      export JAVA_HOME=/opt/jdk/jdk-23
                      export PATH=$MAVEN_HOME/bin:$JAVA_HOME/bin:$PATH
                      
                      mvn clean package -DskipTests
                    '''
                }
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    // --no-cache 保证每次都重新 COPY 最新的 jar
                    def img = docker.build("${FULL_IMAGE}", "--no-cache .")
                }
            }
        }

        stage('Push to ACR') {
            steps {
                script {
                    docker.withRegistry("https://${ACR_DOMAIN}", 'ACR_PASSWD') {
                        docker.image("${FULL_IMAGE}").push()
                        docker.image("${FULL_IMAGE}").push('latest')
                    }
                }
            }
        }

        stage('Deploy on 141') {
            steps {
                sshagent(['vm141-ssh']) {
                    withCredentials([usernamePassword(credentialsId: 'ACR_PASSWD',
                                                      usernameVariable: 'ACR_USER',
                                                      passwordVariable: 'ACR_PWD')]) {
                        sh """
                           ssh -o StrictHostKeyChecking=no ${USER}@${HOST} \
                              "echo ${ACR_PWD} | docker login --username=${ACR_USER} --password-stdin ${ACR_DOMAIN}"
                           ssh ${USER}@${HOST} "docker stop ${IMAGE_NAME} || true"
                           ssh ${USER}@${HOST} "docker rm   ${IMAGE_NAME} || true"
                           ssh ${USER}@${HOST} "docker pull ${FULL_IMAGE}"
                           ssh ${USER}@${HOST} "docker run -d --restart=always -p 8080:8080 --name ${IMAGE_NAME} ${FULL_IMAGE}"
                        """
                    }
                }
            }
        }
    }
    post {
        always {
            // 可选：构建完删除本地镜像，省磁盘
            sh "docker rmi ${FULL_IMAGE} ${ACR_DOMAIN}/${NAMESPACE}/${IMAGE_NAME}:latest || true"
        }
    }
}
