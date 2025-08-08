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

        stage('mvn clean') {
            steps {
                dir('IdeaProjects/untitled') {
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
                dir('IdeaProjects/untitled') {
                    script {
                        // --no-cache 保证每次都重新 COPY 最新的 jar
                        def img = docker.build("${FULL_IMAGE}", "--no-cache .")
                    }
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
                script {                                  // ← 必须包一层 script
                    sshagent(['deploy-root-key']) {
                        withCredentials([usernamePassword(credentialsId: 'ACR_PASSWD',
                                                          usernameVariable: 'ACR_USER',
                                                          passwordVariable: 'ACR_PWD')]) {
                            sh """
                               ssh -o StrictHostKeyChecking=no root@172.16.88.141 \
                                  "echo ${ACR_PWD} | docker login --username=${ACR_USER} --password-stdin crpi-k5gg1rbjrocgkfs3.cn-shenzhen.personal.cr.aliyuncs.com"
                               ssh root@172.16.88.141 "docker stop maven-demo || true"
                               ssh root@172.16.88.141 "docker rm   maven-demo || true"
                               ssh root@172.16.88.141 "docker pull ${FULL_IMAGE}"
                               ssh root@172.16.88.141 "docker run -d --restart=always -p 8080:8080 --name maven-demo ${FULL_IMAGE}"
                            """
                        }
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
