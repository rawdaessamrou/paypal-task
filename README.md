# Paypal Phishing Simulation - Kubernetes Project

## Overview
This project is a full-stack web application deployed on Kubernetes using Minikube. It simulates a PayPal login page that captures and stores user credentials in a MySQL database. The application consists of three main components: a frontend served by Nginx, a Python Flask backend, and a MySQL database.

## Architecture
The frontend presents a PayPal-styled login page to the user. When credentials are submitted, they are sent to the Flask backend via a POST request to the `/submit` endpoint. The backend then stores the credentials in a MySQL database for later retrieval.

## Components

**Frontend** — A static HTML/CSS login page served by an Nginx container. The page is styled to resemble a PayPal login form and supports Arabic language by default.

**Backend** — A Python Flask application that exposes a `/submit` endpoint. It receives the submitted credentials and stores them in the MySQL database using environment variables for configuration.

**Database** — A MySQL container that persists data using a Kubernetes PersistentVolume mounted at `/var/lib/mysql`.

## Kubernetes Resources
The application is deployed on Minikube with the following resources:
- **Deployments** for frontend, backend, and database
- **ClusterIP Services** for internal communication between components
- **PersistentVolumes and PersistentVolumeClaims** for frontend and database storage
- **Ingress** to expose the application externally via the hostname `project.local`
- **RBAC** with a dedicated Role and RoleBinding for the project user
- **ConfigMap** to inject the frontend HTML into the Nginx container

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

## Docker Images
- Frontend: `rawdaessamrou/frontend:latest`
- Backend: `rawdaessamrou/backend:latest`
