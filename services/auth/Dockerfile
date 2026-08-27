# syntax=docker/dockerfile:1.7
# LieShouCloud · Auth Service (port 8083) multi-stage
# docker build context = 仓库根(含 framework submodule + services/,mvn -f pom.xml 一次聚合构建)

FROM maven:3.9.9-eclipse-temurin-21 AS build
WORKDIR /workspace

# 拷贝整个仓库根(含 framework/),maven 从根 pom 聚合构建
COPY . /workspace/
# 阿里云 Maven 镜像加速 (./services/.m2-settings.xml)
COPY services/.m2-settings.xml /root/.m2/settings.xml

RUN --mount=type=cache,target=/root/.m2 \
    mvn -B -ntp -Dmaven.test.skip=true -f pom.xml -pl services/auth -am clean package

FROM eclipse-temurin:21-jre-alpine AS runtime
RUN addgroup -S app && adduser -S app -G app
WORKDIR /app

COPY --from=build /workspace/services/auth/target/lieshoucloud-auth.jar /app/app.jar

ENV JAVA_TOOL_OPTIONS="-XX:MaxRAMPercentage=75.0 -XX:+UseG1GC -Dfile.encoding=UTF-8"
USER app

EXPOSE 8083

HEALTHCHECK --interval=30s --timeout=5s --start-period=40s --retries=3 \
    CMD wget -Y off -qO- http://127.0.0.1:8083/actuator/health >/dev/null || exit 1

ENTRYPOINT ["java", "-jar", "/app/app.jar"]
