# Lab 4: Docker Image Optimization and Export/Load

**Source code:** https://github.com/Ibrahim-Adel15/ImageOptimization-.git

## 1. Clone and build the original image

```bash
git clone https://github.com/Ibrahim-Adel15/ImageOptimization-.git
cd ImageOptimization-

# Build from the existing Dockerfile
docker build -t image1 .
```

## 2. Note the image size and run container1

```bash
docker images image1

docker run -d --name container1 -p 5000:5000 image1
```

> Adjust the port to the one the application uses (check `EXPOSE` in the Dockerfile).

## 3. Verify, then stop and delete the container

```bash
docker ps
docker logs container1
curl http://localhost:5000

docker stop container1
docker rm container1
```

## 4. Write a multi-stage Dockerfile (non-root user)

Save as `Dockerfile` (or `Dockerfile.multi` and build with `-f`).

> This example assumes a Python app. Adapt the base images, build commands, and start command to the actual application in the repo.

```dockerfile
# ---------- Stage 1: Builder ----------
FROM python:3.12-slim AS builder
WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir --prefix=/install -r requirements.txt

# ---------- Stage 2: Runtime ----------
FROM python:3.12-slim
WORKDIR /app

# Create non-root user
RUN useradd -m appuser

# Copy only the installed dependencies and the app code
COPY --from=builder /install /usr/local
COPY --chown=appuser:appuser . .

USER appuser

EXPOSE 5000
CMD ["python", "app.py"]
```

## 5. Build the optimized image and compare sizes

```bash
docker build -t image2 .

docker images | grep -E "image1|image2"
```

| Image | Description | Size |
|---|---|---|
| `image1` | Original single-stage Dockerfile | _fill in_ |
| `image2` | Multi-stage, non-root | _fill in_ |

## 6. Run container2 and verify (running as non-root)

```bash
docker run -d --name container2 -p 5000:5000 image2

docker ps
curl http://localhost:5000

# Verify non-root user
docker exec container2 whoami
docker exec container2 id
```

Expected output of `whoami`: `appuser`

## 7. Stop and delete container2

```bash
docker stop container2
docker rm container2
```

## 8. Save image2 as a TAR file

```bash
docker save -o lab-image.tar image2
ls -lh lab-image.tar
```

## 9. Delete image2 from the local host

```bash
docker rmi image2
docker images | grep image2     # should return nothing
```

## 10. Load the image from the TAR file

```bash
docker load -i lab-image.tar
```

## 11. Verify the loaded image exists

```bash
docker images image2
```

> Note: the lab text mentions `image2.tar` in the load step, but the file was saved as `lab-image.tar` earlier. Use the same file name in both `save` and `load`.
