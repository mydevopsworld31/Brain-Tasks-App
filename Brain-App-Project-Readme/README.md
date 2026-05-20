# 🧠 Brain Tasks App — DevOps Deployment

A full end-to-end DevOps project deploying a React application using Docker, Amazon EKS, AWS CodePipeline, CodeBuild, and CloudWatch monitoring.

---

## 📌 Project Overview

This project demonstrates the deployment of a React-based task management application using modern DevOps practices — from containerization and registry management to Kubernetes orchestration, CI/CD automation, and cloud monitoring.

---

## 🛠️ Tech Stack

| Category | Tools |
|---|---|
| Cloud | AWS EC2, Amazon EKS |
| Containerization | Docker, DockerHub |
| Orchestration | Kubernetes (kubectl) |
| CI/CD | AWS CodeBuild, AWS CodePipeline |
| Version Control | Git, GitHub |
| Monitoring | Amazon CloudWatch |

---

## 📦 Application Details

| Property | Value |
|---|---|
| Application | Brain Tasks App (React) |
| Source Repository | https://github.com/Vennilavanguvi/Brain-Tasks-App.git |
| Local Port | 3000 |
| Container/K8s Port | 80 |

---

## 🚀 Deployment Steps

### Step 1 — Application Setup

Clone the repository and install dependencies locally to verify the app runs before containerizing.

```bash
git clone https://github.com/Vennilavanguvi/Brain-Tasks-App.git
cd Brain-Tasks-App
npm install
npm run build
```

> The app runs on **port 3000** locally. Verify it loads in your browser at `http://localhost:3000` before proceeding.

---

### Step 2 — Dockerize the Application

Create a `Dockerfile` at the root of the project using **Nginx** as the web server to serve the production build.

```dockerfile
# Dockerfile
FROM node:18-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

Build and tag the Docker image:

```bash
docker build -t brain-app .
docker tag brain-app mydockerimage31/brain-app:v1
```

Run locally to verify:

```bash
docker run -p 80:80 mydockerimage31/brain-app:v1
```

---

### Step 3 — Push Image to DockerHub Registry

Log in to DockerHub and push the tagged image:

```bash
docker login
docker push mydockerimage31/brain-app:v1
```

> **Alternative:** You can also use **Amazon ECR** as your registry. Create a repository in ECR and use `aws ecr get-login-password` to authenticate before pushing.

---

### Step 4 — Set Up Amazon EKS Cluster

Create an EKS cluster and configure `kubectl` to connect to it:

```bash
# Create EKS cluster (using eksctl)
eksctl create cluster \
  --name brain-tasks-cluster \
  --region ap-south-1 \
  --nodegroup-name brain-nodes \
  --node-type t3.small \
  --nodes 2 \
  --nodes-min 2 \
  --nodes-max 4

# Update kubeconfig
aws eks update-kubeconfig --region ap-south-1 --name brain-tasks-cluster

# Verify cluster is running
kubectl get nodes
```

---

### Step 5 — Write Kubernetes YAML Files

**`deployment.yaml`** — Deploys the containerized app with replica scaling:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: brain-tasks-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: brain-tasks
  template:
    metadata:
      labels:
        app: brain-tasks
    spec:
      containers:
        - name: brain-tasks
          image: mydockerimage31/brain-app:v1
          ports:
            - containerPort: 80
```

**`service.yaml`** — Exposes the app to the internet via an AWS LoadBalancer:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: brain-tasks-service
spec:
  type: LoadBalancer
  selector:
    app: brain-tasks
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

Apply both files:

```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml

# Get the LoadBalancer external URL
kubectl get svc brain-tasks-service
```

---

### Step 6 — Set Up AWS CodeBuild

Create a `buildspec.yml` at the project root. This file defines the build commands CodeBuild will execute:

