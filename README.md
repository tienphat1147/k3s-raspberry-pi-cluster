# k3s-raspberry-pi-cluster
Deploying a lightweight K3s cluster on Raspberry Pi and running a fullstack eCommerce web app (ProShop) with React frontend, Express backend, and MongoDB. Manual YAML-based deployment without scripts. Designed for education and demonstration purposes.

# ProShop Deployment Guide on K3s

---

## 📚 Table of Contents

- [Prerequisites](#-prerequisites)
- [1. Build Docker Images](#-1-build-docker-images)
- [2. Deploy to K3s Cluster](#-2-deploy-to-k3s-cluster)
- [3. Verify the deployment](#step-3-verify-the-deployment)
- [4. Seed initial data into MongoDB](#-4-seed-initial-data-into-mongodb)
- [5. Access the ProShop Web App](#-5-access-the-proshop-web-app)
- [6. (Optional) Test HPA with Load Testing](#-6-optional-test-hpa-with-load-testing)

---

## 📋 Prerequisites

- A running **K3s cluster** with `kubectl` properly configured.
- A **Docker Hub** account (or other container registry) to push images.
- Docker installed on your local machine to build images.

---

## 🚀 1. Build Docker Images

### Backend

```bash
cd backend
docker build -t <dockerhub-username>/proshop-backend:v1 .
docker push <dockerhub-username>/proshop-backend:v1
cd ..
```
### Frontend
```
cd frontend
docker build -t <dockerhub-username>/proshop-frontend:v1 .
docker push <dockerhub-username>/proshop-frontend:v1
cd ..
```

## 🚀  2. Deploy to K3s Cluster

### Step 1: Create Namespace (Optional)
```bash
kubectl create namespace ecommerce-web
```
### Step 2: Deploy ProShop on K3s
```bash
cd k3s-manifest
kubectl apply -f .
```
### Step 3. Verify the deployment

Check pods status:
```bash
kubectl get pods -n ecommerce-web -o wide
```
Check services:
```
kubectl get svc -n ecommerce-web
```
Check ingress:
```bash
kubectl get ingress -n ecommerce-web
```
## 🚀 4. Seed initial data into MongoDB

To populate the database, exec into the backend pod and run the seeder script:
### Step 1. Find the backend pod name:
```bash
kubectl get pods -n ecommerce -l app=proshop-backend -o jsonpath="{.items[0].metadata.name}"
```
### Step 2. Exec into the backend pod (replace <backend-pod-name> with the actual pod name):
```bash
kubectl exec -it -n ecommerce <backend-pod-name> -- /bin/sh
```
### Step 3. Run the seeder script inside the pod:
```bash
node seeder.js
```
### Step 4. Exit the pod shell after seeding is complete:
```bash
exit
```
## 🚀 5. Access the ProShop Web App
Once all services are up and running, and the Ingress has been applied, you can access the application using the domain configured in your `ingress.yaml`.

For example: http://proshop.group14.vn

### 🖼️ Web UI after deployment

![Screenshot 2025-06-08 165329](https://github.com/user-attachments/assets/e2079327-5104-4152-ac74-e4ffe259d920)

## 🚀 6. (Optional) Test HPA with Load Testing

### Step 1: Apply HPA configuration
```bash
kubectl apply -f hpa-backend.yaml -n ecommerce-web
```
### Step 2: Verify HPA status
```bash
kubectl get hpa -n ecommerce-web
```
### Step 3: Generate load using Locust
You can run Locust locally or inside a pod. Here's an example using local mode:

```bash
locust -f locustfile.py --host http://<your-backend-service>
```
You can create a locustfile.py to simulate multiple requests. Watch how HPA scales the pods dynamically.

