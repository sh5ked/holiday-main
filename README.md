# Holiday Events – DevOps Project

## Overview

Holiday Events is a Node.js/Express application deployed using a complete CI/CD process with GitHub, Jenkins, Docker, GHCR and Ansible.

The project demonstrates automatic build, image publishing, deployment to a separate server, and application health verification.

---

## Project Structure

```text
holiday-main/
├── server.js
├── package.json
├── package-lock.json
├── Dockerfile
├── Jenkinsfile
├── inventory.ini
├── deploy.yml
├── README.md
├── public/
└── data/
```

### Main Files

* `server.js` – Node.js/Express application.
* `Dockerfile` – Instructions for building the Docker image.
* `Jenkinsfile` – Defines the CI/CD pipeline and automatic SCM polling.
* `inventory.ini` – Ansible inventory containing the deployment server.
* `deploy.yml` – Ansible Playbook used to deploy the application.
* `README.md` – Project documentation.

---

## CI/CD Pipeline

The pipeline runs automatically after a new commit is pushed to the `main` branch.

```text
Git Push
   ↓
GitHub
   ↓
Jenkins SCM Polling
   ↓
Docker Build
   ↓
Push Image to GHCR
   ↓
Ansible Deployment
   ↓
SSH to Deployment Server
   ↓
Docker Pull
   ↓
Stop & Remove Previous Container
   ↓
Run New Container
   ↓
GET /health
```

### Pipeline Stages

1. **Build Docker Image**
   Jenkins builds the application into a Docker image.

2. **Push Image to GHCR**
   The image is pushed to GitHub Container Registry.

3. **Ansible Deployment**
   Jenkins runs Ansible against the deployment server.

4. **Health Check**
   Ansible verifies that the new application responds successfully through `/health`.

---

## Required Machines / Servers

### Jenkins / Control Machine

The main Windows machine runs:

* Docker Desktop
* Jenkins
* WSL
* Ansible

Jenkins is available at:

```text
http://localhost:8080
```

### Deployment Server

A separate Ubuntu server is used for application deployment:

```text
IP: 192.168.30.130
User: shaked
```

The server must have Docker installed and the `shaked` user must have permission to run Docker commands.

---

## Deployment Process

The deployment is performed by Ansible using SSH.

The Playbook performs:

1. Login to GHCR.
2. Pull the required Docker image.
3. Check whether the previous container exists.
4. Stop the previous container, if running.
5. Remove the previous container, if it exists.
6. Start the new container.
7. Perform a health check.

The Docker image tag is based on the Jenkins Build Number.

Example:

```text
Build #15
↓
ghcr.io/sh5ked/holiday-events:15
```

This allows each Jenkins build to have its own identifiable image version.

---

## Application Port

The Node.js application listens on port:

```text
3000
```

Docker exposes it on the deployment server through:

```text
8081:3000
```

Therefore the application is available at:

```text
http://192.168.30.130:8081
```

---

## Health Check

The application provides:

```text
GET /health
```

The health check is performed after the new container is started.

Ansible sends:

```text
GET http://127.0.0.1:8081/health
```

The deployment is considered successful when the endpoint returns:

```text
HTTP 200
```

Ansible retries the check if the application needs additional time to start.

---

## Ansible Configuration

### Inventory

`inventory.ini` contains the deployment server:

```ini
[deployment]
deployment-server ansible_host=192.168.30.130 ansible_user=shaked
```

### Playbook

The deployment logic is defined in:

```text
deploy.yml
```

---

## Jenkins Credentials

Secrets are stored in Jenkins Credentials and are not stored in GitHub.

Required credentials:

```text
ghcr-credentials
holiday-deploy-ssh
```

* `ghcr-credentials` – Used for authentication with GHCR.
* `holiday-deploy-ssh` – SSH private key used to connect to the deployment server.

No passwords, tokens or private keys are stored in the repository.

---

## Important Configuration

Before running the project, make sure:

* Docker Desktop is running.
* Jenkins is running.
* The deployment server `192.168.30.130` is online.
* Docker is running on the deployment server.
* Jenkins has the required credentials.
* `inventory.ini` contains the correct deployment server IP.
* Jenkins is configured to use `Jenkinsfile`.
* The repository branch is `main`.

---

## Running the Project

Start Jenkins:

```bash
docker start jenkins
```

Open:

```text
http://localhost:8080
```

Push a new commit:

```bash
git add .
git commit -m "Update application"
git push origin main
```

Jenkins detects the change automatically and starts the CI/CD pipeline.

At the end of a successful deployment, the application is available at:

```text
http://192.168.30.130:8081
```

Health check:

```text
http://192.168.30.130:8081/health
```
