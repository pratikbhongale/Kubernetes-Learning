
# Kubernetes Networking – LoadBalancer Deep Dive

## Table of Contents

1. [Introduction](#introduction)
2. [Service Types Recap](#service-types-recap)
3. [What is a LoadBalancer Service](#what-is-a-loadbalancer-service)
4. [Traffic Flow & Architecture](#traffic-flow--architecture)
5. [Key Fields in LoadBalancer YAML](#key-fields-in-loadbalancer-yaml)
6. [Strategies & Session Affinity](#strategies--session-affinity)
7. [External IP & Minikube Tunnel](#external-ip--minikube-tunnel)
8. [Hands-On Examples](#hands-on-examples)

   * Node.js App
   * Python Flask App
   * Redis Service
   * HTTP Echo Server
   * Microservice Backend
9. [Scaling & Testing Load Balancing](#scaling--testing-load-balancing)
10. [Tips & Tricks](#tips--tricks)
11. [Common Issues & Debugging](#common-issues--debugging)
12. [Summary](#summary)

---

## Introduction

Kubernetes Services are used to **expose pods** to other pods or to external traffic. Among the service types:

* **ClusterIP** → internal only
* **NodePort** → exposes via node IP
* **LoadBalancer** → exposes externally with automatic load balancing

This guide focuses on **LoadBalancer Services**, including practical Minikube examples.

---

## Service Types Recap

| Service Type | Accessibility | Notes                                                                       |
| ------------ | ------------- | --------------------------------------------------------------------------- |
| ClusterIP    | Internal only | Default; for inter-pod communication                                        |
| NodePort     | NodeIP:Port   | Useful for testing or local exposure                                        |
| LoadBalancer | External IP   | Automatically exposes service to external traffic; often used in production |

---

## What is a LoadBalancer Service

* Exposes a Kubernetes Service **to external traffic**.
* Automatically distributes traffic across multiple pods.
* In cloud environments, Kubernetes creates a **cloud load balancer** automatically.
* In Minikube or bare-metal clusters, additional configuration is needed (e.g., `minikube tunnel` or MetalLB).

**Use Cases:**

* Web applications
* Public APIs
* SaaS dashboards
* Mobile backend services

---

## Traffic Flow & Architecture

### Cloud Environment:

```text
Client → Cloud Load Balancer → Service → Pods
```

### Minikube (Local):

```text
Client → Minikube Tunnel → Service → Pods
```

### Internal Flow:

```text
External-IP → NodePort → ClusterIP → Pod TargetPort
```

---

## Key Fields in LoadBalancer YAML

```yaml
spec:
  type: LoadBalancer
  selector:
    app: myapp
  ports:
    - port: 80          # Exposed service port
      targetPort: 8080  # Container application port
      nodePort: 30007   # NodePort (internal, optional)
```

| Field           | Purpose                              |
| --------------- | ------------------------------------ |
| type            | LoadBalancer creates external LB     |
| selector        | Matches pods                         |
| port            | Service port (entry point)           |
| targetPort      | Container port                       |
| nodePort        | Node port for NodePort routing       |
| sessionAffinity | (Optional) Stick traffic to same pod |

---

## Strategies & Session Affinity

### Load Balancing Strategies

1. **Round Robin** – Default, evenly distributes traffic across pods.
2. **Random** – Rarely used.
3. **Hash-based** – Distributes requests based on IP or headers.

### Session Affinity (Sticky Sessions)

```yaml
sessionAffinity: ClientIP
```

* Ensures traffic from the same client always hits the same pod.
* Useful for login sessions or carts.

---

## External IP & Minikube Tunnel

* In cloud: automatically assigned by the provider.
* In Minikube (local), `EXTERNAL-IP` shows `<pending>` by default.

**Command:**

```bash
minikube tunnel
```

* Simulates cloud load balancer locally.
* Must run continuously.

Check services:

```bash
kubectl get svc
```

Access your app:

```bash
http://<EXTERNAL-IP>
```

---

## Hands-On Examples

### 1️⃣ Node.js Express App

**Deployment**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: node-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: node-app
  template:
    metadata:
      labels:
        app: node-app
    spec:
      containers:
      - name: node-app
        image: node:18
        command: ["node", "-e"]
        args:
          - >
            const http = require('http');
            const PORT = 3000;
            const os = require('os');
            http.createServer((req, res) => {
              res.end(`Hello from ${os.hostname()}\n`);
            }).listen(PORT);
        ports:
        - containerPort: 3000
```

**Service**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: node-app-lb
spec:
  type: LoadBalancer
  selector:
    app: node-app
  ports:
    - port: 80
      targetPort: 3000
```

---

### 2️⃣ Python Flask App

**Deployment**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: flask-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: flask-app
  template:
    metadata:
      labels:
        app: flask-app
    spec:
      containers:
      - name: flask-app
        image: tiangolo/uwsgi-nginx-flask:python3.11
        ports:
        - containerPort: 80
```

**Service**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: flask-lb
spec:
  type: LoadBalancer
  selector:
    app: flask-app
  ports:
    - port: 8080
      targetPort: 80
```

---

### 3️⃣ Redis Service

**Deployment**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis
        image: redis:7
        ports:
        - containerPort: 6379
```

**Service**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: redis-lb
spec:
  type: LoadBalancer
  selector:
    app: redis
  ports:
    - port: 6379
      targetPort: 6379
```

---

### 4️⃣ HTTP Echo Server

**Deployment**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: echo-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: echo-app
  template:
    metadata:
      labels:
        app: echo-app
    spec:
      containers:
      - name: echo
        image: hashicorp/http-echo
        args:
          - "-text=Hello from $(HOSTNAME)"
        ports:
        - containerPort: 5678
```

**Service**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: echo-lb
spec:
  type: LoadBalancer
  selector:
    app: echo-app
  ports:
    - port: 80
      targetPort: 5678
```

---

### 5️⃣ Microservice Backend

**Deployment**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
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
          - "-text=Hello from backend $(HOSTNAME)"
        ports:
        - containerPort: 8080
```

**Service**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-lb
spec:
  type: LoadBalancer
  selector:
    app: backend
  ports:
    - port: 80
      targetPort: 8080
```

---

## Scaling & Testing Load Balancing

1. Scale replicas:

```bash
kubectl scale deployment <deployment-name> --replicas=5
```

2. Curl the external IP multiple times:

```bash
curl http://<EXTERNAL-IP>
```

3. Optional: Enable **sessionAffinity**:

```yaml
sessionAffinity: ClientIP
```

* Forces same client to hit the same pod.

---

## Tips & Tricks

* Use **`kubectl get pods -o wide`** to see pod IPs and node assignment.
* Use **lightweight apps** (http-echo) to visualize LoadBalancer distribution.
* **`minikube service <service-name>`** opens the LB service in your browser.
* Keep **`minikube tunnel`** running when testing LoadBalancer.
* Use **`kubectl describe svc <service>`** to debug LoadBalancer events.

---

## Common Issues & Debugging

| Issue                       | Cause                   | Solution                                                     |
| --------------------------- | ----------------------- | ------------------------------------------------------------ |
| EXTERNAL-IP `<pending>`     | No cloud LB             | Run `minikube tunnel` (local) or deploy MetalLB (bare metal) |
| Traffic not hitting pods    | Wrong selector          | Check `kubectl get pods --show-labels`                       |
| Sticky sessions not working | SessionAffinity not set | Add `sessionAffinity: ClientIP`                              |
| Port mapping mismatch       | `port` vs `targetPort`  | Ensure containerPort matches targetPort in Service           |

---

## Summary

By learning LoadBalancer services, you now understand:

* How Kubernetes exposes services externally
* Traffic flow from client → LB → NodePort → ClusterIP → Pods
* port vs targetPort vs nodePort
* sessionAffinity & sticky sessions
* Minikube tunnel & local testing
* Hands-on deployments with Node.js, Flask, Redis, and echo servers
* Scaling, testing, and debugging LoadBalancer traffic

This provides both **strong theoretical knowledge** and **practical hands-on experience**, preparing you for **real Kubernetes projects and interviews**.

---


