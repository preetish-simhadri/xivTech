# Hello World Kubernetes Demo

## Project Overview
This project demonstrates how to deploy a simple "Hello World" web application to a Kubernetes cluster. The application consists of a basic HTML page served by NGINX, containerized with Docker, and deployed to a local Kubernetes cluster using Kind (Kubernetes in Docker).

## Prerequisites
Before you begin, ensure you have the following tools installed on your machine:

- **Docker**: For building and running containers [Install Docker](https://docs.docker.com/get-docker/)
- **Kind**: For creating a local Kubernetes cluster [Install Kind](https://kind.sigs.k8s.io/docs/user/quick-start/#installation)
- **kubectl**: For interacting with Kubernetes clusters [Install kubectl](https://kubernetes.io/docs/tasks/tools/)

## Step-by-Step Setup Instructions

### 1. Creating the Kubernetes Cluster
Create a local Kubernetes cluster using Kind:

```bash
# Create a cluster named hello-world-cluster
kind create cluster --name hello-world-cluster

# Verify the cluster is running
kubectl get nodes
```

### 2. Building and Loading the Docker Image
Build the Docker image containing NGINX and our HTML page:

```bash
# Build the Docker image (run from the project root)
docker build -t hello-world-nginx -f docker/Dockerfile .

# Load the image into the Kind cluster
kind load docker-image hello-world-nginx --name hello-world-cluster
```

### 3. Deploying to Kubernetes
Apply the Kubernetes manifests to deploy the application:

```bash
# Apply the deployment and service configurations
kubectl apply -f k8s/

# Verify the deployment is running
kubectl get deployments

# Verify the service is created
kubectl get services
```

### 4. Accessing the Application
Get the NodePort assigned to the service and access the application:

```bash
# Get the NodePort
kubectl get service hello-world-nginx

# The output will show the NodePort (e.g., 80:30123/TCP)
# Access the application at localhost:<NodePort>
# For example: http://localhost:30123
```

You can also use the following command to access the application:

```bash
# Get the NodePort using kubectl
NODE_PORT=$(kubectl get -o jsonpath="{.spec.ports[0].nodePort}" services hello-world-nginx)

# Access the application
curl http://localhost:$NODE_PORT
```

## Repository Structure

```
.
├── app/
│   └── index.html             # HTML file with Hello World content
├── docker/
│   └── Dockerfile             # Docker configuration for NGINX
├── k8s/
│   ├── deployment.yaml        # Kubernetes deployment configuration
│   └── service.yaml           # Kubernetes service configuration
├── demo/
│   └── video.mp4              # Demo video of the setup process
└── README.md                  # Project documentation
```

## Optional: Argo CD Setup

To implement continuous delivery with Argo CD:

1. Install Argo CD in your Kubernetes cluster:

```bash
# Create namespace
kubectl create namespace argocd

# Apply Argo CD installation manifest
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Wait for all pods to be ready
kubectl wait --for=condition=Ready pods --all -n argocd --timeout=300s
```

2. Access the Argo CD API Server:

```bash
# Port-forward the Argo CD API server
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

3. Get the initial admin password:

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

4. Log in to the Argo CD UI at http://localhost:8080 with username `admin` and the password from the previous step.

5. Create an application in Argo CD pointing to your Git repository containing this project.

---

Submit this project by pushing it to GitHub/GitLab. Make sure all the steps are reproducible for anyone who wants to try it out.

