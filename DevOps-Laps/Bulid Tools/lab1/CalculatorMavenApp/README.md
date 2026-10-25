# Lab 1: Building and Packaging a Java Application with Maven

**Source code:** https://github.com/Ibrahim-Adel15/CalculatorMavenApp.git

## 1. Install Java and Maven

```bash
sudo apt update
sudo apt install -y openjdk-17-jdk maven

java -version
mvn -version
```

## 2. Clone the source code

```bash
git clone https://github.com/Ibrahim-Adel15/CalculatorMavenApp.git
cd CalculatorMavenApp
```

## 3. Run unit tests

```bash
mvn test
```

## 4. Build the app

```bash
mvn package
```

Artifact generated at: `target/calculator.jar`

## 5. Run the app

```bash
java -jar target/calculator.jar
```

## 6. Verify the app is working

- Check that the app starts without errors and responds correctly to input.
- Confirm the artifact exists:

```bash
ls -lh target/calculator.jar
```
