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
│   ├── service.yaml           # Kubernetes service configuration
│   └── argocd-app.yaml        # Argo CD application manifest
├── demo/
│   └── steps_video.mp4        # Demo video of the setup process
└── README.md                  # Project documentation
```

## Argo CD Implementation

This project includes a working implementation of GitOps-based continuous delivery using Argo CD. The following steps outline how it was set up and how you can access it:

1. Argo CD was installed in the Kubernetes cluster:

```bash
# Create namespace
kubectl create namespace argocd

# Apply Argo CD installation manifest
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Wait for all pods to be ready
kubectl wait --for=condition=Ready pods --all -n argocd --timeout=300s
```

2. Access the Argo CD UI by setting up port forwarding:

```bash
# Port-forward the Argo CD API server
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

3. Get the initial admin password:

```bash
# This command will output the initial admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | ForEach-Object { [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($_)) }
```

4. Log in to the Argo CD UI:
   - Open your browser and navigate to: https://localhost:8080
   - Username: `admin`
   - Password: Use the output from the command in step 3
   - Note: You might need to accept the self-signed certificate warning in your browser

5. The application has been configured with the following settings:
   - Application Name: hello-world-nginx
   - Source Repository: https://github.com/preetish-simhadri/xivTech
   - Path in Repository: k8s/
   - Destination Namespace: default
   - Sync Policy: Automated (with self-healing and pruning enabled)

6. The application configuration is defined in the `k8s/argocd-app.yaml` file:

```bash
# Apply the Argo CD application manifest
kubectl apply -f k8s/argocd-app.yaml
```

With this setup, any changes pushed to the k8s directory in the GitHub repository will automatically be synchronized to your Kubernetes cluster, demonstrating a complete CI/CD pipeline.

---

Submit this project by pushing it to GitHub/GitLab. Make sure all the steps are reproducible for anyone who wants to try it out.

