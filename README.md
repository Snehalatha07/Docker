# Docker

## Multistage dockerfile

```dockerfile
# -------- Build Stage --------
FROM maven:3.9.6-eclipse-temurin-17 AS builder

WORKDIR /app

COPY pom.xml .
COPY src ./src

RUN mvn clean package -DskipTests

# -------- Run Stage --------
FROM eclipse-temurin:17-jdk-jammy

WORKDIR /app

COPY --from=builder /app/target/*.jar app.jar

CMD ["java", "-jar", "app.jar"]
```

##Project structure
```text
java-multistage-app/
│
├── pom.xml
└── src/
    └── main/
        └── java/
            └── com/
                └── example/
                    └── App.java
```

  ## pom.xml
  ```xml
  <project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         http://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>java-multistage-app</artifactId>
    <version>1.0</version>

    <!-- Spring Boot Parent -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.0</version>
        <relativePath/>
    </parent>

    <properties>
        <java.version>17</java.version>
    </properties>

    <!-- ✅ Dependencies MUST be inside project -->
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <!-- Spring Boot Plugin (IMPORTANT) -->
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

##App.Java
```dockerfile
package com.example;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.*;

@SpringBootApplication
@RestController
public class App {

    public static void main(String[] args) {
        SpringApplication.run(App.class, args);
    }

    @GetMapping("/")
    public String home() {
        return "Hello from Spring Boot Docker!";
    }
}
```

## Build & Run
```text
docker build -t java-maven-app .
docker run java-maven-app
```
#Docker
##Output
```text
Hello from Maven Multi-Stage Docker Build!
```

## Important Understanding (Interview)

> 👉 **What happens internally?**
>
> **Stage 1 (Maven image)**
> - Downloads dependencies
> - Runs `mvn clean package`
> - Creates `.jar` file inside `/target`
>
> **Stage 2 (Lightweight JDK image)**
> - Copies only the `.jar`
> - Runs the application

## Single-Stage Dockerfile (Java Maven)
```text
FROM maven:3.9.6-eclipse-temurin-17

WORKDIR /app

# Copy project files
COPY pom.xml .
COPY src ./src

# Build the application
RUN mvn clean package -DskipTests

# Run the JAR
CMD ["java", "-jar", "target/java-multistage-app-1.0.jar"]
```

## Build & Run
```text
docker build -t java-single-stage .
docker run java-single-stage
```

##project structure
```text
java-multistage-app/
├── pom.xml
└── src/main/java/com/example/App.java
```
