# Lab 6: Docker Environment Variables and Resource Management

**Source code:** https://github.com/Ibrahim-Adel15/PythonApp.git

## 1. Clone the application

```bash
git clone https://github.com/Ibrahim-Adel15/PythonApp.git
cd PythonApp
```

## 2. Create the Dockerfile

```dockerfile
FROM python:3.12-slim

WORKDIR /app

# Install dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

EXPOSE 5000

CMD ["python", "app.py"]
```

> Adjust the file name in `CMD` to the app's entry point. The app must listen on `0.0.0.0:5000` to be reachable from outside the container.

## 3. Build the image `ivolve1`

```bash
docker build -t ivolve1 .
```

## 4. Run container1 with environment variables and resource limits

```bash
docker run -d \
  --name container1 \
  -p 5000:5000 \
  -e APP_NAME=iVolveApp1 \
  -e APP_ENV=stg \
  --cpus="1" \
  --memory="256m" \
  ivolve1
```

Verify the limits:

```bash
docker inspect container1 --format 'CPUs: {{.HostConfig.NanoCpus}}  Memory: {{.HostConfig.Memory}}'
```

(`1000000000` NanoCpus = 1 CPU, `268435456` bytes = 256 MB)

## 5. Verify the application is working

```bash
curl http://localhost:5000
```

Or open `http://localhost:5000` in a browser.

## 6. Verify the environment variables inside the container

```bash
docker exec container1 env | grep APP_
```

Expected:

```
APP_NAME=iVolveApp1
APP_ENV=stg
```

## 7. Monitor CPU and memory usage

```bash
docker stats container1
```

Use `docker stats --no-stream container1` for a single snapshot. The memory limit should show as 256MiB.

## 8. Create `env.txt` and run container2 using it

```bash
cat > env.txt << 'ENVEOF'
APP_NAME=iVolveApp2
APP_ENV=prod
ENVEOF
```

```bash
docker run -d \
  --name container2 \
  -p 5001:5000 \
  --env-file env.txt \
  ivolve1
```

> Port `5000` on the host is already used by container1, so container2 is mapped to host port `5001`. Alternatively, stop container1 first and use `-p 5000:5000`.

Verify:

```bash
docker exec container2 env | grep APP_
curl http://localhost:5001
```

Expected:

```
APP_NAME=iVolveApp2
APP_ENV=prod
```

## 9. Create a new image with hardcoded environment variables

Create a new Dockerfile (for example `Dockerfile.env`):

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

# Hardcoded environment variables
ENV APP_NAME=iVolveApp3
ENV APP_ENV=dev

EXPOSE 5000

CMD ["python", "app.py"]
```

Build it:

```bash
docker build -f Dockerfile.env -t ivolve2 .
```

## 10. Run a container and validate the variables from the Dockerfile

The name `container2` is already used, so remove the old one first:

```bash
docker rm -f container2

docker run -d \
  --name container2 \
  -p 5000:5000 \
  ivolve2
```

> If container1 is still running on port 5000, stop it first: `docker rm -f container1`, or map a different host port (for example `-p 5002:5000`).

Validate:

```bash
docker exec container2 env | grep APP_
curl http://localhost:5000
```

Expected:

```
APP_NAME=iVolveApp3
APP_ENV=dev
```

## Cleanup

```bash
docker rm -f container1 container2
docker rmi ivolve1 ivolve2
rm env.txt
```

## Summary: Ways to set environment variables

| Method | Command / Instruction | Use case |
|---|---|---|
| Inline | `docker run -e KEY=value` | Quick, few variables |
| From file | `docker run --env-file env.txt` | Many variables, per-environment config |
| In Dockerfile | `ENV KEY=value` | Default values baked into the image |

Variables passed with `-e` or `--env-file` **override** values set with `ENV` in the Dockerfile.
