# PayPal Phishing Simulation — Kubernetes Project

## Overview

This project is a full-stack web application deployed on Kubernetes using Minikube. It simulates a PayPal login page that captures and stores user credentials in a MySQL database — built as a phishing-awareness simulation environment to demonstrate how such attacks are structured.

Here's what was built: a real-looking PayPal login page, a Python Flask backend API, and a MySQL database — all wired together and secured with TLS, running entirely on Kubernetes.

The application consists of three main components: a frontend served by Nginx, a Python Flask backend, and a MySQL database.

---

## What This Project Covers

- Set up a clean project structure with separate frontend, backend, and Kubernetes manifest directories.
- Wrote custom Dockerfiles (using Alpine 3.18 for the frontend), built images for both frontend and backend, and pushed them to Docker Hub.
- Generated TLS certificates, defined a custom kubeconfig user (`project` user) using that key/cert pair, created a new kubeconfig context, and set up a Role + RoleBinding so the user only has the permissions needed — nothing more.
- Created PersistentVolumes and PersistentVolumeClaims for both the frontend and the MySQL database, so data survives pod restarts.
- Deployed three services:
  - **Frontend:** Nginx serving the phishing page — instead of using an init container to pull the `index.html`, I used a ConfigMap to store the frontend code and mounted it as a volume directly inside the Nginx Pod. Clean, declarative, and no external dependencies at runtime.
  - **Backend:** A custom container running the logic layer.
  - **Database:** MySQL with persistent storage mounted at `/var/lib/mysql`.
- Exposed everything through an Ingress resource and mapped a custom hostname in `/etc/hosts` pointing to the Minikube IP.
- End-to-end verified: opened the hostname in a browser, filled in fake credentials, and confirmed they landed in the database.

> What makes this project valuable: it ties together so many concepts in one flow — image builds, RBAC, init containers, persistent storage, service networking, and ingress — all in a way that actually makes sense together.

---

## Architecture

The frontend presents a PayPal-styled login page to the user. When credentials are submitted, they are sent to the Flask backend via a POST request to the `/submit` endpoint. The backend then stores the credentials in a MySQL database for later retrieval.

---

## Components

**Frontend** — A static HTML/CSS login page served by an Nginx container. The page is styled to resemble a PayPal login form and supports Arabic language by default.

**Backend** — A Python Flask application that exposes a `/submit` endpoint. It receives the submitted credentials and stores them in the MySQL database using environment variables for configuration.

**Database** — A MySQL container that persists data using a Kubernetes PersistentVolume mounted at `/var/lib/mysql`.

---

## Kubernetes Resources

The application is deployed on Minikube with the following resources:

- **Deployments** for frontend, backend, and database
- **ClusterIP Services** for internal communication between components
- **PersistentVolumes and PersistentVolumeClaims** for frontend and database storage
- **Ingress** to expose the application externally via the hostname `project.local`
- **RBAC** with a dedicated Role and RoleBinding for the project user
- **ConfigMap** to inject the frontend HTML into the Nginx container

---

## Quick Start

**1. Start Minikube and enable Ingress:**
```bash
minikube start
minikube addons enable ingress
```

**2. Deploy all resources:**
```bash
kubectl apply -f project-files/frontend-pv-pvc.yaml
kubectl apply -f project-files/db-pv-pvc.yaml
kubectl apply -f project-files/frontend-deployment.yaml
kubectl apply -f project-files/backend-deployment.yaml
kubectl apply -f project-files/db-deployment.yaml
kubectl apply -f project-files/services.yaml
kubectl apply -f project-files/ingress.yaml
```

**3. Add host entry (Linux/WSL):**
```bash
echo "$(minikube ip) project.local" | sudo tee -a /etc/hosts
```

**4. Access the application:**
```bash
minikube service frontend-nodeport --url
```

**5. Verify credentials stored in database:**
```bash
kubectl exec -it $(kubectl get pod -l app=db -o jsonpath="{.items[0].metadata.name}") \
  -- mysql -u root -prootpassword phishing_database -e "SELECT * FROM credentials;"
```

---

## Docker Images

| Component | Image |
|-----------|-------|
| Frontend  | `rawdaessamrou/frontend:latest` |
| Backend   | `rawdaessamrou/backend:latest` |
