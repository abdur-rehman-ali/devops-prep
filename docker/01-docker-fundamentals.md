# Docker Fundamentals

A concise set of DevOps interview notes covering Docker fundamentals, traditional deployment, container architecture, and the role of containers in DevOps and microservices.

---

## Table of Contents

1. [Traditional Deployment Before Containers](#1-traditional-deployment-before-containers)
2. [Challenges with Traditional Deployment](#2-challenges-with-traditional-deployment)
3. [Container Image Build and Deployment Flow](#3-container-image-build-and-deployment-flow)
4. [Advantages of Docker](#4-advantages-of-docker)
5. [Virtual Machines vs Containers](#5-virtual-machines-vs-containers)
6. [Important Components in Container Architecture](#6-important-components-in-container-architecture)
7. [Why Containerization Matters for DevOps and Microservices](#7-why-containerization-matters-for-devops-and-microservices)
8. [Quick Interview Revision](#8-quick-interview-revision)

---

## 1. Traditional Deployment Before Containers

Before containers, application deployment was commonly **documentation-driven**.

Teams maintained setup documents describing how to prepare a server and deploy an application.

### Typical Process

```text
Provision Server / VM
        ↓
Install Operating System
        ↓
Install Runtime
(Java / Python / Node.js)
        ↓
Install Dependencies
        ↓
Configure Environment
        ↓
Deploy Application
        ↓
Start Application
```

### Main Steps

- **Hardware / VM Setup**
  - Provision a physical server or virtual machine.

- **Operating System Setup**
  - Install and configure Linux, Windows, or another supported OS.

- **Runtime Installation**
  - Install Java, Python, Node.js, Ruby, etc.

- **Dependency Configuration**
  - Install libraries, packages, system dependencies, and configuration.

- **Application Deployment**
  - Copy the application code or artifact to the server.
  - Configure and start the application.

### Interview Summary

> Traditional deployments required preparing each environment separately and manually installing the runtime, dependencies, configuration, and application.

---

## 2. Challenges with Traditional Deployment

Traditional deployment creates several operational problems.

### Multiple Setup Steps

Every server must be prepared with the correct:

- OS
- Runtime
- Libraries
- Dependencies
- Configuration
- Application version

### Time-Consuming

The same setup process may need to be repeated across:

```text
Local → Development → QA → Staging → Production
```

### Configuration Drift

Over time, environments can become different.

Example:

```text
Development
Node.js 20
Library X v3.1

Production
Node.js 18
Library X v2.9
```

The application may work in development but fail in production.

### Manual Errors

Manual installation and configuration increase the chance of:

- Missing dependencies
- Incorrect versions
- Wrong environment variables
- Incorrect configuration
- Inconsistent server setup

### Different Deployment Instructions

Different technology stacks require different setup procedures.

Examples:

```text
Java Application
→ Install JDK
→ Configure JVM
→ Deploy JAR/WAR

Python Application
→ Install Python
→ Create virtual environment
→ Install pip packages

Node.js Application
→ Install Node.js
→ Install npm packages
```

### Interview Summary

> The main problem with traditional deployment is environment inconsistency. Manual setup creates configuration drift, deployment errors, and the classic "works on my machine" problem.

---

## 3. Container Image Build and Deployment Flow

Containerization packages the application and its runtime dependencies into a portable image.

### Deployment Flow

```text
Application Code
       ↓
   Dockerfile
       ↓
docker build
       ↓
Container Image
       ↓
docker push
       ↓
Container Registry
       ↓
docker pull
       ↓
Run Container
       ↓
DEV / QA / STAGE / PROD
```

### Step 1: Build the Container Image

Create an image containing:

- Application code
- Runtime
- Required libraries
- Application dependencies
- Required user-space OS packages
- Startup instructions

Example:

```bash
docker build -t my-app:1.0 .
```

### Step 2: Push the Image to a Registry

Upload the image to a container registry.

```bash
docker push my-registry/my-app:1.0
```

Common registries include:

- Docker Hub
- Amazon ECR
- Google Artifact Registry
- Azure Container Registry

### Step 3: Deploy the Image

The target environment pulls and runs the same image.

```bash
docker pull my-registry/my-app:1.0

docker run my-registry/my-app:1.0
```

The same image can move through:

```text
Development
     ↓
QA
     ↓
Staging
     ↓
Production
```

### Key Principle

> Build once, deploy the same immutable image everywhere.

---

## 4. Advantages of Docker

### Standardized Deployment Package

Docker provides one common packaging format regardless of the application technology.

```text
Java App   ─┐
Python App ─┼──→ Container Image
Node App   ─┘
```

### Standardized Deployment Process

Applications can be started using the same operational model.

```bash
docker run <image>
```

The application inside the image could be:

- Java
- Python
- Node.js
- Go
- Ruby
- .NET

### Consistent Environments

The same application image is used across environments.

```text
Laptop → CI → QA → Staging → Production
          Same Image
```

This reduces environment-related failures.

### Better Automation

Containers integrate naturally with:

- GitHub Actions
- GitLab CI
- Jenkins
- AWS CodePipeline
- Kubernetes

### Standardized Operations

Containers make it easier to standardize:

- Logging
- Metrics
- Tracing
- Health checks
- Deployment
- Scaling

### Lightweight Compared with VMs

Containers share the host machine's kernel.

Therefore they generally:

- Start faster
- Use less memory
- Consume less disk space
- Allow higher workload density

---

## 5. Virtual Machines vs Containers

| Feature | Virtual Machines | Containers |
|---|---|---|
| Architecture | Virtualized machine with a full guest OS | Isolated processes sharing the host kernel |
| Virtualization Layer | Hypervisor | Container runtime / engine |
| OS | Each VM has its own guest OS | Containers share the host kernel |
| Startup | Usually slower | Usually much faster |
| Resource Usage | Higher | Lower |
| Image Size | Usually GBs | Often MBs to hundreds of MBs |
| Isolation | Stronger OS-level boundary | Process-level isolation |
| Density | Fewer workloads per host | More workloads per host |
| Portability | Good | Very high for compatible platforms |
| Typical Use | Strong isolation, different OS requirements | Microservices, CI/CD, cloud-native workloads |

### Architecture Comparison

#### Virtual Machines

```text
+---------------------------+
|        Application        |
+---------------------------+
|        Guest OS           |
+---------------------------+
|          VM               |
+---------------------------+
|       Hypervisor          |
+---------------------------+
|         Host OS           |
+---------------------------+
|        Hardware           |
+---------------------------+
```

Each VM includes its own operating system.

#### Containers

```text
+-------------+-------------+
| Container A | Container B |
| App + Deps  | App + Deps  |
+-------------+-------------+
|     Container Runtime     |
+---------------------------+
|       Host OS Kernel      |
+---------------------------+
|          Hardware         |
+---------------------------+
```

Containers share the host kernel while maintaining isolated processes and filesystems.

### Important Interview Point

A container image does **not** contain a complete operating system kernel.

It usually contains:

```text
Application
+ Runtime
+ Libraries
+ User-space OS files/packages
```

The container uses the **host machine's kernel**.

---

## 6. Important Components in Container Architecture

The main components are:

```text
Dockerfile
    ↓
Container Image
    ↓
Container Registry
    ↓
Container Engine
    ↓
Running Container
```

### Container Image

A **container image** is an immutable package containing everything required to start an application.

It typically includes:

- Application code
- Runtime
- Libraries
- Dependencies
- User-space OS packages
- Startup command

Example:

```text
my-api:1.0
my-api:1.1
my-api:2.0
```

Each tag can represent a different version.

### Container

A **container** is a running instance of a container image.

```text
Image = Blueprint
Container = Running Instance
```

Example:

```bash
docker run nginx
```

Here:

```text
nginx image
     ↓
running nginx container
```

Multiple containers can be created from the same image.

```text
nginx:latest
   ├── Container 1
   ├── Container 2
   └── Container 3
```

### Container Engine / Runtime

The container engine manages containers.

Responsibilities include:

- Pull images
- Create containers
- Start and stop containers
- Configure networking
- Configure storage
- Maintain isolation

Docker commonly uses lower-level components such as:

```text
Docker
   ↓
containerd
   ↓
runc
   ↓
Linux Kernel
```

### Container Registry

A registry stores and distributes container images.

Examples:

- Docker Hub
- Amazon ECR
- Google Artifact Registry
- Azure Container Registry
- GitHub Container Registry

Typical flow:

```text
Developer
   ↓
Build Image
   ↓
Push
   ↓
Registry
   ↓
Pull
   ↓
Server / Kubernetes
```

### Container Repository

A repository is a collection of related images for a particular application.

Example:

```text
Repository: payment-service

payment-service:1.0
payment-service:1.1
payment-service:1.2
payment-service:latest
```

### Registry vs Repository

```text
Registry
│
├── catalog-service
│   ├── 1.0
│   └── 2.0
│
├── payment-service
│   ├── 1.0
│   └── 1.1
│
└── order-service
    └── 3.0
```

A **registry** contains many repositories.

A **repository** contains different versions/tags of one application or image.

---

## 7. Why Containerization Matters for DevOps and Microservices

## What is Containerization?

Containerization packages:

```text
Application
+
Runtime
+
Dependencies
+
Configuration requirements
```

into a portable container image.

The goal is to run the application consistently across environments.

---

### Consistent Environments

Without containers:

```text
Developer Machine
Node 20
↓
Production
Node 18
↓
Possible Failure
```

With containers:

```text
Developer
   ↓
Container Image
   ↓
QA
   ↓
Staging
   ↓
Production

Same runtime + same dependencies
```

This greatly reduces the:

> "It works on my machine."

problem.

---

### Faster Deployment

Instead of manually installing dependencies:

```text
Install runtime
Install packages
Configure system
Copy application
Start service
```

you can run:

```bash
docker run my-app
```

---

### Better CI/CD Automation

Containers fit naturally into CI/CD pipelines.

```text
Developer Push
     ↓
CI Pipeline
     ↓
Run Tests
     ↓
Build Image
     ↓
Push Image
     ↓
Deploy
```

Example:

```text
GitHub
   ↓
GitHub Actions
   ↓
docker build
   ↓
Amazon ECR
   ↓
Kubernetes / ECS
```

---

### Microservice Independence

Each microservice can have its own container image.

Example:

```text
E-Commerce Application

Catalog Service
Go
↓
catalog-service image

Cart Service
Java
↓
cart-service image

Checkout Service
Node.js
↓
checkout-service image

Order Service
Python
↓
order-service image
```

Despite using different technologies, deployment becomes standardized.

```bash
docker run catalog-service
docker run cart-service
docker run checkout-service
docker run order-service
```

---

### Independent Scaling

Containers can be scaled independently.

Example:

```text
Catalog Service
2 Containers

Checkout Service
10 Containers

Payment Service
5 Containers
```

If checkout traffic increases, only the Checkout Service needs additional instances.

With Kubernetes:

```bash
kubectl scale deployment checkout --replicas=10
```

---

### Supports Orchestration

Containerized applications work well with orchestration platforms such as Kubernetes.

Kubernetes can manage:

- Scheduling
- Scaling
- Service discovery
- Load balancing
- Self-healing
- Rolling deployments
- Health checks

---

### Better Observability

A consistent container model makes it easier to standardize:

```text
Logs
Metrics
Traces
Health Checks
```

across applications written in different languages.

---

### Platform Portability

A container image can run on any compatible container platform.

Examples:

```text
Developer Laptop
AWS
Azure
GCP
On-Premises Server
Kubernetes
```

> Portability still depends on compatible CPU architecture, operating-system kernel requirements, and external infrastructure.

---

### Resource Efficiency

Containers share the host kernel.

This normally results in:

- Smaller images than VMs
- Faster startup
- Lower memory overhead
- Higher application density

Example:

```text
Physical Server
│
├── Container 1
├── Container 2
├── Container 3
├── Container 4
├── Container 5
└── ...
```

A comparable VM-based setup would typically require significantly more memory and storage.

---

## 8. Quick Interview Revision

### What problem does Docker solve?

> Docker packages an application with its runtime and dependencies into a consistent image, reducing environment differences and making deployments repeatable.

### What is a Docker image?

> An immutable package containing application code, runtime, libraries, dependencies, and startup instructions.

### What is a container?

> A running instance of a container image.

### What is a container registry?

> A centralized system used to store and distribute container images.

### What is the difference between an image and a container?

```text
Image     → Blueprint / Package
Container → Running Instance
```

### Why are containers lighter than VMs?

> Containers share the host OS kernel instead of running a complete guest operating system for every workload.

### What is the standard container deployment flow?

```text
Code
 ↓
Dockerfile
 ↓
Build Image
 ↓
Push to Registry
 ↓
Pull Image
 ↓
Run / Deploy Container
```

### Why is Docker useful for microservices?

> Each service can be packaged, deployed, versioned, and scaled independently while using the same deployment process regardless of programming language.

### What does "build once, run anywhere" mean?

> Build an immutable image once and promote that same artifact through development, testing, staging, and production instead of rebuilding it for each environment.

---

## Key Takeaways

```text
Traditional Deployment
        ↓
Manual Setup
        ↓
Environment Differences
        ↓
Deployment Problems
```

Docker changes the model to:

```text
Application + Dependencies
          ↓
     Container Image
          ↓
      Registry
          ↓
Same Image Everywhere
          ↓
Consistent Deployment
```

The core idea is:

> **Package once, deploy consistently, automate everything around it.**
