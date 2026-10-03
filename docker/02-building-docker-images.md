# Docker: Building Docker Images - Interview Questions & Answers


## Table of Contents

1. [What is a Dockerfile? How do you build a Docker image?](#1-what-is-a-dockerfile-how-do-you-build-a-docker-image)
2. [What is a Base Image?](#2-what-is-a-base-image)
3. [Why is the Base Image a Key Choice?](#3-why-is-the-base-image-a-key-choice)
4. [What is the difference between ENTRYPOINT and CMD in a Dockerfile? Explain with example.](#4-what-is-the-difference-between-entrypoint-and-cmd-in-a-dockerfile-explain-with-example)
5. [What is the difference between ADD and COPY in a Dockerfile?](#5-what-is-the-difference-between-add-and-copy-in-a-dockerfile)
6. [How do you tag a Container image? Why is tagging important?](#6-how-do-you-tag-a-container-image-why-is-tagging-important)
7. [Explain the layered approach to building a Container image.](#7-explain-the-layered-approach-to-building-a-container-image)
8. [How can you design Dockerfiles to maximize layer reuse?](#8-how-can-you-design-dockerfiles-to-maximize-layer-reuse)
9. [What is a multi-stage build, and how can it be used to reduce the size of a container image?](#9-what-is-a-multi-stage-build-and-how-can-it-be-used-to-reduce-the-size-of-a-container-image)
10. [Is it possible to make changes to an existing Container image?](#10-is-it-possible-to-make-changes-to-an-existing-container-image)
11. [What is the purpose of the .dockerignore file?](#11-what-is-the-purpose-of-the-dockerignore-file)
12. [What Are Alternatives to Writing Dockerfile?](#12-what-are-alternatives-to-writing-dockerfile)
13. [What is OCI and Why Is It Important?](#13-what-is-oci-and-why-is-it-important)

---

## 1. What is a Dockerfile? How do you build a Docker image?

A **Dockerfile** contains instructions Docker uses to build an image.

### Python backend example

```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 8000
CMD ["python", "app.py"]
```

Build and run it:

```bash
docker build -t python-api:1.0 .
docker run -d -p 8000:8000 python-api:1.0
```

For a React application, the same idea applies: choose a Node base image, install dependencies, copy the source code, and build the application.

---

## 2. What is a Base Image?

A **base image** is the starting point of a Docker image. It usually provides the operating-system environment or language runtime your application needs.

Examples for our stack:

```dockerfile
# Python backend
FROM python:3.12-slim

# React build environment
FROM node:24-alpine
```

We then add our application code, dependencies, and configuration on top of that base image.

---

## 3. Why is the Base Image a Key Choice?

The base image affects:

- **Size:** Smaller images download and deploy faster.
- **Security:** Fewer unnecessary packages generally mean a smaller attack surface.
- **Runtime requirements:** A Python backend needs Python, while building React normally requires Node.js.
- **Maintenance:** Well-maintained official images receive updates and have better documentation.

For example, `python:3.12-slim` is usually preferable to a much larger general-purpose image when the application only needs Python.

---

## 4. What is the difference between ENTRYPOINT and CMD in a Dockerfile? Explain with example.

**`ENTRYPOINT` defines the main executable**, while **`CMD` provides the default command or arguments**.

```dockerfile
ENTRYPOINT ["python"]
CMD ["app.py"]
```

By default:

```bash
docker run my-python-app
# Runs: python app.py
```

If we pass another argument:

```bash
docker run my-python-app test.py
# Runs: python test.py
```

Here `test.py` replaces **CMD**, while `ENTRYPOINT` (`python`) stays unchanged.

**Key difference:**

- `CMD` is easily overridden by arguments passed to `docker run`.
- `ENTRYPOINT` stays fixed unless explicitly overridden with `--entrypoint`.
- When used together: **ENTRYPOINT = executable**, **CMD = default arguments**.

Easy way to remember: **ENTRYPOINT = what to run; CMD = what to run with it by default.**

---

## 5. What is the difference between ADD and COPY in a Dockerfile?

`COPY` simply copies files or directories from the build context into the image.

```dockerfile
COPY requirements.txt /app/
COPY src/ /app/src/
```

`ADD` can also copy files, but has extra capabilities such as handling certain archives and remote URLs.

```dockerfile
ADD frontend-assets.tar.gz /app/static/
```

For normal Python or React application files, **prefer `COPY` when you only need to copy files** because its behavior is simpler and clearer.

---

## 6. How do you tag a Container image? Why is tagging important?

Use `-t` when building an image:

```bash
docker build -t python-api:1.0 .
docker build -t react-frontend:1.0 .
```

You can also add another tag:

```bash
docker tag python-api:1.0 python-api:latest
```

Tags are important because they help with **versioning, deployments, and rollbacks**.

```text
python-api:1.0
python-api:1.1
react-frontend:1.0
```

If version `1.1` causes a problem, the deployment can be rolled back to `1.0`.

---

## 7. Explain the layered approach to building a Container image.

Docker images are built in **layers**. Instructions such as `FROM`, `RUN`, and `COPY` create layers, and Docker can cache unchanged layers for future builds.

Example:

```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
```

If only Python source code changes, Docker may reuse the cached dependency layers and rebuild only the later `COPY . .` layer.

If an earlier layer changes, that layer and the layers after it may need to be rebuilt.

---

## 8. How can you design Dockerfiles to maximize layer reuse?

Put **less frequently changing dependencies first** and **frequently changing source code later**.

### Python backend

```dockerfile
FROM python:3.12-slim
WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .
CMD ["python", "app.py"]
```

Now changing `app.py` does not require reinstalling Python packages unless `requirements.txt` also changes.

### React frontend

```dockerfile
FROM node:24-alpine
WORKDIR /app

COPY package.json package-lock.json ./
RUN npm ci

COPY . .
RUN npm run build
```

Changing React source code does not rerun `npm ci` unless the dependency files change.

**Interview point:** copy dependency files first, install dependencies, then copy application code.

---

## 9. What is a multi-stage build, and how can it be used to reduce the size of a container image?

A **multi-stage build** uses one stage to build the application and another smaller stage to run or serve the final output. Build tools and source files do not need to remain in the final image.

This is especially useful for React.

```dockerfile
# Stage 1: Build React
FROM node:24-alpine AS builder
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 2: Serve only the production build
FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
EXPOSE 80
```

The final image contains the built React files and Nginx, but not Node.js, `node_modules`, or the React source/build environment.

### Benefits

- Smaller production image
- Fewer unnecessary tools and packages
- Smaller attack surface
- Faster image transfer and deployment

---

## 10. Is it possible to make changes to an existing Container image?

Docker images are designed to be **immutable/read-only**. Instead of modifying an existing image, we normally change the source or Dockerfile and build a **new image version**.

For example:

```bash
docker build -t python-api:1.0 .

# After changing the application
docker build -t python-api:1.1 .
```

Docker also allows changes made inside a running container to be committed into a new image:

```bash
docker commit my-running-container python-api:modified
```

However, for normal application development and CI/CD, rebuilding from the Dockerfile is preferable because it is reproducible and keeps the infrastructure definition in source control.

---

## 11. What is the purpose of the .dockerignore file?

`.dockerignore` prevents unnecessary files and directories from being sent in the Docker build context.

For a Python + React project:

```dockerignore
# React
node_modules/
dist/
build/

# Python
__pycache__/
*.pyc
.venv/
venv/

# Git / editor / local files
.git/
.env
*.log
.DS_Store
.vscode/
.idea/
```

This helps keep the build context smaller and avoids accidentally copying local development files into the image.

---

## 12. What Are Alternatives to Writing Dockerfile?

The source mentions tools such as **Jib** and **Buildpacks** that can create container images without manually writing a Dockerfile.

For a Python or React application, **Buildpacks** are the more relevant example because they support multiple languages and can detect the application runtime and build an image automatically.

Conceptually:

```bash
pack build my-python-app --builder <builder>
```

or for an appropriate React/Node project:

```bash
pack build my-react-app --builder <builder>
```

For interviews, the main point is: **Dockerfiles give explicit control, while tools such as Buildpacks can automate image creation and common best practices.**

---

## 13. What is OCI and Why Is It Important?

**OCI (Open Container Initiative)** defines open, vendor-neutral standards for container technologies.

It standardizes areas such as:

- **Image specification:** how container images are structured.
- **Runtime specification:** how containers are run.
- **Distribution specification:** how container images are distributed through registries.

For example, whether our image contains a **Python API** or **React frontend**, OCI standards help make container images portable across compatible container tools and cloud platforms.

The key interview point is: **OCI reduces vendor lock-in by providing common container standards.**

---