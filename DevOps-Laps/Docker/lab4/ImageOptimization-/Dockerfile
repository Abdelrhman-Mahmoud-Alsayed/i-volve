FROM maven:3.9.15-eclipse-temurin-17-alpine AS builder 
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN mvn package

FROM eclipse-temurin:17-jre-alpine
RUN adduser -D node
WORKDIR /app
RUN chown node:node /app
USER node
COPY --from=builder /app/target/demo-0.0.1-SNAPSHOT.jar app.jar
USER node
EXPOSE 8080
CMD ["java", "-jar", "target/demo-0.0.1-SNAPSHOT.jar"]
