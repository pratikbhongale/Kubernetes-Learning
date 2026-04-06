# 📘 Kubernetes Resource Management – Complete Beginner to Interview Guide

---

# 🎯 Objective

This guide will help you:

* Understand **Kubernetes Resource Management** from basics
* Gain **hands-on skills**
* Learn **debugging techniques**
* Prepare for **interviews & real-world scenarios**

---

# 📦 1. What is Resource Management?

Resource management in Kubernetes ensures that:

* Applications get the resources they need
* No single pod consumes everything
* Cluster remains stable and efficient

---

# ⚙️ 2. Core Concepts

## 🔹 Resources

* CPU (millicores: `100m = 0.1 CPU`)
* Memory (Mi, Gi)

---

## 🔹 Requests vs Limits

```yaml
resources:
  requests:
    cpu: "100m"
    memory: "128Mi"
  limits:
    cpu: "200m"
    memory: "256Mi"
```

| Type     | Meaning            |
| -------- | ------------------ |
| Requests | Minimum guaranteed |
| Limits   | Maximum allowed    |

---

## 🧠 Behavior

| Resource | If Exceeded            |
| -------- | ---------------------- |
| CPU      | Throttled (slow)       |
| Memory   | Pod killed (OOMKilled) |

---

# 🧪 3. Basic Pod Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: basic-pod
spec:
  containers:
  - name: nginx
    image: nginx
    resources:
      requests:
        cpu: "100m"
        memory: "128Mi"
      limits:
        cpu: "200m"
        memory: "256Mi"
```

---

# 🚀 4. Deployment Example

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        resources:
          requests:
            cpu: "100m"
            memory: "128Mi"
          limits:
            cpu: "200m"
            memory: "256Mi"
```

---

# 🏗️ 5. Namespace + ResourceQuota

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: resource-demo
```

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: demo-quota
  namespace: resource-demo
spec:
  hard:
    pods: "5"
    requests.cpu: "1"
    requests.memory: 1Gi
```

---

# 🔍 6. Important Commands

## ✅ Create resources

```bash
kubectl apply -f file.yaml
```

## ✅ View pods

```bash
kubectl get pods
kubectl get pods -n resource-demo
```

## ✅ Describe pod (MOST IMPORTANT)

```bash
kubectl describe pod <pod-name>
```

## ✅ View YAML

```bash
kubectl get pod <pod-name> -o yaml
```

## ✅ Check usage

```bash
kubectl top pod
kubectl top node
```

## ✅ Logs

```bash
kubectl logs <pod-name>
```

## ✅ Exec into pod

```bash
kubectl exec -it <pod-name> -- sh
```

---

# 📊 7. Debugging Guide

## 🧭 Step-by-step

1. Check pod status

```bash
kubectl get pods
```

2. Describe pod

```bash
kubectl describe pod <pod-name>
```

3. Check logs

```bash
kubectl logs <pod-name>
```

4. Check usage

```bash
kubectl top pod
```

---

# 🚨 8. Common Errors & Fixes

---

## ❌ OOMKilled

### Cause:

Memory exceeded limit

### Debug:

```bash
kubectl describe pod <pod-name>
```

### Fix:

Increase memory limit

---

## ❌ Pod Pending

### Cause:

Not enough resources

### Error:

```
Insufficient memory/cpu
```

### Fix:

* Reduce requests
* Add nodes

---

## ❌ CPU Throttling

### Cause:

Low CPU limit

### Fix:

Increase CPU limit

---

## ❌ CrashLoopBackOff

### Cause:

* App crash
* Resource issue

### Debug:

```bash
kubectl logs <pod-name>
```

---

# 🧠 9. Strategies & Tricks

✔ Always define requests & limits
✔ Keep memory limit slightly higher than request
✔ Avoid very low CPU limits
✔ Use `kubectl describe` first when debugging
✔ Compare **usage vs limits**

---

# 🧪 10. Hands-On Exercises

---

## 🟢 Exercise 1: Basic Pod

* Create a pod with CPU & memory limits
* Verify using `describe`

---

## 🟢 Exercise 2: Trigger OOM

* Set memory limit to 100Mi
* Run stress container
* Observe crash

---

## 🟢 Exercise 3: Pending Pod

* Set memory request to 10Gi
* Observe scheduling failure

---

## 🟢 Exercise 4: Deployment

* Create deployment with 2 replicas
* Verify resources on each pod

---

# 🎯 11. Interview Questions

---

## ❓ What are requests and limits?

✅ Answer:
Requests define minimum guaranteed resources, limits define maximum usage allowed.

---

## ❓ What happens if memory exceeds limit?

✅ Answer:
Pod is killed with OOMKilled error.

---

## ❓ What happens if CPU exceeds limit?

✅ Answer:
CPU is throttled, application slows down.

---

## ❓ How do you check resource usage?

✅ Answer:
Using:

```bash
kubectl top pod
```

---

# 🧠 12. Scenario-Based Questions

---

## 🔴 Scenario 1

**Pod is restarting continuously**

### Solution:

* Check:

```bash
kubectl describe pod
```

* Look for:

```
OOMKilled
```

* Fix: Increase memory

---

## 🔴 Scenario 2

**Pod stuck in Pending**

### Solution:

* Check:

```bash
kubectl describe pod
```

* Error:

```
Insufficient resources
```

* Fix:

  * Reduce request
  * Add node

---

## 🔴 Scenario 3

**Application is slow**

### Solution:

* Check CPU usage

```bash
kubectl top pod
```

* Increase CPU limit

---

# 🎤 13. How to Answer Scenario in Interview (With Demo)

### Structure:

1. Identify problem
2. Check pod status
3. Use describe
4. Analyze resource issue
5. Fix configuration

---

## 💬 Example Answer

> “First, I check pod status using `kubectl get pods`. Then I describe the pod to check events. If I see OOMKilled, I increase memory limits. I verify using `kubectl top pod`.”

---

# 🧱 14. Important Concepts You Should Not Miss

* Requests = scheduling decision
* Limits = runtime restriction
* Memory issues = crashes
* CPU issues = performance
* `kubectl describe` = primary debugging tool

---

# 🧠 15. Final Mental Model

Think like this:

* Requests → Reservation
* Limits → Cap
* Describe → Debug tool
* Top → Real usage

---

# 🚀 Final Advice

If you master:

* Writing YAML with resources
* Debugging using `describe`
* Understanding failures

👉 You are **interview-ready for resource management**

---

# ✅ Next Steps

* Practice all exercises
* Simulate failures
* Explain concepts out loud

---

💡 This combination of **theory + hands-on + debugging** is what makes you strong in Kubernetes.
