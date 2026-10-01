# Stage 1: Build mã nguồn với JDK
FROM eclipse-temurin:17-jdk-alpine AS builder
WORKDIR /app
COPY . .
RUN chmod +x ./gradlew
RUN ./gradlew bootJar

# Stage 2: Chạy ứng dụng với JRE siêu nhẹ
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY --from=builder /app/build/libs/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
