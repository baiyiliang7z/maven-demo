# 使用适合你的JDK版本的基础镜像
FROM eclipse-temurin:11-jdk

# 创建工作目录
WORKDIR /app

# 复制JAR文件到容器中（使用通配符匹配版本号）
COPY target/*.jar app.jar

# 暴露端口（根据你的应用配置）
EXPOSE 8080

# 设置JVM参数（根据需求调整）
ENV JAVA_OPTS="-Xmx512m -Xms256m"

# 启动命令
ENTRYPOINT ["sh", "-c", "java ${JAVA_OPTS} -jar app.jar"]
