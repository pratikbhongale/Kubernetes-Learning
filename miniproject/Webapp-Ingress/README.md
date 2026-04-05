# 🚀 Kubernetes Multi-Service Application with Ingress (HTTP)

## 📌 Project Overview

This project demonstrates a **real-world Kubernetes deployment** of a multi-service application using:

* Frontend (NGINX)
* Backend (API service)
* Kubernetes Services for internal communication
* Ingress for external access (HTTP)

⚠️ Note: This version uses **HTTP only (no TLS/HTTPS)** to focus on core networking fundamentals.

---

## 🎯 Objectives

* Understand Kubernetes architecture in practice
* Learn service-to-service communication
* Implement Ingress for routing traffic
* Build debugging and operational thinking

---

## 🏗️ Architecture

```text
User (Browser)
       ↓
   HTTP Request
       ↓
Ingress (NGINX)
       ↓
--------------------------
|                        |
Frontend Service     Backend Service
       ↓                    ↓
Frontend Pods         Backend Pods
```

---

## 📁 Project Structure

```text
k8s-project/
│
├── namespace.yaml
├── frontend-deployment.yaml
├── frontend-service.yaml
├── backend-deployment.yaml
├── backend-service.yaml
└── ingress.yaml
```

---

# 🧩 Kubernetes Manifests (FULL CODE)

---

## 1️⃣ Namespace

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: demo-app
```

Apply:

```bash
kubectl apply -f namespace.yaml
```

---

## 2️⃣ Frontend Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  namespace: demo-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

---

## 3️⃣ Frontend Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-svc
  namespace: demo-app
spec:
  selector:
    app: frontend
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP
```

---

## 4️⃣ Backend Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
  namespace: demo-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: backend
        image: hashicorp/http-echo
        args:
        - "-text=Hello from Backend"
        ports:
        - containerPort: 5678
```

---

## 5️⃣ Backend Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-svc
  namespace: demo-app
spec:
  selector:
    app: backend
  ports:
  - port: 80
    targetPort: 5678
  type: ClusterIP
```

---

## 6️⃣ Ingress Resource (HTTP Only)

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: demo-ingress
  namespace: demo-app
spec:
  rules:
  - host: demo.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-svc
            port:
              number: 80

      - path: /api
        pathType: Prefix
        backend:
          service:
            name: backend-svc
            port:
              number: 80
```

---

# ⚙️ Setup Instructions

---

## Step 1: Start Cluster

```bash
minikube start
```

---

## Step 2: Enable Ingress Controller

```bash
minikube addons enable ingress
```

---

## Step 3: Apply All Resources

```bash
kubectl apply -f .
```

---

## Step 4: Get Minikube IP

```bash
minikube ip
```

Example:

```text
192.168.49.2
```

---

## Step 5: Configure Hosts File

Edit `/etc/hosts`:

```text
192.168.49.2 demo.local
```

---

## Step 6: Access Application

* http://demo.local → Frontend
* http://demo.local/api → Backend

---

# 🔄 Traffic Flow

```text
User → Ingress → Service → Pod
```

Example:

```text
http://demo.local/api
        ↓
Ingress
        ↓
backend-svc
        ↓
backend pod
```

---

# 💡 Use Cases

* Microservices architecture learning
* API gateway simulation
* Internal developer platforms
* Routing multiple services via single domain

---

# ⚠️ Common Errors & Debugging

---

## 🔴 Ingress Not Working

```bash
kubectl get pods -n ingress-nginx
```

👉 Fix:

```bash
minikube addons enable ingress
```

---

## 🔴 404 Error

```bash
kubectl describe ingress demo-ingress -n demo-app
```

👉 Check:

* Path
* Service name

---

## 🔴 Service Not Connecting to Pods

```bash
kubectl get pods --show-labels
kubectl describe svc frontend-svc
```

👉 Cause:

* Label mismatch

---

## 🔴 Pod Crash

```bash
kubectl logs <pod-name>
```

---

## 🔴 Cannot Access Domain

👉 Ensure `/etc/hosts` is configured correctly.

---

# 🛠️ Debugging Workflow (Golden Steps)

```bash
kubectl get pods
kubectl logs <pod>
kubectl get svc
kubectl describe ingress
```

Test internally:

```bash
kubectl exec -it <pod> -- curl backend-svc
```

---

# 🧠 DevOps Mindset

---

## 🔑 Think in Flow

Always think:

```text
User → Ingress → Service → Pod
```

---

## 🔑 Debug Backwards

Start from user request → go layer by layer

---

## 🔑 Build Incrementally

* Pod
* Deployment
* Service
* Ingress

---

## 🔑 Expect Failures

Failure is part of the process — debugging is the skill.

---

## 🔑 Focus on Networking

Most issues are:

* Wrong ports
* Wrong labels
* Wrong paths

---

# 💡 Tips & Tricks

* Use `kubectl describe` frequently
* Validate labels carefully
* Test services before ingress
* Use port-forward for quick testing:

```bash
kubectl port-forward svc/frontend-svc 8080:80
```

---

# 📈 Future Enhancements

* Add database with persistent storage
* Add HTTPS (TLS)
* Add autoscaling (HPA)
* Add monitoring (Prometheus + Grafana)
* Deploy on cloud (EKS/GKE/AKS)

---

# 🏁 Conclusion

This project helps you move from:

👉 “I understand Kubernetes basics”
to
👉 “I can build and debug real systems”

---

# 📌 Resume Line

> Built a multi-service Kubernetes application with Ingress-based routing, implementing deployments, services, and real-world debugging practices.

---

## 🙌 Final Advice

Build → Break → Debug → Repeat

That’s the real DevOps journey 🚀

---
