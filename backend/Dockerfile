# --- Build stage ---
FROM amazoncorretto:21 AS build
WORKDIR /workspace

# Gradle 캐시 최적화: wrapper/gradle 관련 파일을 먼저 복사
COPY gradlew gradlew
COPY gradle gradle
COPY build.gradle settings.gradle gradle.properties* ./
# 소스는 마지막에 복사 (캐시 활용)
RUN sed -i 's/\r$//' gradlew && chmod +x gradlew

# 🔥 Gradle 캐시 마운트 (wrapper / caches 분리 권장)
RUN --mount=type=cache,target=/root/.gradle/wrapper \
    --mount=type=cache,target=/root/.gradle/caches \
    ./gradlew --no-daemon dependencies --stacktrace --info

COPY src src

# 빌드
RUN --mount=type=cache,target=/root/.gradle/wrapper \
    --mount=type=cache,target=/root/.gradle/caches \
    ./gradlew --no-daemon -x test bootJar


# --- Run stage ---
FROM amazoncorretto:21-alpine AS runtime
WORKDIR /app
ENV TZ=Asia/Seoul
ENV JAVA_OPTS="-XX:MaxRAMPercentage=75 -XX:+UseG1GC"

# ---- 최종 이미지 1: 도커-내부 빌드 사용 ----
FROM runtime AS image-from-build
COPY --from=build /workspace/build/libs/*.jar /app/app.jar
ENTRYPOINT ["java","-jar","/app/app.jar"]

# ---- 최종 이미지 2: 외부(JAR) 포장 전용 ----
FROM runtime AS image-from-jar
# CI에서 ./gradlew -x test bootJar 로 만든 산출물을 사용
COPY build/libs/*.jar /app/app.jar
ENTRYPOINT ["java","-jar","/app/app.jar"]
