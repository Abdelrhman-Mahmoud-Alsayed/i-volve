# Lab 3: Creating a Docker Image for a Java Application

**Source code:** https://github.com/Ibrahim-Adel15/CalculatorMavenApp.git

```bash
git clone https://github.com/Ibrahim-Adel15/CalculatorMavenApp.git
cd CalculatorMavenApp
```

## Part 1: Build Inside Docker

The application is built **inside** the Docker image using a Maven image with Java 17.

**Dockerfile**

```dockerfile
FROM maven:3.9-eclipse-temurin-17

# Working directory and non-root user
RUN useradd -m appuser
WORKDIR /app

# Copy source code
COPY . .
RUN chown -R appuser:appuser /app
USER appuser

# Build the application
RUN mvn package

EXPOSE 8080

# Run the generated JAR
CMD ["java", "-jar", "target/calculator.jar"]
```

**Build, run, verify, and clean up**

```bash
# Build image named app1
docker build -t app1 .

# Note the image size
docker images app1

# Run container1
docker run -d --name container1 -p 8080:8080 app1

# Verify the application
docker ps
docker logs container1

# Delete the container and the image
docker rm -f container1
docker rmi app1
```

## Part 2: Build Locally

The application is built **on the host machine** with Maven, then only the JAR is copied into a lightweight Java 17 runtime image.

**Step 1: Build locally**

```bash
mvn package
```

**Step 2: Dockerfile**

```dockerfile
FROM eclipse-temurin:17-jre

# Working directory and non-root user
RUN useradd -m appuser
WORKDIR /app

# Copy the pre-built JAR
COPY target/calculator.jar app.jar
RUN chown appuser:appuser app.jar
USER appuser

EXPOSE 8080

CMD ["java", "-jar", "app.jar"]
```

**Step 3: Build, run, verify, and clean up**

```bash
# Build image named app2
docker build -t app2 .

# Note the image size
docker images app2

# Run container2
docker run -d --name container2 -p 8080:8080 app2

# Verify the application
docker ps
docker logs container2

# Delete the container and the image
docker rm -f container2
docker rmi app2
```

> If the app is a console application that waits for input, run it with `docker run -it` instead of `-d`.

## Verify the non-root user

```bash
docker run --rm app1 whoami
docker inspect app1 --format '{{.Config.User}}'
```

## Bonus: Image Size Comparison

| | `app1` (Build inside Docker) | `app2` (Build locally) |
|---|---|---|
| **Base image** | `maven:3.9-eclipse-temurin-17` | `eclipse-temurin:17-jre` |
| **Contains** | Maven, full JDK, source code, `.m2` dependencies, JAR | JRE and JAR only |
| **Approx. size** | ~500-700 MB | ~200-250 MB |
| **Needs Maven on host?** | No | Yes |
| **Security surface** | Larger (compilers, build tools, source) | Smaller |

**Explanation**

- **Build inside Docker:** the build is reproducible and needs nothing but Docker on the host, but the final image carries all the build tools and downloaded dependencies, making it large.
- **Build locally:** the image holds only the runtime and the JAR, so it is smaller, faster to pull and deploy, and more secure, but it depends on Maven and Java being installed on the host.

> Tip: a **multi-stage build** gives the best of both worlds: build inside Docker, then copy only the JAR into a JRE image.
