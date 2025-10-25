# learn-dockerizing-restapi 🚀🐳

[![Repository](https://img.shields.io/badge/repo-ridhorambu93/learn--dockerizing--restapi-blue)](https://github.com/ridhorambu93/learn-dockerizing-restapi) [![License](https://img.shields.io/github/license/ridhorambu93/learn-dockerizing-restapi)](https://github.com/ridhorambu93/learn-dockerizing-restapi/blob/main/LICENSE) [![Language](https://img.shields.io/badge/JavaScript-96.1%25-yellow)](#languages) [![Dockerfile](https://img.shields.io/badge/Dockerfile-3.9%25-blue)](#languages)

A hands-on, easy-to-follow example repository that demonstrates how to dockerize a Node.js REST API — Dockerfile best practices, development workflows with Docker Compose, and notes for production readiness.

---

## Table of Contents 📚
- [About](#about)
- [Languages](#languages)
- [Features ✨](#features-)
- [Quick Start ▶️](#quick-start-️)
- [Run with Docker 🐳](#run-with-docker-)
  - [Build image](#build-image)
  - [Run container](#run-container)
  - [Docker Compose](#docker-compose)
- [Example Dockerfile (multi-stage)](#example-dockerfile-multi-stage)
- [Environment Variables 🔐](#environment-variables-)
- [Tests 🧪](#tests-)
- [Production Tips 🛠️](#production-tips-)
- [CI/CD Example (GitHub Actions) ⚙️](#cicd-example-github-actions-)
- [Contributing 🤝](#contributing-)
- [License 📄](#license)
- [Contact ✉️](#contact)

---

## About
This repository is a compact, practical guide to containerizing REST APIs built with Node.js (Express or similar). It focuses on readable Dockerfiles, reproducible local development with Docker Compose, and notes to help you transition to production.

Repo: https://github.com/ridhorambu93/learn-dockerizing-restapi

---

## Languages 🧾
- JavaScript — 96.1%  
- Dockerfile — 3.9%

---

## Features ✨
- Clear, minimal Dockerfile examples (including multi-stage builds)
- Docker Compose setup for local development
- Example project structure for a REST API
- Commands to build, run, and test locally and inside containers
- Practical production tips (security, image size, healthchecks)

---

## Quick Start ▶️

Clone the repository and try it locally:

```bash
git clone https://github.com/ridhorambu93/learn-dockerizing-restapi.git
cd learn-dockerizing-restapi
npm install        # or yarn install
npm start          # or the start script defined in package.json
```

Open the API at: http://localhost:3000 (replace port if different in your project)

---

## Run with Docker 🐳

### Build image
From the repository root:
```bash
docker build -t learn-dockerizing-restapi:latest .
```

For a production-style multi-stage image:
```bash
docker build -t learn-dockerizing-restapi:prod -f Dockerfile .
```

### Run container
```bash
docker run -d --name restapi -p 3000:3000 \
  --env-file .env \
  learn-dockerizing-restapi:latest
```

Follow logs:
```bash
docker logs -f restapi
```

Stop & remove:
```bash
docker stop restapi && docker rm restapi
```

### Docker Compose
If `docker-compose.yml` is present:
```bash
docker-compose up --build
# or run in background:
docker-compose up -d
```

---

## Example Dockerfile (multi-stage)
A recommended minimal multi-stage Dockerfile to build a smaller production image:

```dockerfile
# Stage 1: build
FROM node:18-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci --production=false
COPY . .
RUN npm run build || true

# Stage 2: runtime
FROM node:18-alpine
WORKDIR /app
ENV NODE_ENV=production
COPY --from=build /app/package*.json ./
COPY --from=build /app/node_modules ./node_modules
COPY --from=build /app/dist ./dist
COPY --from=build /app/src ./src
USER node
EXPOSE 3000
CMD ["node", "dist/index.js"] # adjust to your actual entry point
```

Adjust `npm run build`, file paths, and entrypoint according to your repository structure.

---

## Environment Variables 🔐
Store sensitive configuration in a `.env` file (do not commit). Example:
```
PORT=3000
NODE_ENV=production
DATABASE_URL=postgres://user:pass@db:5432/dbname
```
Ensure `.env` is listed in `.gitignore`.

---

## Tests 🧪
If tests exist in the project:
```bash
npm test
```
You can also run tests inside a container to guarantee environment parity:
```bash
docker run --rm learn-dockerizing-restapi:latest npm test
```

---

## Production Tips 🛠️
- Use multi-stage builds to reduce image size.
- Avoid running the app as root — use a non-root user (see `USER node` above).
- Add a HEALTHCHECK in the Dockerfile for orchestration readiness:
  ```dockerfile
  HEALTHCHECK --interval=30s --timeout=5s --start-period=5s \
    CMD curl -f http://localhost:3000/health || exit 1
  ```
- Log to stdout/stderr for centralised logging.
- Manage secrets using your orchestrator (Kubernetes secrets, Docker secrets, Vault).
- Pin base images and tag your images by digest or semantic version; avoid :latest in production.

---

## CI/CD Example (GitHub Actions) ⚙️
A simple workflow to install, test, build, and optionally push a Docker image:

```yaml
name: CI

on:
  push:
    branches:
