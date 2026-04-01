# Kubernetes Ingress – Complete Beginner to Strong Fundamentals

## Table of Contents

1. [Introduction](#introduction)
2. [Why Ingress is Needed](#why-ingress-is-needed)
3. [Ingress vs Service](#ingress-vs-service)
4. [Architecture & Flow](#architecture--flow)
5. [Core Concepts](#core-concepts)
6. [Hands-On Examples](#hands-on-examples)
7. [Commands Cheat Sheet](#commands-cheat-sheet)
8. [Exercises](#exercises)
9. [Interview Questions & Solutions](#interview-questions--solutions)
10. [Scenario-Based Questions & Solutions](#scenario-based-questions--solutions)
11. [Tips & Tricks](#tips--tricks)
12. [Common Errors & Debugging](#common-errors--debugging)
13. [References](#references)

---

## Introduction

**Ingress** in Kubernetes is a resource that allows external HTTP/HTTPS traffic to reach services inside a cluster. It acts as a **smart traffic router**, enabling:

* Single entry point for multiple services
* Path-based or host-based routing
* TLS/HTTPS management
* Clean, scalable URL management

Ingress requires an **Ingress Controller** (like NGINX) to work; defining an Ingress resource alone is not enough.

---

## Why Ingress is Needed

Without Ingress, exposing multiple services requires:

* NodePort (one port per service)
* LoadBalancer (one external IP per service)

Challenges:

* Multiple ports → hard to manage
* Not user-friendly
* Not production-ready

**Ingress solves this by routing HTTP(S) traffic to multiple services via a single IP.**

---

## Ingress vs Service

| Feature              | Service                           | Ingress                           |
| -------------------- | --------------------------------- | --------------------------------- |
| Purpose              | Expose pods internally/externally | Route HTTP(S) traffic to services |
| Access Layer         | TCP/UDP                           | HTTP/HTTPS only                   |
| Single Entry Point   | No                                | Yes                               |
| Routing by Path/Host | No                                | Yes                               |
| Type                 | ClusterIP, NodePort, LoadBalancer | Resource + Controller             |

**Example**

Without Ingress:

```
http://192.168.49.2:30001 -> App1
http://192.168.49.2:30002 -> App2
```

With Ingress:

```
http://myapp.local/app1 -> App1
http://myapp.local/app2 -> App2
```

---

## Architecture & Flow

```
Browser/User
      |
      v
Ingress Controller (NGINX)
      |---- /      ---> Service A -> Pod A
      |---- /app2  ---> Service B -> Pod B
```

**Flow Explanation:**

1. Browser sends request to `myapp.local`
2. Ingress Controller checks rules (host/path)
3. Routes to correct **Service** (ClusterIP)
4. Service forwards traffic to Pod
5. Pod responds → Browser

---

## Core Concepts

1. **Ingress Resource** – Defines rules for routing HTTP traffic.
2. **Ingress Controller** – Implements routing. Must be installed.
3. **Host-based Routing** – Different domains/subdomains → different services.
4. **Path-based Routing** – `/app1`, `/app2` → different services.
5. **TLS/HTTPS** – Handled centrally at Ingress level.
6. **Namespace** – Ingress and services must reside in the same namespace.

---

## Hands-On Examples

### Step 1: Enable Ingress in Minikube

```bash
minikube addons enable ingress
```

---

### Step 2: Create Namespace

```bash
kubectl create namespace nginx
```

---

### Step 3: Deployment YAML

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  namespace: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

---

### Step 4: Service YAML

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
  namespace: nginx
spec:
  selector:
    app: my-app
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP
```

---

### Step 5: Ingress YAML

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
  namespace: nginx
spec:
  rules:
  - host: myapp.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-service
            port:
              number: 80
```

---

### Step 6: Update /etc/hosts

```text
192.168.49.2 myapp.local
```

---

### Step 7: Test

```bash
kubectl get all -n nginx
kubectl describe ingress my-ingress -n nginx
```

Browser → `http://myapp.local`

✅ NGINX page should appear

---

## Commands Cheat Sheet

```bash
kubectl create namespace nginx
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f ingress.yaml
kubectl get ingress -n nginx
kubectl describe ingress my-ingress -n nginx
kubectl logs -n kube-system -l app.kubernetes.io/name=ingress-nginx
kubectl get pods -n nginx
```

---

## Exercises

1. Create another app (`my-app-2`) and a service `my-service-2`.
2. Update Ingress to route `/app2` → `my-service-2`.
3. Test browser access using path-based routing.
4. Add a PVC for one app and store some static files to serve via NGINX.

✅ Goal: Practice **Deployment → Service → Ingress → Browser** flow.

---

## Interview Questions & Solutions

**Q1:** Difference between NodePort, LoadBalancer, and Ingress?
**A1:** NodePort exposes a service on a port; LoadBalancer gives an external IP; Ingress routes HTTP(S) traffic to multiple services via one entry point.

**Q2:** Can Ingress route TCP traffic?
**A2:** No, Ingress is for HTTP/HTTPS. Use NodePort/LoadBalancer for TCP/UDP.

**Q3:** Why use ClusterIP with Ingress?
**A3:** Ingress routes traffic to services via ClusterIP internally. NodePort is not needed.

---

## Scenario-Based Questions & Solutions

**Scenario:** You have 3 microservices (web, api, auth). You want all of them under one domain using paths:

* `/` → web
* `/api` → api
* `/auth` → auth

**Solution:**

```yaml
rules:
- host: myapp.local
  http:
    paths:
    - path: /
      backend: web-service
    - path: /api
      backend: api-service
    - path: /auth
      backend: auth-service
```

**Tip:** When asked in an interview, always explain:

* One IP
* Path-based routing
* ClusterIP services
* Ingress controller required

---

## Tips & Tricks

1. Always specify **namespace** for Ingress + Service + Deployment.
2. Use **`kubectl describe ingress`** to debug routing issues.
3. Use **`kubectl logs -n kube-system`** for Ingress Controller logs.
4. For testing, edit **`/etc/hosts`** to map host to Minikube IP.
5. Always match **host header** in browser with Ingress rules.
6. Start simple: one app → multiple apps → TLS → real-world scenarios.

---

## Common Errors & Debugging

| Error                     | Cause                        | Fix                                         |
| ------------------------- | ---------------------------- | ------------------------------------------- |
| Site cannot be reached    | Browser host header mismatch | Check `/etc/hosts` entry                    |
| Ingress rules not applied | Ingress controller missing   | `minikube addons enable ingress`            |
| Pod not reachable         | Service name/port mismatch   | Verify Service name & port                  |
| TLS errors                | Certificate misconfigured    | Add secret and update Ingress `tls` section |

---

## Additional Concepts

* **TLS/HTTPS in Ingress**
* **Annotations for NGINX Ingress** (rewrite, rate-limit)
* **Multiple host routing** (subdomain routing)
* **Ingress vs Gateway (for advanced)**

---

## Summary – Flow Recap

```
Browser → Ingress Controller → Service → Pod → Browser
```

* Pod hosts app
* Service exposes Pod internally (ClusterIP)
* Ingress routes HTTP(S) traffic to multiple services using **host/path rules**
* Ingress Controller (NGINX) handles routing

With this README, you can:

* Understand theory
* Practice hands-on
* Solve exercises
* Prepare for interviews

---

This README gives **beginner → intermediate coverage** for Ingress with **all YAMLs, commands, tips, exercises, and interview prep**.

---
