# Build stage
FROM maven:3.9-eclipse-temurin-25-alpine AS build
WORKDIR /app

COPY pom.xml .
RUN mvn dependency:go-offline -B

COPY src ./src
RUN mvn clean package

# Runtime stage
FROM eclipse-temurin:25-jre-alpine
WORKDIR /app

COPY --from=build /app/target/*.jar app.jar

EXPOSE ${API_PORT}
ENTRYPOINT ["java", "-jar", "/app/app.jar"]
