FROM tomcat:9.0-jdk11

# 移除默认应用
RUN rm -rf /usr/local/tomcat/webapps/*

# 复制你的应用
COPY *.jar /usr/local/tomcat/webapps/ROOT.war

# 可选：复制自定义配置
# COPY config/server.xml /usr/local/tomcat/conf/server.xml
# COPY config/context.xml /usr/local/tomcat/conf/context.xml

# 设置环境变量
ENV JAVA_OPTS="-Djava.security.egd=file:/dev/./urandom -Djava.awt.headless=true -Xmx512m -XX:MaxPermSize=256m"

EXPOSE 8080
CMD ["catalina.sh", "run"]
