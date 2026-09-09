# Project Walkthrough & Evidence: Movie Picture Pipeline

All CI/CD pipelines, AWS infrastructure, and deployments for the **Movie Picture Pipeline** have been completed, verified, and are live.

---

## 1. Project URLs & Repository

| Component | URL |
| :--- | :--- |
| **GitHub Repository** | [https://github.com/Tharunks007/movie-picture-pipeline](https://github.com/Tharunks007/movie-picture-pipeline) |
| **Frontend Application (Live URL)** | [http://a945b59b1b77b4bfca31b71e6745bccb-1983175658.us-east-1.elb.amazonaws.com](http://a945b59b1b77b4bfca31b71e6745bccb-1983175658.us-east-1.elb.amazonaws.com) |
| **Backend API `/movies` Endpoint** | [http://a0001996bbdff4eb487bdcea5ae4bdb4-1119427289.us-east-1.elb.amazonaws.com/movies](http://a0001996bbdff4eb487bdcea5ae4bdb4-1119427289.us-east-1.elb.amazonaws.com/movies) |

---

## 2. GitHub Actions CI/CD Pipeline Runs

All 4 pipelines were configured, executed automatically, and completed with **100% green status**.

| Workflow | File | Trigger | Run ID | Status | URL |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Frontend CI** | `.github/workflows/frontend-ci.yaml` | Pull Request (`#1`) | `34360746920` | `SUCCESS` | [View Run](https://github.com/Tharunks007/movie-picture-pipeline/actions/runs/34360746920) |
| **Backend CI** | `.github/workflows/backend-ci.yaml` | Pull Request (`#2`) | `34362509149` | `SUCCESS` | [View Run](https://github.com/Tharunks007/movie-picture-pipeline/actions/runs/34362509149) |
| **Backend CD** | `.github/workflows/backend-cd.yaml` | Push to `main` | `34363279428` | `SUCCESS` | [View Run](https://github.com/Tharunks007/movie-picture-pipeline/actions/runs/34363279428) |
| **Frontend CD** | `.github/workflows/frontend-cd.yaml` | Push to `main` | `34369097231` | `SUCCESS` | [View Run](https://github.com/Tharunks007/movie-picture-pipeline/actions/runs/34369097231) |

---

## 3. Workflow Specifications & Rubric Verification

### Frontend CI (`frontend-ci.yaml`)
- **Parallel Execution**: `lint` and `test` run concurrently.
- **Dependency Caching**: Utilizes `actions/cache@v4` with `package-lock.json` hash.
- **Sequential Build**: `build` job uses `needs: [lint, test]`.
- **Docker Build Argument**: Builds using `--build-arg REACT_APP_MOVIE_API_URL=http://localhost:5000`.

### Backend CI (`backend-ci.yaml`)
- **Parallel Execution**: `lint` and `test` run concurrently.
- **Python & Pipenv Setup**: Python 3.10 with pipenv virtualenv caching.
- **Test Failure Simulation Support**: Evaluates `os.getenv("FAIL_TEST", 200)` via pytest.
- **Sequential Build**: `build` job runs only after `lint` and `test` pass (`needs: [lint, test]`).

### Backend CD (`backend-cd.yaml`)
- **Lint & Test**: Passes before build/deploy.
- **AWS Credentials**: Securely accessed via `${{ secrets.AWS_ACCESS_KEY_ID }}` and `${{ secrets.AWS_SECRET_ACCESS_KEY }}`. No credentials hardcoded anywhere.
- **ECR Integration**: Uses `aws-actions/amazon-ecr-login@v2` to authenticate and push images tagged with commit SHA and `latest`.
- **K8s Deployment**: Dynamically updates the container image using `kustomize edit set image` and deploys to EKS using `kubectl apply`.

### Frontend CD (`frontend-cd.yaml`)
- **Lint & Test**: Passes before build/deploy.
- **Dynamic API URL Injection**: Builds container with `--build-arg REACT_APP_MOVIE_API_URL="${{ secrets.REACT_APP_MOVIE_API_URL }}"` pointing to the deployed backend LoadBalancer URL.
- **ECR Push & K8s Deployment**: Image tagged with commit SHA and deployed to EKS via Kustomize.

---

## 4. Live Verification Evidence

### A. Backend `/movies` Response
```bash
$ curl -i http://a0001996bbdff4eb487bdcea5ae4bdb4-1119427289.us-east-1.elb.amazonaws.com/movies
```
**Output:**
```http
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 133
Access-Control-Allow-Origin: *

{"movies":[{"id":"123","title":"Top Gun: Maverick"},{"id":"456","title":"Sonic the Hedgehog"},{"id":"789","title":"A Quiet Place"}]}
```

### B. Individual Movie Endpoints
- `/movies/123`: `{"movie":{"description":"Fighter planes","title":"Top Gun: Maverick"}}`
- `/movies/456`: `{"movie":{"description":"Blue Sega character","title":"Sonic the Hedgehog"}}`
- `/movies/789`: `{"movie":{"description":"Scary monsters","title":"A Quiet Place"}}`

### C. Frontend Live Web Page
```bash
$ curl -i http://a945b59b1b77b4bfca31b71e6745bccb-1983175658.us-east-1.elb.amazonaws.com
```
**Output:**
```html
HTTP/1.1 200 OK
Content-Length: 644
Content-Type: text/html; charset=utf-8

<!doctype html><html lang="en"><head><meta charset="utf-8"/><link rel="icon" href="/favicon.ico"/><meta name="viewport" content="width=device-width,initial-scale=1"/><title>React App</title><script defer="defer" src="/static/js/main.93cf7e2c.js"></script><link href="/static/css/main.86fcb180.css" rel="stylesheet"></head><body><noscript>You need to enable JavaScript to run this app.</noscript><div id="root"></div></body></html>
```

### D. Kubernetes Cluster Status (`default` namespace)
```
NAME                        READY   STATUS    RESTARTS   AGE
backend-985698899-6djxn     1/1     Running   0          65m
frontend-55f7cfd599-qwxsb   1/1     Running   0          13m

NAME         TYPE           CLUSTER-IP       EXTERNAL-IP                                                               PORT(S)        AGE
backend      LoadBalancer   172.20.83.91     a0001996bbdff4eb487bdcea5ae4bdb4-1119427289.us-east-1.elb.amazonaws.com   80:32369/TCP   66m
frontend     LoadBalancer   172.20.109.252   a945b59b1b77b4bfca31b71e6745bccb-1983175658.us-east-1.elb.amazonaws.com   80:30953/TCP   13m
```

### E. EKS Worker Nodes
```
NAME                         STATUS   ROLES    AGE    VERSION
ip-10-0-1-140.ec2.internal   Ready    <none>   140m   v1.31.13-eks-ecaa3a6
```

---

## 5. Teardown / Cleanup Instructions (Step 14)

> [!CAUTION]
> **Do not destroy AWS resources until after your project has been evaluated or screenshots have been submitted!**

When you are ready to destroy the AWS infrastructure to avoid consuming credits, run the following command from the workspace:

```powershell
.\setup\terraform\bin\terraform.exe -chdir=setup/terraform destroy -auto-approve
```
