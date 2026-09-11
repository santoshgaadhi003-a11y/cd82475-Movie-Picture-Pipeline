# Movie Picture Pipeline - Project Evidence & Screenshots

This document contains the visual proof and verification artifacts for the **Movie Picture Pipeline** CI/CD implementation using **GitHub Actions**, containerization, and **Kubernetes**.

All workflows have been executed and verified directly on the GitHub repository:
**Repository Actions URL:** [https://github.com/santoshgaadhi003-a11y/cd82475-Movie-Picture-Pipeline/actions](https://github.com/santoshgaadhi003-a11y/cd82475-Movie-Picture-Pipeline/actions)

---

## 1. Frontend Continuous Integration (CI) Pipeline
- **Workflow**: `frontend-ci.yaml`
- **Execution**: Runs `Frontend Lint (ESLint)` and `Frontend Unit Tests (Jest)` in parallel, followed by `Frontend Application Build Verification`.
- **Status**: ✅ All jobs passed (100% Green).
- **Live Run URL**: [Frontend CI Run #34621006629](https://github.com/santoshgaadhi003-a11y/cd82475-Movie-Picture-Pipeline/actions/runs/34621006629)

![Frontend CI Success](screenshots/01_frontend_ci_pipeline_success.jpg)

---

## 2. Backend Continuous Integration (CI) Pipeline
- **Workflow**: `backend-ci.yaml`
- **Execution**: Runs `Backend Lint (Flake8)` and `Backend Unit Tests (Pytest)` in parallel, followed by `Backend Container Build Verification`.
- **Status**: ✅ All jobs passed (100% Green).
- **Live Run URL**: [Backend CI Run #34627658379](https://github.com/santoshgaadhi003-a11y/cd82475-Movie-Picture-Pipeline/actions/runs/34627658379)

![Backend CI Success](screenshots/02_backend_ci_pipeline_success.jpg)

---

## 3. Simulated Test Failure Caught by CI Quality Gates
- **Objective**: Demonstrates that the CI pipeline prevents faulty code from reaching production when tests or linters fail (simulated via `FAIL_TEST=true` / `FAIL_LINT=true`).
- **Result**: Immediate pipeline halt, preventing downstream artifact creation and cluster deployment.
- **Live Run Evidence**: [Backend CI Build Gate Failure #34621006713](https://github.com/santoshgaadhi003-a11y/cd82475-Movie-Picture-Pipeline/actions/runs/34621006713)

![Simulated Test Failure](screenshots/03_ci_simulated_test_failure_caught.jpg)

---

## 4. Continuous Deployment (CD) Pipeline & Kubernetes Rollout
- **Workflows**: `frontend-cd.yaml` & `backend-cd.yaml`
- **Execution**: Validates code quality gates, builds and tags Docker container images with the exact commit Git SHA (`${{ github.sha }}`), pushes to Amazon ECR, updates Kubernetes manifests using Kustomize (`kustomize edit set image`), applies to the cluster, and verifies rollout status.
- **Status**: ✅ All deployment stages completed successfully.
- **Live Run URLs**:
  - [Frontend CD Run #34621006497](https://github.com/santoshgaadhi003-a11y/cd82475-Movie-Picture-Pipeline/actions/runs/34621006497)
  - [Backend CD Run #34627658340](https://github.com/santoshgaadhi003-a11y/cd82475-Movie-Picture-Pipeline/actions/runs/34627658340)

![CD Pipeline Deployment Success](screenshots/04_cd_pipeline_deployment_success.jpg)

---

## 5. Deployed Application Frontend Verification
- **Application URL**: `http://localhost:3000` (or Kubernetes Service / Ingress endpoint)
- **Features Displayed**:
  - `Movie List` header and list of items (`Top Gun: Maverick`, `Sonic the Hedgehog`, `A Quiet Place`) retrieved via GET request to the backend `/movies` API endpoint.
  - `Movie Details` section displaying the selected movie title and description (`Top Gun: Maverick` - `Fighter planes`).

![Deployed Frontend Application](screenshots/05_movie_picture_frontend_deployed.jpg)
