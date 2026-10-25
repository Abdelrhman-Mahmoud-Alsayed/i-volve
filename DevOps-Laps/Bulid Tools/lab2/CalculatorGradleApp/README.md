# Lab 2: Building and Packaging an Application with Gradle

**Source code:** https://github.com/Ibrahim-Adel15/CalculatorGradleApp.git

## 1. Install Gradle

```bash
sudo apt update
sudo apt install -y openjdk-17-jdk gradle

gradle -version
```

> Alternative: install via [SDKMAN](https://sdkman.io/) with `sdk install gradle`.

## 2. Clone the source code

```bash
git clone https://github.com/Ibrahim-Adel15/CalculatorGradleApp.git
cd CalculatorGradleApp
```

## 3. Run unit tests

```bash
gradle test
```

## 4. Build the app

```bash
gradle build
```

Artifact generated at: `build/libs/calculator.jar`

## 5. Run the app

```bash
java -jar build/libs/calculator.jar
```

## 6. Verify the app is working

- Check that the app starts without errors and responds correctly to input.
- Confirm the artifact exists:

```bash
ls -lh build/libs/calculator.jar
```
