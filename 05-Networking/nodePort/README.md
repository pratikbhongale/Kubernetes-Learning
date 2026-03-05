
# Kubernetes Service: NodePort — Complete Guide

A practical and conceptual guide to understanding **NodePort Services in Kubernetes** with theory, YAML examples, debugging techniques, commands, and hands-on exercises.

---

# 1. Introduction

In **Kubernetes**, Pods are ephemeral. Their IP addresses change whenever they restart or are recreated. Because of this, directly accessing Pods is unreliable.

To solve this problem, Kubernetes provides a **Service**, which acts as a stable network endpoint for Pods.

One of the most important Service types for beginners is **NodePort**.

A NodePort service allows external users to access applications running inside a Kubernetes cluster.

---

# 2. What is a Kubernetes Service?

A **Service** is an abstraction that exposes a set of Pods as a network service.

It provides:

* Stable IP address
* Stable DNS name
* Load balancing across Pods

Instead of connecting directly to Pods, clients connect to the Service.

Traffic flow:

Client → Service → Pod

---

# 3. What is NodePort?

A **NodePort Service** exposes your application on a static port on every Node in the cluster.

This allows external users to access applications running inside Kubernetes.

Access pattern:

NodeIP:NodePort

Example:

[http://192.168.1.10:30007](http://192.168.1.10:30007)

Traffic flow:

External User
↓
Node IP : NodePort
↓
Kubernetes Service
↓
Pod

---

# 4. NodePort Port Range

Kubernetes restricts NodePorts to a predefined range:

30000 – 32767

This avoids conflicts with common system ports.

Example NodePort:

30007
31543
32080

---

# 5. Ports Used in NodePort Services

NodePort services involve three different ports.

## 5.1 Service Port (port)

The port where the Service listens inside the cluster.

Example:

port: 80

Pods inside the cluster connect using:

service-name:80

---

## 5.2 TargetPort

The port on the Pod container where the application is running.

Example:

targetPort: 8080

Service forwards traffic to this port.

---

## 5.3 NodePort

The port exposed on every Kubernetes node.

Example:

nodePort: 30007

Users access the application via:

NodeIP:30007

---

# 6. Complete Traffic Flow

Example configuration:

nodePort: 30007
port: 80
targetPort: 8080

Flow:

User
↓
NodeIP:30007
↓
Service Port 80
↓
Pod Port 8080
↓
Application

---

# 7. Basic Deployment Example

Create a deployment running nginx.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
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
        ports:
        - containerPort: 80
```

Apply deployment:

```
kubectl apply -f deployment.yaml
```

Check Pods:

```
kubectl get pods
```

---

# 8. NodePort Service Example

Expose the deployment using a NodePort Service.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport-service
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30007
```

Apply:

```
kubectl apply -f service.yaml
```

Check service:

```
kubectl get svc
```

Example output:

```
NAME                     TYPE       CLUSTER-IP       PORT(S)
nginx-nodeport-service   NodePort   10.96.23.12      80:30007/TCP
```

---

# 9. Accessing the Application

Get node IP:

```
kubectl get nodes -o wide
```

Example:

```
192.168.49.2
```

Open in browser:

```
http://192.168.49.2:30007
```

You should see the nginx welcome page.

---

# 10. NodePort Without Specifying nodePort

Kubernetes can automatically assign a NodePort.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport-auto
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
```

Check assigned port:

```
kubectl get svc
```

Example:

```
80:31543/TCP
```

Here, 31543 is the NodePort.

---

# 11. Using Different Target Ports

Sometimes the container runs on a different port.

Deployment:

```yaml
containers:
- name: webapp
  image: nginx
  ports:
  - containerPort: 8080
```

Service:

```yaml
ports:
- port: 80
  targetPort: 8080
  nodePort: 30010
```

Traffic flow:

NodeIP:30010 → Service:80 → Pod:8080

---

# 12. Services Use Labels and Selectors

Pods are selected using labels.

Pod label:

```
app: nginx
```

Service selector:

```
selector:
  app: nginx
```

Service sends traffic to all Pods matching this label.

---

# 13. NodePort Architecture

External User
↓
Node IP : NodePort
↓
Service
↓
Load balancing
↓
Pod1
Pod2
Pod3

Kubernetes automatically distributes traffic between Pods.

---

# 14. Important Kubernetes Commands

List services:

```
kubectl get svc
```

Detailed service info:

```
kubectl describe svc service-name
```

Check Pods:

```
kubectl get pods
```

Check labels:

```
kubectl get pods --show-labels
```

Check endpoints:

```
kubectl get endpoints
```

Get node IP:

```
kubectl get nodes -o wide
```

Test service internally:

```
kubectl run testpod --image=busybox -it --rm -- sh
```

Inside the pod:

```
wget -qO- service-name:80
```

---

# 15. Debugging NodePort Services

When NodePort fails, check the following.

## Step 1: Check service

```
kubectl get svc
```

## Step 2: Check service configuration

```
kubectl describe svc service-name
```

## Step 3: Check pods

```
kubectl get pods
```

## Step 4: Check endpoints

```
kubectl get endpoints service-name
```

If endpoints show `<none>`, there is a label mismatch.

## Step 5: Check node IP

```
kubectl get nodes -o wide
```

Access service using NodeIP:NodePort.

---

# 16. Hands-On Practice Lab

## Exercise 1

Create a deployment with 3 nginx pods.

Verify:

```
kubectl get pods
```

---

## Exercise 2

Create a NodePort service exposing the deployment.

Verify:

```
kubectl get svc
```

---

## Exercise 3

Access application using NodeIP:NodePort.

---

## Exercise 4

Break the service by changing the selector label.

Check endpoints:

```
kubectl get endpoints
```

Fix the label to restore service.

---

## Exercise 5

Scale deployment:

```
kubectl scale deployment nginx-deployment --replicas=5
```

Verify service still works.

---

# 17. NodePort vs ClusterIP

| Feature         | ClusterIP | NodePort    |
| --------------- | --------- | ----------- |
| External access | No        | Yes         |
| Access method   | Internal  | NodeIP:Port |
| Load balancing  | Yes       | Yes         |

---

# 18. Common Mistakes

1. Label mismatch between Service and Pods
2. Pods not running
3. Incorrect targetPort
4. Node IP used incorrectly
5. Firewall blocking NodePort

---

# 19. Strategies and Tricks

## Strategy 1 — Always Check Endpoints

If NodePort doesn't work:

```
kubectl get endpoints
```

No endpoints = label mismatch.

---

## Strategy 2 — Use Labels Carefully

Pods and services must share the same labels.

---

## Strategy 3 — Test Internally First

Always test service inside cluster before external access.

---

## Strategy 4 — Use Auto NodePort for Learning

Avoid manual port conflicts.

---

## Strategy 5 — Keep Deployment and Service in Same Namespace

Otherwise service will not discover pods.

---

# 20. NodePort Interview Questions

1. What is NodePort in Kubernetes?
2. What is the NodePort range?
3. How does NodePort expose services externally?
4. Difference between ClusterIP and NodePort?
5. What happens if NodePort is not specified?
6. How does a service discover Pods?
7. What happens if service selectors don't match Pod labels?
8. Which command shows service endpoints?
9. How does Kubernetes load balance traffic?
10. Why is NodePort rarely used in production?

---

# 21. Key Takeaways

NodePort exposes applications outside the cluster.

Important components:

NodePort → Service Port → TargetPort → Pod

Access format:

NodeIP:NodePort

Understanding NodePort builds the foundation for learning advanced Kubernetes networking concepts later.


That version looks **much stronger on a DevOps portfolio**.
