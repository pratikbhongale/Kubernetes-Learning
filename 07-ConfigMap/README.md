# Kubernetes ConfigMaps — Complete Guide

## Table of Contents

1. Introduction
2. Why ConfigMaps Are Needed
3. What is a ConfigMap
4. ConfigMap Architecture
5. Creating ConfigMaps
6. Using ConfigMaps in Pods
7. ConfigMaps as Environment Variables
8. ConfigMaps as Volumes (Files)
9. ConfigMap with Deployments
10. Important Commands
11. Strategies & Best Practices
12. Beginner Exercises
13. Interview Questions
14. Scenario Based Questions
15. Debugging & Common Errors
16. Tips for Real Projects
17. Summary

---

# 1. Introduction

In Kubernetes, applications often require **configuration values** such as:

* Environment
* Service URLs
* Logging levels
* Feature flags
* Application settings

Example:

```
APP_ENV=production
APP_COLOR=blue
LOG_LEVEL=debug
```

Hardcoding these values inside container images is a bad practice.

Kubernetes solves this problem using **ConfigMaps**.

---

# 2. Why ConfigMaps Are Needed

Without ConfigMaps:

```
Application Code
      +
Configuration
      =
Container Image
```

If configuration changes:

* Rebuild image
* Push image
* Redeploy application

This is inefficient.

---

### With ConfigMaps

```
Container Image
      +
ConfigMap
      =
Running Pod
```

Benefits:

* Configuration separated from application
* No image rebuild required
* Same image works in dev/staging/prod

---

# 3. What is a ConfigMap

A **ConfigMap** is a Kubernetes object used to store **non-sensitive configuration data** as **key-value pairs**.

Example:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_COLOR: blue
  APP_ENV: production
```

This data can be injected into Pods as:

1. Environment variables
2. Files
3. Command arguments

---

# 4. ConfigMap Architecture

```
ConfigMap
   │
   │ (Configuration values)
   ▼
Kubernetes Pod
   │
   ▼
Container Application
```

The application reads configuration **at runtime**.

---

# 5. Creating ConfigMaps

## Method 1: Using YAML

Create file:

`configmap.yaml`

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_COLOR: blue
  APP_ENV: production
```

Apply it:

```bash
kubectl apply -f configmap.yaml
```

Verify:

```bash
kubectl get configmaps
```

---

## Method 2: Using Command Line

```bash
kubectl create configmap app-config \
  --from-literal=APP_COLOR=blue \
  --from-literal=APP_ENV=production
```

---

## Method 3: From File

Create file:

```
config.properties
```

```
APP_COLOR=blue
APP_ENV=production
```

Command:

```bash
kubectl create configmap app-config --from-file=config.properties
```

---

# 6. Viewing ConfigMaps

List ConfigMaps:

```
kubectl get configmaps
```

Detailed view:

```
kubectl describe configmap app-config
```

YAML format:

```
kubectl get configmap app-config -o yaml
```

JSON format:

```
kubectl get configmap app-config -o json
```

---

# 7. Using ConfigMap in a Pod (Environment Variables)

Example Pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-pod
spec:
  containers:
  - name: demo-container
    image: nginx
    env:
    - name: APP_COLOR
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: APP_COLOR
```

Apply:

```
kubectl apply -f pod.yaml
```

Check environment variables:

```
kubectl exec -it configmap-pod -- env
```

Expected output:

```
APP_COLOR=blue
```

---

# 8. Import All Values Using envFrom

Instead of specifying each key:

```yaml
envFrom:
- configMapRef:
    name: app-config
```

Example:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-pod2
spec:
  containers:
  - name: demo-container
    image: nginx
    envFrom:
    - configMapRef:
        name: app-config
```

All variables will be available.

---

# 9. Using ConfigMap as Files (Volume)

Some applications read config files.

Example ConfigMap:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: file-config
data:
  app.properties: |
    APP_COLOR=blue
    APP_ENV=production
```

Pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-volume-pod
spec:
  containers:
  - name: demo-container
    image: nginx
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config

  volumes:
  - name: config-volume
    configMap:
      name: file-config
```

Verify:

```
kubectl exec -it configmap-volume-pod -- ls /etc/config
```

View file:

```
kubectl exec -it configmap-volume-pod -- cat /etc/config/app.properties
```

---

# 10. ConfigMap with Deployment (Real Usage)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: nginx
        image: nginx
        envFrom:
        - configMapRef:
            name: app-config
