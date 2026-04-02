# Kubernetes Project

## Overview
Production-grade Kubernetes cluster with monitoring, ingress, and DNS.

## Tools Used
- Kind (Kubernetes in Docker)
- Kubernetes Dashboard
- NGINX Ingress Controller

## Files
- kind-config.yaml - Cluster configuration
- dashboard-admin-user.yaml - Dashboard admin user
- sample-app.yaml - Sample nginx application

## How to Run

### 1. Create the cluster
kind create cluster --name my-cluster --config kind-config.yaml

### 2. Install the Dashboard
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
kubectl apply -f dashboard-admin-user.yaml

### 3. Install Ingress Controller
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.9.6/deploy/static/provider/kind/deploy.yaml

### 4. Deploy Sample App
kubectl apply -f sample-app.yaml

### 5. Add DNS
echo "127.0.0.1 hello.local" | sudo tee -a /etc/hosts

### 6. Test
curl http://hello.local
