# k3s-raspberry-pi-cluster
Deploying a lightweight K3s cluster on Raspberry Pi and running a fullstack eCommerce web app (ProShop) with React frontend, Express backend, and MongoDB.

This project demonstrates manual YAML-based deployment on K3s without using Helm or automation scripts. It is designed for educational and demonstration purposes, including:

- Understanding Kubernetes resource configuration and deployment.
- Practicing image building, service exposure, and Ingress setup.
- Testing scalability using **Horizontal Pod Autoscaler (HPA)** under high load with **Locust**.


# ProShop Deployment Guide on K3s

---

## 📚 Table of Contents

- [📋 Prerequisites](#-prerequisites)  
- [🧭 System Architecture Overview](#system-architecture-overview)  
- [🐳 1: Build Docker Images](#-1-build-docker-images)  
  - [📦 Backend](#backend)  
  - [🎨 Frontend](#frontend)  
- [🚀 2: Deploy to K3s Cluster](#-2-deploy-to-k3s-cluster)  
  - [📂 Create Namespace (Optional)](#step-1-create-namespace-optional)  
  - [📥 Apply Kubernetes Manifests](#step-2-deploy-proshop-on-k3s)  
  - [🔎 Verify Deployment](#step-3-verify-the-deployment)  
- [🍃 3: Seed Initial Data into MongoDB](#-3-seed-initial-data-into-mongodb)  
- [🌐 4: Access the ProShop Web App](#-4-access-the-proshop-web-app)  
  - [🖼️ Web UI Screenshot](#️-web-ui-after-deployment)  
  - [🛣️ Ingress Networking Flow](#️-ingress-networking-flow-in-k3s)  
- [🚀 5: Test HPA with Load Testing](#-5-test-hpa-with-load-testing)  
  - [⚙️ Apply HPA Config](#step-1-apply-hpa-configuration)  
  - [🔍 Verify HPA](#step-2-verify-hpa-status)  
  - [🧪 Load Testing with Locust](#step-3-load-testing-with-locust)  
- [📌 Cluster High Availability Notes](#-cluster-high-availability-notes)  
- [📜 License & Credits](#-license--credits)

---
## System Architecture Overview

![image](https://github.com/user-attachments/assets/249063a7-5ad1-40aa-a3b1-bae94fdde68d)
- This setup includes 6 Raspberry Pi devices: 3 master nodes and 3 worker nodes. The cluster runs K3s in a High Availability (HA) configuration, allowing for stable and resilient orchestration of containerized applications. An admin machine is used to manage the cluster via kubectl.

- All devices are connected through a local network using a switch. The switch is linked to a router that provides internet access, enabling the cluster to pull container images and communicate externally when needed.

## 📋 Prerequisites

- A running **K3s cluster** with `kubectl` properly configured.
- A **Docker Hub** account (or other container registry) to push images.
- Docker installed on your local machine to build images.

---

## 🐳 1. Build Docker Images

### 📦 Backend

```bash
cd backend
docker build -t <dockerhub-username>/proshop-backend:v1 .
docker push <dockerhub-username>/proshop-backend:v1
cd ..
```
### 🎨 Frontend
```
cd frontend
docker build -t <dockerhub-username>/proshop-frontend:v1 .
docker push <dockerhub-username>/proshop-frontend:v1
cd ..
```

## 🚀  2. Deploy to K3s Cluster

### 📂 Step 1: Create Namespace (Optional)
```bash
kubectl create namespace ecommerce-web
```
### 📥 Step 2: Deploy ProShop on K3s
```bash
cd k3s-manifest
kubectl apply -f .
```
### 🔎 Step 3. Verify the deployment

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
## 🍃 3. Seed initial data into MongoDB

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
## 🌐 4. Access the ProShop Web App
Once all services are up and running, and the Ingress has been applied, you can access the application using the domain configured in your `ingress.yaml`.

For example: http://proshop.group14.vn

### 🖼️ Web UI after deployment

![Screenshot 2025-06-08 165329](https://github.com/user-attachments/assets/e2079327-5104-4152-ac74-e4ffe259d920)

## 🛣️  Ingress Networking Flow in K3s

The diagram below illustrates how external traffic is routed into the K3s cluster using Traefik as the Ingress Controller:

- The Client makes a request (e.g. via a domain like proshop.group14.vn).
- The request reaches the cluster through a NodePort, which exposes the Traefik Ingress Controller.
- Traefik handles the routing using Ingress rules, directing traffic to the correct Service.
- The Service then forwards the request to the appropriate Pod(s) running the application.

![image](https://github.com/user-attachments/assets/c998beb7-424f-4ced-87af-5cf211199b8f)

## 🚀 5. Test HPA with Load Testing

📌 Objective
Horizontal Pod Autoscaler (HPA) automatically scales the number of pods in a deployment based on observed CPU utilization or other custom metrics. In this project, HPA helps the backend scale out under high traffic conditions, ensuring performance and stability.

### ⚙️ Step 1: Apply HPA configuration
```bash
kubectl apply -f hpa-backend.yaml -n ecommerce-web
```
### 🔍 Step 2: Verify HPA status
```bash
kubectl get hpa -n ecommerce-web
```
### 🧪 Step 3: Load Testing with Locust
You can run Locust locally or inside a pod. Here's an example using local mode:

```bash
locust -f locustfile.py --host http://<your-backend-service>
```
Open http://localhost:8089 in your browser and configure number of users and spawn rate.

![image](https://github.com/user-attachments/assets/bd762304-f5f3-4d26-bb93-1fc510159dcd)

Locust provides real-time charts and statistics to monitor load testing performance. 

![Screenshot 2025-06-08 144520](https://github.com/user-attachments/assets/c0ca4b36-64ca-4e58-bcdd-df1b732b0167)

On the K3s server, observe the scaling in real-time:

```bash
kubectl get hpa -n ecommerce-web
kubectl get pods -n ecommerce-web
```

