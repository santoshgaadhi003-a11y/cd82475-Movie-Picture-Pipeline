# Movie Picture CI/CD Pipeline Architecture & Operations Guide

## Executive Summary
This document provides a comprehensive operational overview of the automated Continuous Integration and Continuous Deployment (CI/CD) pipelines designed for the **Movie Picture** application. Using **GitHub Actions**, containerization with **Docker**, and orchestration with **Kubernetes (Amazon EKS)** and **Amazon ECR**, this pipeline ensures rapid, dependable, and reproducible releases.

---

## 1. System Architecture & Component Overview

```mermaid
flowchart LR
    Dev([Developer]) -->|Pull Request| GH_PR[GitHub Actions CI]
    Dev -->|Merge to main| GH_CD[GitHub Actions CD]

    subgraph GitHub Actions CI
        direction TB
        LINT_J[Linting: ESLint / Flake8]
        TEST_J[Testing: Jest / Pytest]
        BUILD_J[Build & Verification]
        LINT_J & TEST_J --> BUILD_J
    end

    GH_PR --> LINT_J & TEST_J

    subgraph GitHub Actions CD
        direction TB
        CD_LINT[Lint Check]
        CD_TEST[Test Suite]
        CD_BUILD[Docker Build & Push to ECR]
        CD_DEPLOY[Kustomize Apply & Rollout to EKS]
        CD_LINT & CD_TEST --> CD_BUILD --> CD_DEPLOY
    end

    GH_CD --> CD_LINT & CD_TEST

    CD_BUILD -->|Push Image with Git SHA| ECR[(Amazon ECR)]
    CD_DEPLOY -->|Deploy via Kustomize| EKS[(Amazon EKS Cluster)]
    ECR -->|Pull Image| EKS
```

### Key Components:
- **Frontend App (`starter/frontend`)**: React 18 single-page application communicating with the backend movies REST API. Containerized with Node Alpine and served via `serve`.
- **Backend App (`starter/backend`)**: Python 3.10 Flask REST API exposing `/movies`. Containerized with Python Alpine and served via uWSGI on port 5000.
- **Continuous Integration (CI)**: Validates code quality and unit tests on pull requests in parallel before verifying application buildability.
- **Continuous Deployment (CD)**: Automatically triggers upon merges to `main`, tags Docker images with the exact commit Git SHA (`${{ github.sha }}`), pushes them to Amazon ECR, updates Kubernetes manifests using `kustomize`, and verifies rollout status in Amazon EKS.

---

## 2. GitHub Actions Workflows Specification

The pipeline is split into four distinct, modular workflows in `.github/workflows/`:

| Workflow File | Target Component | Trigger Events | Parallel Jobs | Dependent Jobs |
|---|---|---|---|---|
| `frontend-ci.yaml` | `starter/frontend/**` | `pull_request` (target: `main`), `workflow_dispatch` | `lint`, `test` | `build` (requires `lint` + `test`) |
| `frontend-cd.yaml` | `starter/frontend/**` | `push` (branch: `main`), `workflow_dispatch` | `lint`, `test` | `build` (requires `lint` + `test`), `deploy` (requires `build`) |
| `backend-ci.yaml` | `starter/backend/**` | `pull_request` (target: `main`), `workflow_dispatch` | `lint`, `test` | `build` (requires `lint` + `test`) |
| `backend-cd.yaml` | `starter/backend/**` | `push` (branch: `main`), `workflow_dispatch` | `lint`, `test` | `build` (requires `lint` + `test`), `deploy` (requires `build`) |

### Workflow Safeguards & Best Practices:
1. **Path Filtering**: Workflows only trigger when relevant directories change (`starter/frontend/**` or `starter/backend/**`), preventing unnecessary runner minutes.
2. **Strict Quality Gates**: Deployment (`deploy`) and Docker pushing (`build`) are strictly blocked unless all tests and linters pass successfully (`needs: [lint, test]`).
3. **Immutable Image Tagging**: Container images are tagged with `${{ github.sha }}`. This establishes 1:1 traceability between the running container and the Git commit history.
4. **Dynamic Manifest Patching**: Uses `kustomize edit set image` to inject the SHA-tagged container image without permanently modifying tracked git files.
5. **Rollout Verification**: Executes `kubectl rollout status` with a timeout to prevent silent deployment failures.
6. **Detailed Job Summaries**: Emits rich markdown dashboards to `$GITHUB_STEP_SUMMARY` for instant visibility.

---

## 3. Required GitHub Secrets Configuration

To enable the CD workflows to push container images to Amazon ECR and deploy to your Amazon EKS cluster, configure the following secrets in **GitHub Repository Settings -> Secrets and variables -> Actions**:

| Secret Name | Required? | Description | Example / Default |
|---|---|---|---|
| `AWS_ACCESS_KEY_ID` | **Yes** | AWS Access Key for `github-action-user` | `AKIAIOSFODNN7EXAMPLE` |
| `AWS_SECRET_ACCESS_KEY` | **Yes** | AWS Secret Access Key for `github-action-user` | `wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY` |
| `AWS_DEFAULT_REGION` | Optional | AWS Region where resources reside | `us-east-1` (default) |
| `EKS_CLUSTER_NAME` | Optional | Name of the Amazon EKS cluster | `cluster` (default) |
| `FRONTEND_ECR_REPO` | Optional | Frontend ECR repository name or full URL | `frontend` |
| `BACKEND_ECR_REPO` | Optional | Backend ECR repository name or full URL | `backend` |
| `REACT_APP_MOVIE_API_URL` | Optional | Backend LoadBalancer service endpoint | `http://<load-balancer-dns>` |

---

## 4. Local Testing & Simulating Quality Gate Failures

### Testing the Frontend
```bash
cd starter/frontend

# Install dependencies cleanly
npm ci

# Run linter
npm run lint

# Simulate lint failure (verifies CI catches bad formatting)
FAIL_LINT=true npm run lint

# Run unit tests
CI=true npm test

# Simulate test failure (verifies CI catches broken assertions)
FAIL_TEST=true CI=true npm test
```

### Testing the Backend
```bash
cd starter/backend

# Install virtualenv dependencies
pipenv install --dev

# Run linter
pipenv run lint

# Simulate lint failure
pipenv run lint-fail

# Run unit tests
pipenv run test

# Simulate test failure
FAIL_TEST=true pipenv run test
```

---

## 5. Kubernetes Verification Commands

After the Continuous Deployment workflow finishes, verify the deployment directly via `kubectl`:

```bash
# Verify deployments are healthy
kubectl get deployments -n default

# Inspect pods and verify image tag matches the Git commit SHA
kubectl get pods -n default -o wide

# Check LoadBalancer services and retrieve external URLs
kubectl get svc -n default

# Test backend API
curl http://<BACKEND_LOADBALANCER_URL>/movies

# Open the frontend in your browser at http://<FRONTEND_LOADBALANCER_URL>
```