```

Apply:

```
kubectl apply -f deployment.yaml
```

---

# 11. Strategies & Best Practices

### 1. Separate Configuration

Do not hardcode values in images.

Bad:

```
ENV APP_ENV=production
```

Good:

Use ConfigMap.

---

### 2. Use Different ConfigMaps Per Environment

```
app-config-dev
app-config-staging
app-config-prod
```

---

### 3. Keep Secrets Separate

Never store sensitive data in ConfigMaps.

Use **Secrets** instead.

---

### 4. Use envFrom for Simplicity

Cleaner YAML.

---

### 5. Use ConfigMap Versioning

Example:

```
app-config-v1
app-config-v2
```

Helps avoid breaking changes.

---

# 12. Beginner Exercises

## Exercise 1

Create ConfigMap:

```
APP_ENV=dev
APP_NAME=test-app
```

Tasks:

1. Create ConfigMap YAML
2. Deploy Pod using it
3. Verify environment variables

---

## Exercise 2

Create ConfigMap from file:

```
app.properties
```

Mount it inside container at:

```
/config
```

Verify inside container.

---

## Exercise 3

Create Deployment with:

* 3 replicas
* ConfigMap environment variables

Check if all pods receive variables.

---

# 13. Interview Questions

### Q1: What is a ConfigMap?

**Answer**

A ConfigMap stores non-sensitive configuration data in key-value format and allows injecting configuration into Pods without rebuilding container images.

---

### Q2: Ways to use ConfigMaps?

Answer:

1. Environment variables
2. Environment variable groups
3. Volume mounted files
4. Command arguments

---

### Q3: Difference between ConfigMap and Secret?

| ConfigMap     | Secret             |
| ------------- | ------------------ |
| Non-sensitive | Sensitive data     |
| Plain text    | Base64 encoded     |
| Config values | Passwords/API keys |

---

# 14. Scenario Based Questions

## Scenario 1

Your application has:

```
APP_ENV
LOG_LEVEL
SERVICE_URL
```

These values change between dev and production.

### Solution

Store them in ConfigMap and load using:

```
envFrom:
  configMapRef:
```

Use different ConfigMaps for environments.

---

## Scenario 2

Your Nginx container needs a custom `nginx.conf`.

Solution:

Mount ConfigMap as a volume.

---

# 15. How to Answer Scenario Questions (Interview Tip)

Use this structure:

### Step 1

Identify problem.

Example:

"Application configuration changes between environments."

### Step 2

Choose solution.

"Use ConfigMap to externalize configuration."

### Step 3

Explain implementation.

```
Create ConfigMap → Inject into Pod → Use env/envFrom
```

### Step 4

Give YAML example.

This shows **practical experience**.

---

# 16. Common Errors

## Error 1

```
ConfigMap not found
```

Cause:

Pod referencing non-existent ConfigMap.

Fix:

```
kubectl get configmaps
```

---

## Error 2

Wrong key name.

Example:

```
key: APP_COLOR
```

But ConfigMap has:

```
APP_COLOUR
```

Fix: match key exactly.

---

## Error 3

Pod crash due to missing config.

Debug:

```
kubectl describe pod pod-name
```

---

## Error 4

Env variables not visible.

Check:

```
kubectl exec -it pod-name -- env
```

---

# 17. Debugging Workflow

Step 1

```
kubectl get configmaps
```

Step 2

```
kubectl describe configmap app-config
```

Step 3

```
kubectl describe pod pod-name
```

Step 4

```
kubectl exec -it pod-name -- env
```

---

# 18. Real Project Example

Microservice configuration:

```
PORT=8080
REDIS_HOST=redis-service
PAYMENT_SERVICE=http://payment-service
LOG_LEVEL=debug
```

Stored in ConfigMap.

Application reads them as environment variables.

---

# 19. Important Kubernetes Principle

ConfigMaps help follow the **12-Factor App principle**:

```
Store configuration in environment variables
```

This makes applications **cloud-native**.

---

# 20. Summary

ConfigMaps allow:

* Separation of configuration from code
* Environment flexibility
* Easy updates
* Cleaner Kubernetes architecture

Common usage:

* Environment variables
* Configuration files
* Deployment configuration

---

# Final Tip

A strong Kubernetes engineer should know:

* ConfigMap creation
* Injecting into Pods
* Mounting as volumes
* Debugging configuration issues

Mastering these makes **real production deployments much easier**.

```

This is **how DevOps engineers build portfolio repos**, and it looks great in interviews.
