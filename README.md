# Employee Directory: containers, CI/CD and AWS EKS

A small employee-management web app used to practise the full path from code to a running service: containers, three CI/CD systems, infrastructure as code and Kubernetes on AWS. The app lets you list, add, edit, delete and photograph employees. The interesting part is everything around it.

> Personal hands-on project. Based on DevOps course material.

## Architecture

```
Browser --> frontend (nginx, :8080) --/api--> backend (:5000) --> PostgreSQL
                                                  |
                                                  +--> CloudWatch Logs (optional)
```

- **Frontend:** static HTML pages served by nginx, which proxies `/api` to the backend.
- **Backend:** the same REST API implemented three times so the pipelines can be compared across stacks:
  - `backend/`: Python (Flask, SQLAlchemy)
  - `backend-node/`: Node.js (Express)
  - `backend-java/`: Java (Spring Boot)
- **Database:** PostgreSQL.

### API

| Method | Path | Purpose |
|---|---|---|
| GET | `/api/health` | Health check |
| GET | `/api/employees`, `/api/employees/{id}` | List and fetch employees |
| POST | `/api/employees` | Create an employee (with optional photo) |
| PUT | `/api/employees/{id}` | Update an employee |
| DELETE | `/api/employees/{id}` | Remove an employee |

## Run it locally

```bash
docker compose up --build
# frontend: http://localhost:8080   backend: http://localhost:5000
```

`docker-compose.yml` runs PostgreSQL, the Node backend and the frontend. `docker-compose.python.yml` does the same with the Python backend. Scripts in `scripts/` run each backend without Docker.

## CI/CD

The same pipeline is implemented on three platforms:

| Platform | File | What it does |
|---|---|---|
| GitHub Actions | `.github/workflows/` | `test.yml` on pushes, `deploy-dev.yml` from `develop`, `deploy-prod.yml` from `main` |
| CircleCI | `.circleci/config.yml` | `test`, `ecr-login`, `build-and-push` and `deploy` jobs |
| Jenkins | `Jenkinsfile` | Test, Build and Push, Deploy stages |

Each pipeline runs the tests, builds the image, pushes it to Amazon ECR, and deploys to the cluster. Registry login and cloud credentials come from each platform's secret store.

## Infrastructure and Kubernetes

| Path | Purpose |
|---|---|
| `terraform/` | ECR repositories, RDS PostgreSQL, Secrets Manager secrets and a CloudWatch log group, with `dev`, `stg` and `prod` environments under `terraform/env/` |
| `kubernetes/manifests/` | Plain manifests: namespace, ConfigMap, Secret, PostgreSQL, Deployment, Service, Ingress |
| `kubernetes/helm/` | Helm chart with values and CloudWatch dashboards |
| `kubernetes/argocd/` | Argo CD application for GitOps delivery |

## Configuration

- No real credentials are in this repository. Passwords in `terraform.tfvars` and manifests are placeholders. Provide real values through your CI secrets, Secrets Manager or `TF_VAR_*` variables.
- Image names, the AWS account ID (`123456789012`) and the state bucket (`example-terraform-state`) are placeholders. Replace them with your own.

