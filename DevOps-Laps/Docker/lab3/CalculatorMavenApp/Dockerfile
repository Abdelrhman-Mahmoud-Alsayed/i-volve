
FROM eclipse-temurin:17-jre

RUN useradd -m node

WORKDIR /app

COPY  target/calculator.jar .

RUN chown  node:node /app
USER node

CMD ["java","-jar","target/calculator.jar"] 

EXPOSE 8080

