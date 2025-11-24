# Multi-stage build for Authorization Microservice
FROM maven:3.8-openjdk-8 AS build

WORKDIR /app

# Copy pom.xml and download dependencies (cache layer)
COPY pom.xml .
RUN mvn dependency:go-offline -B

# Copy source code and build
COPY src ./src
RUN mvn clean package -DskipTests

# Production image
FROM openjdk:8-jre-slim

WORKDIR /app

# Copy JAR from build stage
COPY --from=build /app/target/*.jar app.jar

# Create non-root user for security
RUN groupadd -r appuser && useradd -r -g appuser appuser
RUN chown -R appuser:appuser /app
USER appuser

# Expose port
EXPOSE 8400

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=60s --retries=3 \
  CMD curl -f http://localhost:8400/auth/actuator/health || exit 1

# Run with AWS profile
ENTRYPOINT ["java", "-Dspring.profiles.active=aws", "-Xmx512m", "-Xms256m", "-jar", "app.jar"]
