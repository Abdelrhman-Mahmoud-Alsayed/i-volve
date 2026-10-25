# Lab 5: Docker Image Vulnerability Scanning with Trivy

**Source code:** https://github.com/Ibrahim-Adel15/TrivyImage.git

## 1. Clone the application and Dockerfile

```bash
git clone https://github.com/Ibrahim-Adel15/TrivyImage.git
cd TrivyImage
```

## 2. Build the image

```bash
docker build -t trivy-lab:v1 .
docker images trivy-lab:v1
```

## 3. Install Trivy and verify

```bash
sudo apt-get install -y wget gnupg
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | gpg --dearmor | sudo tee /usr/share/keyrings/trivy.gpg > /dev/null
echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb generic main" | sudo tee /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install -y trivy

trivy --version
```

## 4. Scan the image for vulnerabilities

```bash
trivy image trivy-lab:v1
```

## 5. Scan for HIGH and CRITICAL only

```bash
trivy image --severity HIGH,CRITICAL trivy-lab:v1
```

## 6. Generate an HTML report

```bash
trivy image \
  --severity HIGH,CRITICAL \
  --format template \
  --template "@/usr/local/share/trivy/templates/html.tpl" \
  -o report-v1.html \
  trivy-lab:v1
```

Open `report-v1.html` in a browser and review the findings.

> If the template path is different on your system, find it with: `find / -name html.tpl 2>/dev/null`

---

## Bonus: Vulnerability Remediation

### 1. Investigate HIGH and CRITICAL vulnerabilities

For each finding in the report, note:

- **Package / library** affected
- **Installed version** and **Fixed version**
- Whether it comes from the **base image (OS)** or from **application dependencies**

```bash
# Show only vulnerabilities that already have a fix
trivy image --severity HIGH,CRITICAL --ignore-unfixed trivy-lab:v1
```

### 2. Fix the issues

- **Update the base image** in the Dockerfile to a newer, patched version (for example a newer `-slim` or `alpine` tag).
- **Update vulnerable dependencies** (for example in `requirements.txt`, `package.json`, or `pom.xml`) to the fixed versions shown in the report.

Example Dockerfile change:

```dockerfile
# Before
FROM python:3.8

# After
FROM python:3.12-slim
```

### 3. Rebuild and tag as ivolve-app:v2

```bash
docker build -t ivolve-app:v2 .
```

### 4. Rescan and generate a new HTML report

```bash
trivy image \
  --severity HIGH,CRITICAL \
  --format template \
  --template "@/usr/local/share/trivy/templates/html.tpl" \
  -o report-v2.html \
  ivolve-app:v2
```

### 5. Confirm no fixable HIGH or CRITICAL vulnerabilities remain

```bash
trivy image --severity HIGH,CRITICAL --ignore-unfixed --exit-code 1 ivolve-app:v2
echo $?     # 0 means no fixable HIGH/CRITICAL vulnerabilities
```

### Before vs After

| Image | HIGH | CRITICAL |
|---|---|---|
| `trivy-lab:v1` | _fill in_ | _fill in_ |
| `ivolve-app:v2` | _fill in_ | _fill in_ |
