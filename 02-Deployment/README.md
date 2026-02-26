# Kubernetes Deployments

## Table of Contents
1. [What is a Deployment?](#what-is-a-deployment)
2. [Why Use Deployments?](#why-use-deployments)
3. [Real-World Use Cases](#real-world-use-cases)
4. [Deployment YAML Example](#deployment-yaml-example)
5. [Line-by-Line Explanation](#line-by-line-explanation)
6. [Watching Pods and Containers](#watching-pods-and-containers)
7. [Labels, Selectors, and matchLabels](#labels-selectors-and-matchlabels)
8. [Multi-Container Pods](#multi-container-pods)
9. [Tips and Tricks](#tips-and-tricks)
10. [References](#references)

---

## What is a Deployment?

A **Deployment** is a Kubernetes object used to manage **Pods** in a declarative way. It ensures that a **specified number of Pod replicas** are running at all times. Deployments handle **rolling updates**, **scaling**, and **self-healing** automatically.

Think of a Deployment as a “manager” for Pods that guarantees your application is always running in the desired state.

---

## Why Use Deployments?

- Automatically maintain a specified number of Pods (**replicas**)  
- Roll out updates with zero downtime  
- Roll back to previous versions if something breaks  
- Manage multi-container Pods in a single, consistent template  
- Avoid manually creating and deleting Pods  

---

## Real-World Use Cases

- Running web servers (Nginx, Apache) in production  
- Deploying microservices for scalable applications  
- Managing background worker Pods for jobs  
- Ensuring critical apps like databases or API servers stay running  
- CI/CD pipelines where frequent updates are deployed automatically  

---

## Deployment YAML Example

Here’s a basic Deployment YAML:

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
        image: nginx:1.24
        ports:
        - containerPort: 80
````

---

## Line-by-Line Explanation

### `apiVersion: apps/v1`

Specifies the API version for Deployment objects. Deployments belong to the **apps API group**.

### `kind: Deployment`

Specifies that we are creating a **Deployment object**.

### `metadata:`

Contains identifying info about the Deployment, e.g., name and optional labels.

### `spec:`

Defines the desired state:

* **`replicas: 3`** → How many Pods should run
* **`selector.matchLabels`** → Tells Kubernetes which Pods the Deployment manages
* **`template`** → Blueprint for Pods that Deployment will create
* **`template.metadata.labels`** → Labels applied to Pods (**must match selector**)
* **`containers`** → Defines containers inside the Pod:

  * `name` → Container name (does not need to match labels)
  * `image` → Docker image to run
  * `ports` → Container ports

---

## Watching Pods and Containers

* Apply Deployment:

```bash
kubectl apply -f deployment.yaml
```

* Watch Pods:

```bash
kubectl get pods -w
```

* Describe a Pod:

```bash
kubectl describe pod <pod-name>
```

* Logs (for multi-container Pods):

```bash
kubectl logs <pod-name> -c <container-name>
kubectl logs -f <pod-name> -c <container-name> # follow logs
```

✅ Note: Use `-c <container-name>` for multi-container Pods.

---

## Labels, Selectors, and matchLabels

* **Labels** → Key-value tags attached to objects (`app: my-app`)
* **Selector** → Tells Deployment which Pods to manage
* **matchLabels** → Exact match of labels the Deployment uses

**Important Rule:** Selector labels **must match template labels**, but **container names do NOT need to match labels**.

---

## Multi-Container Pods Example

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: multi-container-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: multi-app
  template:
    metadata:
      labels:
        app: multi-app
    spec:
      containers:
      - name: nginx
        image: nginx:1.24
        ports:
        - containerPort: 80
      - name: busybox
        image: busybox
        command: ["sleep", "3600"]
```

* Pods have **2 containers** (`nginx` + `busybox`)
* Logs must specify container:

```bash
kubectl logs <pod-name> -c nginx
kubectl logs <pod-name> -c busybox
```

* `READY 2/2` indicates both containers are running

---

## Tips and Tricks

1. **Always match `selector` labels with Pod template labels**. Otherwise Deployment won’t work.
2. **Container name ≠ labels**. Container name is for logs and internal Pod management.
3. Use `kubectl get pods -w` to **watch Pods come up in real time**.
4. Use `kubectl describe pod <pod-name>` to see container lifecycle events.
5. For multiple containers, always use `-c <container-name>` with logs.
6. Start with single-container Deployments → then try multi-container.
7. Use `replicas` to scale Pods easily:

```bash
kubectl scale deployment nginx-deployment --replicas=5
```

8. You can generate Deployment YAML from `kubectl` (helpful for beginners):

```bash
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > deployment.yaml
```

---

## References

* [Kubernetes Official Docs – Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
* [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

---

**Congratulations!** 🎉
By studying this README, you now understand:

* Why Deployments are used
* How labels, matchLabels, and selectors work
* Multi-container Pods
* Watching Pods and container logs
* Real-world use cases and best practices

