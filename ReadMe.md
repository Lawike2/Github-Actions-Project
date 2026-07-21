# GitHub Actions CI/CD Project — Banking Web Application

## Overview

This repository demonstrates a full end-to-end **CI/CD pipeline** using **GitHub Actions** for a Spring Boot–based **Banking Web Application** (`bankapp`). It covers everything from source code compilation and security scanning, through Docker image publishing, to deployment on an **Amazon EKS** Kubernetes cluster.

---

## Application

The core application is a **Java 17 / Spring Boot 3.3** web app (`com.example.bankapp`) backed by **MySQL 8**. It provides:

- **Account management** — create and manage bank accounts (`Account`, `AccountRepository`, `AccountService`)
- **Transaction tracking** — record and query transactions (`Transaction`, `TransactionRepository`)
- **Web UI** — Thymeleaf-rendered pages served by `BankController`
- **Spring Security** — authentication and authorization configured in `SecurityConfig`

### Tech Stack

| Layer | Technology |
|---|---|
| Language | Java 17 |
| Framework | Spring Boot 3.3 (Web, Data JPA, Security, Thymeleaf) |
| Database | MySQL 8 |
| Build tool | Maven (with Wrapper) |
| Test coverage | JaCoCo |
| Container | Docker (eclipse-temurin:17-jdk-alpine base image) |

---

## CI/CD Pipeline

The pipeline is defined in [`.github/workflows/cicd.yml`](.github/workflows/cicd.yml) and triggers on every push to the `main` branch. It runs on a **self-hosted GitHub Actions runner** and consists of six sequential jobs:

```
compile → security-check → test → build_project_and_sonar_scan → buils_docker_image_and_push → deploy_to_kubernetes
```

### Pipeline Jobs

| Job | Description |
|---|---|
| **compile** | Checks out code and compiles with `mvn compile` using JDK 17 (Temurin). |
| **security-check** | Runs **Trivy** (filesystem vulnerability scan) and **Gitleaks** (secret detection) against the source tree. |
| **test** | Executes unit tests with `mvn test`. |
| **build_project_and_sonar_scan** | Packages the application (`mvn package`), uploads the JAR as a GitHub Actions artifact, and runs a **SonarQube** scan with quality gate enforcement. |
| **buils_docker_image_and_push** | Downloads the JAR artifact, builds a Docker image, and pushes it to **Docker Hub** (`adijaiswal/bankapp:latest`). |
| **deploy_to_kubernetes** | Installs the AWS CLI and `kubectl`, configures credentials for an **AWS EKS** cluster, and applies the Kubernetes manifests (`ds.yml`). |

### Required Secrets & Variables

| Name | Type | Used By |
|---|---|---|
| `SONAR_TOKEN` | Secret | SonarQube scan & quality gate |
| `SONAR_HOST_URL` | Variable | SonarQube scan & quality gate |
| `DOCKERHUB_USERNAME` | Variable | Docker Hub login |
| `DOCKERHUB_TOKEN` | Secret | Docker Hub login |
| `AWS_ACCESS_KEY_ID` | Secret | AWS CLI / EKS deploy |
| `AWS_SECRET_ACCESS_KEY` | Secret | AWS CLI / EKS deploy |
| `EKS_KUBECONFIG` | Secret | kubectl configuration |

---

## Kubernetes Deployment

The [`ds.yml`](ds.yml) manifest deploys both the application and its database to Kubernetes:

- **MySQL Deployment & Service** — runs `mysql:8`, exposes port `3306` internally via `mysql-service`.
- **BankApp Deployment** — runs `adijaiswal/bankapp:latest`, connects to MySQL via environment variables, exposes port `8080`.
- **BankApp Service (LoadBalancer)** — exposes the application externally on port `80` → `8080`.

See [`Setup-RBAC.md`](Setup-RBAC.md) for instructions on creating the required Kubernetes `ServiceAccount`, `Role`, `RoleBinding`, `ClusterRole`, and `ClusterRoleBinding` used for deployment.

---

## Code Quality

- **SonarQube** project key: `GC-Bank` (configured in [`sonar-project.properties`](sonar-project.properties))
- **JaCoCo** generates code coverage reports during the `test` phase, consumed by SonarQube.
- **Trivy** scans all filesystem dependencies for known vulnerabilities.
- **Gitleaks** scans the repository for accidentally committed secrets.

---

## Repository Structure

```
.
├── .github/
│   └── workflows/
│       └── cicd.yml          # GitHub Actions CI/CD pipeline
├── src/
│   ├── main/java/com/example/bankapp/
│   │   ├── BankappApplication.java
│   │   ├── config/SecurityConfig.java
│   │   ├── controller/BankController.java
│   │   ├── model/            # Account, Transaction entities
│   │   ├── repository/       # Spring Data JPA repositories
│   │   └── service/          # AccountService business logic
│   └── test/
├── Dockerfile                # Docker image definition
├── ds.yml                    # Kubernetes deployment manifests
├── Setup-RBAC.md             # Kubernetes RBAC setup guide
├── sonar-project.properties  # SonarQube configuration
└── pom.xml                   # Maven project descriptor
```
