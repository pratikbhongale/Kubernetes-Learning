# Kubernetes Service – ClusterIP (Complete Guide)

## 📌 Overview

`ClusterIP` is the **default Service type** in Kubernetes.

It provides:

* A **stable virtual IP**
* Internal-only communication
* Load balancing across Pods
* Built-in DNS-based service discovery

ClusterIP is the foundation of microservices communication inside Kubernetes.

---

# 1️⃣ Why ClusterIP Exists

## Problem

Pods are **ephemeral**:

* They get dynamic IPs
* If a Pod restarts → IP changes
* Other Pods cannot rely on Pod IPs

Example:

```
Pod A → 10.244.1.5
Pod B → 10.244.1.9
```

If Pod B restarts → new IP assigned.

## Solution

ClusterIP creates:

* A **stable virtual IP**
* A **stable DNS name**
* Automatic load balancing

---

# 2️⃣ How ClusterIP Works Internally

When you create a ClusterIP Service:

1. Kubernetes assigns a virtual IP (e.g., 10.96.0.15)
2. kube-proxy watches the Service
3. kube-proxy creates iptables/IPVS rules
4. Traffic is forwarded to matching Pods
5. Load balancing happens (round-robin)

Flow:

```
Client Pod
    ↓
ClusterIP (Virtual IP)
    ↓
kube-proxy
    ↓
One backend Pod
```

⚠ Important: ClusterIP is NOT a real network interface.
It is managed via iptables or IPVS rules.

---

# 3️⃣ Basic YAML Example

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  type: ClusterIP   # Optional (default)
  selector:
    app: backend
  ports:
    - port: 80
      targetPort: 8080
```

---

# 4️⃣ Important Fields Explained

### `selector`

Matches Pod labels.

If Pods have:

```yaml
labels:
  app: backend
```

Service must use:

```yaml
selector:
  app: backend
```

---

### `port`

Port exposed by Service (used by clients).

### `targetPort`

Container port inside the Pod.

Example:

```
Client → backend-service:80
Service → PodIP:8080
```

---

### `protocol`

Default = TCP

Only define if using UDP or SCTP.

```yaml
protocol: UDP
```

---

# 5️⃣ Deployment + Service Example (Complete Setup)

## Step 1: Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-deployment
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
        image: nginx
        ports:
        - containerPort: 8080
```

Apply:

```bash
kubectl apply -f deployment.yaml
```

---

## Step 2: Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  selector:
    app: backend
  ports:
    - port: 80
      targetPort: 8080
```

Apply:

```bash
kubectl apply -f service.yaml
```

---

# 6️⃣ How to Test ClusterIP

⚠ You CANNOT access ClusterIP from your laptop directly.

You must test from inside the cluster.

---

## Method 1: Exec Into Existing Pod

```bash
kubectl get pods
kubectl exec -it <pod-name> -- sh
curl http://backend-service:80
```

---

## Method 2: Create Temporary Debug Pod

```bash
kubectl run test --image=busybox -it --rm -- sh
```

Inside:

```bash
wget -qO- http://backend-service:80
```

---

## Method 3: Port Forward

```bash
kubectl port-forward service/backend-service 8080:80
```

From your laptop:

```bash
curl http://localhost:8080
```

---

# 7️⃣ DNS & Service Discovery

Kubernetes automatically creates DNS records using **CoreDNS**.

Service FQDN format:

```
<service-name>.<namespace>.svc.cluster.local
```

Example:

```
backend-service.default.svc.cluster.local
```

Inside same namespace:

```
http://backend-service
```

Cross-namespace:

```
http://backend-service.dev
```

---

# 8️⃣ Headless Service (Advanced Concept)

If you define:

```yaml
clusterIP: None
```

It becomes a Headless Service.

It:

* Does NOT load balance
* Returns Pod IPs directly
* Used in StatefulSets
* Used in databases

Example:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: db-headless
spec:
  clusterIP: None
  selector:
    app: db
  ports:
    - port: 5432
```

---

# 9️⃣ Debugging ClusterIP (Step-by-Step)

If service is not working:

### 1. Check Service

```bash
kubectl get svc
```

---

### 2. Check Endpoints

```bash
kubectl get endpoints backend-service
```

If you see:

```
<none>
```

→ Selector mismatch.

---

### 3. Check Pod Labels

```bash
kubectl get pods --show-labels
```

Labels must match Service selector.

---

### 4. Check DNS

Inside pod:

```bash
nslookup backend-service
```

---

### 5. Check NetworkPolicy

If using NetworkPolicy, ensure traffic is allowed.

---

# 🔟 Strategies & Best Practices

## ✅ Strategy 1: Use ClusterIP for Internal Services

Always keep:

* Databases
* Backend APIs
* Caches
* Message brokers

As ClusterIP.

Expose only entry services externally.

---

## ✅ Strategy 2: Label Design Strategy

Use structured labels:

```yaml
labels:
  app: payment
  tier: backend
  env: prod
```

Service selector can target specific tiers:

```yaml
selector:
  app: payment
  tier: backend
```

This supports:

* Blue/Green deployments
* Canary releases
* Version routing

---

## ✅ Strategy 3: Use Named Ports

Instead of numbers:

Deployment:

```yaml
ports:
  - name: http
    containerPort: 8080
```

Service:

```yaml
targetPort: http
```

Prevents breaking changes if port numbers change.

---

## ✅ Strategy 4: Use Session Affinity (If Needed)

For sticky sessions:

```yaml
spec:
  sessionAffinity: ClientIP
```

Useful for legacy apps.

---

## ✅ Strategy 5: Avoid Direct Pod IP Usage

Never use Pod IPs in:

* Config files
* Code
* Environment variables

Always use Service DNS.

---

# 1️⃣1️⃣ Important Interview Concepts

You must know:

* ClusterIP is default
* Only accessible inside cluster
* Uses kube-proxy
* Load balances traffic
* Uses DNS via CoreDNS
* Can expose multiple ports
* Headless services exist
* Uses selectors to map Pods

---

# 1️⃣2️⃣ Common Mistakes

❌ Selector does not match Pod labels
❌ Wrong targetPort
❌ Testing from outside cluster
❌ Forgetting namespace
❌ NetworkPolicy blocking traffic

---

# 1️⃣3️⃣ Real Production Architecture Example

```
Frontend (Ingress)
        ↓
frontend-service (ClusterIP)
        ↓
backend-service (ClusterIP)
        ↓
database-service (ClusterIP)
```

Only Ingress or LoadBalancer is public.

Everything else remains internal.

---

# 1️⃣4️⃣ Key Commands Summary

```bash
kubectl get svc
kubectl describe svc backend-service
kubectl get endpoints
kubectl get pods --show-labels
kubectl exec -it <pod> -- sh
kubectl port-forward service/backend-service 8080:80
```

---

# 🎯 Final Understanding

ClusterIP is:

* The backbone of Kubernetes networking
* Used in almost every microservice architecture
* Required for stable internal communication
* DNS-enabled
* Automatically load-balanced

Mastering ClusterIP means you understand:

* Pod networking
* Service discovery
* Internal traffic routing
* Debugging microservices
