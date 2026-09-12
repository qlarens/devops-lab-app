# ⚙️ DevOps Lab App

[![Python](https://img.shields.io/badge/Python-3.12-3776AB?logo=python&logoColor=white&style=for-the-badge)](https://www.python.org/)
[![Flask](https://img.shields.io/badge/Flask-3.x-000000?logo=flask&logoColor=white&style=for-the-badge)](https://flask.palletsprojects.com/)
[![Docker](https://img.shields.io/badge/Docker-Containerized-2496ED?logo=docker&logoColor=white&style=for-the-badge)](https://www.docker.com/)
[![Kubernetes](https://img.shields.io/badge/Kubernetes-Deployment-326CE5?logo=kubernetes&logoColor=white&style=for-the-badge)](https://kubernetes.io/)
[![Jenkins](https://img.shields.io/badge/Jenkins-CI%2FCD-D24939?logo=jenkins&logoColor=white&style=for-the-badge)](https://www.jenkins.io/)
[![License](https://img.shields.io/badge/License-CC_BY--NC--ND_4.0-blue.svg?style=for-the-badge)](LICENSE)

A lightweight Flask-based sample application used as the deployment target in a DevOps lab environment. The repository demonstrates a simple containerized web service that can be built with Docker, run locally with Docker Compose, and deployed into a Kubernetes cluster through a Jenkins CI/CD pipeline.

## Project Purpose

This repository is intentionally small and focused on showcasing:

- a minimal Python web service
- containerization with Docker
- local orchestration with Docker Compose
- Kubernetes deployment manifests
- CI/CD automation using Jenkins
- image publishing to GitHub Container Registry (GHCR)

## Application Overview

The service exposes two endpoints:

- `/` — returns a simple greeting string
- `/health` — returns `OK` and is suitable for health checks

The application is implemented in [main.py](main.py) and is served by Gunicorn in the container.

## Tech Stack

- Python 3.12
- Flask
- Gunicorn
- Docker
- Kubernetes
- Jenkins
- GitHub Container Registry

## Repository Structure

```text
.
├── Dockerfile
├── Jenkinsfile
├── docker-compose.yml
├── requirements.txt
├── main.py
└── k8s/
    ├── deployment.yaml.template
    ├── ingress.yaml.template
    └── service.yaml
```

## Local Development

### Prerequisites

- Docker
- Docker Compose
- Python 3.12+ (optional for local development outside containers)

### Run with Docker Compose

```bash
docker compose up --build
```

The service will be exposed on:

- `http://localhost:8000/`
- `http://localhost:8000/health`

### Run Manually

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
python main.py
```

The app will bind to `0.0.0.0:8000`.

## Containerization

The Docker image is defined in [Dockerfile](Dockerfile).

### Build the image

```bash
docker build -t devops-lab-app:latest .
```

### Run the image locally

```bash
docker run -p 8000:8000 devops-lab-app:latest
```

## Kubernetes Deployment

The Kubernetes manifests in the [k8s](k8s) directory are templates used during deployment:

- [k8s/deployment.yaml.template](k8s/deployment.yaml.template)
  - creates a 3-replica Deployment in the `devops-lab` namespace
  - uses the image built in Jenkins and stored in GHCR
  - pulls the image using a registry secret named `ghcr-auth`

- [k8s/service.yaml](k8s/service.yaml)
  - exposes the application through a Kubernetes Service

- [k8s/ingress.yaml.template](k8s/ingress.yaml.template)
  - creates an Ingress using the NGINX ingress class
  - routes requests to the service with a host generated from the node IP

## CI/CD Pipeline

The Jenkins pipeline is defined in [Jenkinsfile](Jenkinsfile).

### Pipeline stages

1. Checkout the source from SCM
2. Prepare the image tag and full image reference
3. Build the Docker image
4. Log in to GHCR and push the image
5. Apply Kubernetes manifests and wait for rollout completion

### Important environment variables

The pipeline expects the following Jenkins credentials:

- `ghcr-creds` — username/password for GitHub Container Registry
- `k3s-kubeconfig` — Kubernetes config file for deployment access

The pipeline also uses the environment variables:

- `REGISTRY = ghcr.io`
- `IMAGE_NAME = gutamurr/devops-lab-app`
- `GHCR_EMAIL = gutamurr@gmail.com`

## Deployment Notes

The deployment flow assumes:

- an accessible Kubernetes cluster
- a namespace named `devops-lab`
- a registry secret named `ghcr-auth`
- a reachable Ingress controller

The rollout status check is performed with:

```bash
kubectl rollout status deployment/app -n devops-lab
```

## Health Check

A basic health check endpoint is available at:

```text
/health
```

This endpoint can be used for liveness/readiness checks in container or Kubernetes monitoring setups.

## License

This repository is distributed under the Creative Commons Attribution-NonCommercial-NoDerivatives 4.0 International License. See [LICENSE](LICENSE) for details.
