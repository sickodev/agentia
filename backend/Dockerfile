# ==============================================================================
# Build Stage
# ==============================================================================
FROM maven:3.9-eclipse-temurin-21-alpine AS builder
WORKDIR /app

COPY pom.xml .
RUN mvn dependency:go-offline -B || true

COPY src ./src
RUN mvn clean package -DskipTests

# ==============================================================================
# Runtime Stage
# ==============================================================================
FROM eclipse-temurin:21-jre-alpine
WORKDIR /app

RUN mkdir -p /app/data
VOLUME /app/data

COPY --from=builder /app/target/*.jar app.jar

ENV SPRING_DATASOURCE_URL=jdbc:sqlite:/app/data/agentville.db
EXPOSE 8080

ENTRYPOINT ["java", "-XX:+UseZGC", "-XX:+ZGenerational", "-jar", "app.jar"]
