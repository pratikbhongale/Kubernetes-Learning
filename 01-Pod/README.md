# Kubernetes Pods - Step-by-Step Guide

This README is a complete beginner-to-advanced guide to understanding **Kubernetes Pods**, how to create them using YAML, and tips & tricks to remember the concepts easily.  

---

## Table of Contents

1. [What is a Pod?](#what-is-a-pod)
2. [Pod YAML Structure](#pod-yaml-structure)
3. [Creating Your First Pod](#creating-your-first-pod)
4. [Labels and Selectors](#labels-and-selectors)
5. [Multi-Container Pods](#multi-container-pods)
6. [Pod Lifecycle & Status](#pod-lifecycle--status)
7. [Port Forwarding](#port-forwarding)
8. [Resources & Environment Variables](#resources--environment-variables)
9. [Volumes in Pods](#volumes-in-pods)
10. [Checking Logs & Exec into Pod](#checking-logs--exec-into-pod)
11. [Deleting Pods](#deleting-pods)
12. [Common Errors & Fixes](#common-errors--fixes)
13. [Tips, Tricks, and Memory Hacks](#tips-tricks-and-memory-hacks)
14. [Recommended Next Steps](#recommended-next-steps)

---

## What is a Pod?

A **Pod** is the smallest deployable unit in Kubernetes.  
It is a wrapper around **one or more containers** that:

- Share the **same network namespace** (same IP)
- Share **storage volumes** if defined
- Run on the same **Node**

Think of it as a “container group” managed by Kubernetes.

---

## Pod YAML Structure

Every Pod YAML follows this flow:

```text
apiVersion → kind → metadata → spec → containers → (name, image, ports, env, volumeMounts)
````

**Explanation:**

* **apiVersion**: Kubernetes API version (`v1` for Pods)
* **kind**: Object type (`Pod`)
* **metadata**: Information about the Pod (name, labels)
* **spec**: How the Pod runs (containers, restartPolicy, volumes)
* **containers**: List of containers inside the Pod
* **name**: Name of the container
* **image**: Docker image to run
* **ports**: Ports exposed by the container
* **env**: Environment variables
* **volumeMounts**: Attach storage to the container

---

## Creating Your First Pod

Create a file `nginx-pod.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
    - name: nginx-container
      image: nginx:latest
      ports:
        - containerPort: 80
```

Apply it:

```bash
kubectl apply -f nginx-pod.yaml
kubectl get pods
```

You should see:

```
nginx-pod   Running
```

---

## Labels and Selectors

**Labels** are key-value tags attached to Pods:

```yaml
metadata:
  labels:
    app: web
    tier: frontend
```

**Why labels are useful:**

* Select Pods using `kubectl get pods -l app=web`
* Group Pods logically (frontend/backend)
* Connect Pods to Services using selectors
* Used by Deployments/ReplicaSets

---

## Multi-Container Pods

Pods can have multiple containers sharing the same network:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-pod
spec:
  containers:
    - name: nginx
      image: nginx:latest
    - name: busybox
      image: busybox
      command: ["sleep", "3600"]
```

* Containers inside a Pod **can communicate via localhost**
* Share volumes if defined

---

## Pod Lifecycle & Status

Pod phases:

* **Pending**: Pod accepted but not running yet
* **ContainerCreating**: Containers being initialized
* **Running**: Containers are running
* **Succeeded**: Containers finished successfully
* **Failed**: Containers terminated with errors
* **Unknown**: Node unreachable

Check status:

```bash
kubectl get pods
kubectl describe pod nginx-pod
```

---

## Port Forwarding

Access Pod locally:

```bash
kubectl port-forward nginx-pod 8080:80
```

Open in browser:

```
http://localhost:8080
```

---

## Resources & Environment Variables

### Resource Requests & Limits

```yaml
resources:
  requests:
    memory: "64Mi"
    cpu: "250m"
  limits:
    memory: "128Mi"
    cpu: "500m"
```

* **requests** = minimum required
* **limits** = maximum allowed

### Environment Variables

```yaml
env:
  - name: APP_MODE
    value: dev
```

---

## Volumes in Pods

Pods are ephemeral. Data can be stored using **volumes**:

```yaml
volumes:
  - name: my-volume
    emptyDir: {}
```

Mount in container:

```yaml
volumeMounts:
  - mountPath: /data
    name: my-volume
```

---

## Checking Logs & Exec into Pod

* Check logs:

```bash
kubectl logs nginx-pod
```

* Exec into container:

```bash
kubectl exec -it nginx-pod -- /bin/bash
# or sh if bash not available
kubectl exec -it nginx-pod -- sh
```

---

## Deleting Pods

```bash
kubectl delete pod nginx-pod
kubectl delete -f nginx-pod.yaml
```

---

## Common Errors & Fixes

* **ImagePullBackOff** → Wrong image name or network issue
* **ContainerCreating** → Pod is initializing, check `kubectl describe pod`
* **CrashLoopBackOff** → Container starts then crashes repeatedly

---

## Tips, Tricks, and Memory Hacks

1. **Flow mnemonic for Pod YAML**:

   > `WHAT → WHO → HOW` → `apiVersion → kind → metadata → spec`

2. **Container section order**:

   > Name → Image → Ports → Env → VolumeMounts (**NIP EV**)

3. **Labels vs Selector**:

   * Labels = identity/tag of the Pod
   * Selector = who controls/targets the Pod

4. **Cheat Sheet Sentence**:

   > “A Kind Who Specifies Containers, Ports, Env, and Volumes”

5. **Multi-container Pods**: Containers share **IP & localhost**, useful for sidecars.

---

> 🏁 By mastering Pods, you’ve built the foundation to manage **Kubernetes workloads effectively**.
> Next step: move from Pods → Deployments → Services for real-world applications.