```yaml
version: 0.2

phases:
  pre_build:
    commands:
      - echo Logging in to DockerHub...
      - echo $DOCKERHUB_PASSWORD | docker login -u $DOCKERHUB_USERNAME --password-stdin
      - IMAGE_TAG=$(echo $CODEBUILD_RESOLVED_SOURCE_VERSION | cut -c 1-7)

  build:
    commands:
      - echo Building Docker image...
      - docker build -t brain-app .
      - docker tag brain-app mydockerimage31/brain-app:$IMAGE_TAG
      - docker tag brain-app mydockerimage31/brain-app:latest

  post_build:
    commands:
      - echo Pushing Docker image...
      - docker push mydockerimage31/brain-app:$IMAGE_TAG
      - docker push mydockerimage31/brain-app:latest
      - echo Build complete.

artifacts:
  files:
    - deployment.yaml
    - service.yaml
```

**Create the CodeBuild Project:**
- Source: Connect to your GitHub repository
- Environment: Managed image → Amazon Linux / Ubuntu
- Buildspec: Use `buildspec.yml` from source
- Environment variables: Add `DOCKERHUB_USERNAME` and `DOCKERHUB_PASSWORD` as secrets (use AWS Secrets Manager)

---

### Step 7 — Set Up AWS CodePipeline

Create a pipeline with the following stages:

| Stage | Configuration |
|---|---|
| **Source** | GitHub → select your repo and branch (`main`) |
| **Build** | AWS CodeBuild → select the project from Step 6 |
| **Deploy** | EKS deploy stage → `kubectl apply -f deployment.yaml -f service.yaml` |

Push your code to trigger the pipeline:

```bash
git add .
git commit -m "Initial deployment"
git push origin main
```

---

### Step 8 — Troubleshooting: Pod Scheduling Issue

**Issue encountered:** `Too many pods` error — pods couldn't be scheduled due to insufficient node resources (t3.micro).

**Solution:** Scale the node group from 2 to 3 nodes:

```bash
eksctl scale nodegroup \
  --cluster brain-tasks-cluster \
  --name brain-nodes \
  --nodes 3 \
  --region ap-south-1
```

> **Tip:** Use `t3.small` or larger instance types to avoid pod scheduling limits. `t3.micro` has a low ENI pod limit.

---

### Step 9 — Monitoring with CloudWatch

Enable CloudWatch Container Insights for your EKS cluster and verify pod logs:

```bash
# View pod logs
kubectl get pods
kubectl logs <pod-name>

# Describe a pod for events/errors
kubectl describe pod <pod-name>
```

In the AWS Console:
- Navigate to **CloudWatch → Log Groups**
- Find logs under `/aws/eks/brain-tasks-cluster/`
- Monitor build logs under **CodeBuild → Build history**

---

## 🌐 Live Application

Application is exposed via AWS LoadBalancer:

```
http://a63aee061b39847d38bd25db59f6f39a-1219897690.ap-south-1.elb.amazonaws.com
```

---

## 📁 Repository Structure

```
Brain-Tasks-App/
├── public/
├── src/
├── Dockerfile
├── buildspec.yml
├── deployment.yaml
├── service.yaml
├── package.json
└── README.md
```

---

## 📤 Submission Checklist

- [x] GitHub repository with full source code
- [x] `README.md` with setup instructions and pipeline explanation
- [x] `Dockerfile`, `buildspec.yml`, `deployment.yaml`, `service.yaml` included
- [x] Application deployed and accessible via Kubernetes LoadBalancer
- [x] LoadBalancer ARN: `a63aee061b39847d38bd25db59f6f39a-1219897690.ap-south-1.elb.amazonaws.com`
- [ ] Screenshots of pipeline, EKS cluster, and CloudWatch logs (attach to submission document)

---

## 💡 Key Learnings

- Docker containerization with multi-stage builds
- Kubernetes Deployments and Services on Amazon EKS
- EKS cluster creation and `kubectl` configuration
- Writing `buildspec.yml` for AWS CodeBuild
- End-to-end CI/CD with AWS CodePipeline
- Debugging real-world pod scheduling issues
- Node group scaling in Kubernetes
- Application monitoring using CloudWatch

---

## ✅ Conclusion

Successfully deployed a scalable React application using a full DevOps pipeline — from source code on GitHub to a live, publicly accessible URL — using Docker, Amazon EKS, AWS CodeBuild, and CodePipeline with CloudWatch monitoring.
