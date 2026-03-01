
# Kubernetes Namespaces – Complete Guide

## **Table of Contents**

1. [Introduction](#introduction)
2. [Why Namespaces Are Important](#why-namespaces-are-important)
3. [Default Namespaces](#default-namespaces)
4. [Creating Namespaces](#creating-namespaces)
5. [Deploying Resources in Namespaces](#deploying-resources-in-namespaces)
6. [Querying Resources](#querying-resources)
7. [Switching Namespaces](#switching-namespaces)
8. [Name Isolation & Conflicts](#name-isolation--conflicts)
9. [Deleting Namespaces](#deleting-namespaces)
10. [Strategies & Tricks](#strategies--tricks)
11. [Summary](#summary)

---

## **1. Introduction**

In Kubernetes, a **Namespace** is a **logical partition of the cluster**, used to organize and isolate resources.

* Multiple teams or environments can share the same cluster safely.
* Namespaces help avoid resource name collisions.
* They are lightweight and do not affect Pod behavior—they only provide **organization, isolation, and scoping**.

---

## **2. Why Namespaces Are Important**

Namespaces are crucial when you have:

* **Multi-team clusters** → isolate team resources.
* **Environment separation** → dev, test, prod.
* **Resource management** → easier to track resources per namespace.
* **Access control (RBAC)** → apply permissions per namespace.

**Example Use Cases:**

| Scenario                                      | How Namespaces Help                                     |
| --------------------------------------------- | ------------------------------------------------------- |
| Dev and Prod environments on the same cluster | Dev resources in `dev` namespace, Prod in `prod`        |
| Two teams using same resource names           | Namespaces allow duplicate Pod or Service names         |
| Limited resources per team                    | Resource quotas (advanced) can be applied per namespace |

---

## **3. Default Namespaces**

Kubernetes comes with some default namespaces:

| Namespace         | Purpose                                        |
| ----------------- | ---------------------------------------------- |
| `default`         | Default for all resources if none is specified |
| `kube-system`     | System components like `kube-dns`              |
| `kube-public`     | Publicly readable, usually for cluster info    |
| `kube-node-lease` | Node heartbeat tracking                        |

---

## **4. Creating Namespaces**

You can create namespaces via **YAML** or **kubectl commands**.

### YAML Example:

**File: `namespaces.yaml`**

```yaml id="ns-yaml"
apiVersion: v1
kind: Namespace
metadata:
  name: dev
---
apiVersion: v1
kind: Namespace
metadata:
  name: test
---
apiVersion: v1
kind: Namespace
metadata:
  name: prod
```

Apply it:

```bash id="ns-yaml-apply"
kubectl apply -f namespaces.yaml
kubectl get namespaces
```

### kubectl Command Example:

```bash id="ns-create-cmd"
kubectl create namespace dev
kubectl create namespace test
kubectl create namespace prod
```

---

## **5. Deploying Resources in Namespaces**

Resources can be deployed in a **specific namespace** using YAML or `kubectl`.

### **a) Pod in a namespace (YAML)**

**File: `nginx-dev.yaml`**

```yaml id="pod-dev"
apiVersion: v1
kind: Pod
metadata:
  name: nginx-dev
  namespace: dev
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
```

Apply it:

```bash id="pod-dev-apply"
kubectl apply -f nginx-dev.yaml
kubectl get pods --namespace=dev
```

### **b) Using kubectl flag**

```bash id="pod-dev-create"
kubectl create deployment nginx --image=nginx --namespace=dev
```

---

## **6. Querying Resources**

* List Pods in a namespace:

```bash id="pods-dev"
kubectl get pods --namespace=dev
```

* List all Pods across namespaces:

```bash id="pods-all"
kubectl get pods --all-namespaces
```

* List all resources in a namespace:

```bash id="all-resources"
kubectl get all --namespace=dev
```

---

## **7. Switching Namespaces**

You can set a **default namespace** for your current kubectl context:

```bash id="switch-ns-dev"
kubectl config set-context --current --namespace=dev
kubectl get pods  # now defaults to dev namespace
```

Switch back to `prod`:

```bash id="switch-ns-prod"
kubectl config set-context --current --namespace=prod
kubectl get pods
```

---

## **8. Name Isolation & Conflicts**

Namespaces **allow identical resource names** in different namespaces:

* `nginx` Pod in `dev`
* `nginx` Pod in `prod`

No conflict occurs because each is scoped to its namespace.

Check all Pods with:

```bash id="pods-all-namespaces"
kubectl get pods --all-namespaces
```

---

## **9. Deleting Namespaces**

Deleting a namespace **removes all resources inside it**:

```bash id="ns-delete"
kubectl delete namespace test
kubectl delete namespace dev
kubectl delete namespace prod
```

---

## **10. Strategies & Tricks**

1. **Environment-based separation**

   * Common strategy: `dev`, `test`, `prod` namespaces.
   * Makes deployments predictable and avoids accidental production changes.

2. **Team-based separation**

   * Each team gets its own namespace (`team-a`, `team-b`).
   * RBAC can enforce access per namespace.

3. **Namespace-specific querying**

   * Always use `--namespace=<name>` to avoid confusion.
   * Use `kubectl get all --namespace=<name>` for complete view.

4. **Default namespace switching**

   * Switch context to frequently used namespace to save typing.

5. **Use `kubectl describe namespace <name>`**

   * Shows metadata, labels, annotations, and status.

6. **Resource name strategy**

   * Prefix resource names with namespace name for clarity (e.g., `dev-nginx`).

---

## **11. Summary**

* **Namespaces** = logical cluster partitions.
* **Purpose:** Isolation, organization, name conflict prevention, environment separation.
* **Key operations:** create, deploy resources, query, switch, delete.
* **Best practices:** Use namespaces for **env/team separation**, switch context wisely, follow naming conventions.
* **Hands-on exercises:** Deploy Pods in multiple namespaces, switch default namespaces, view resources, delete namespaces.

By following these examples, any learner will gain **strong theoretical understanding** and **practical skills** with Kubernetes namespaces.

---

This README is now a **complete beginner-to-advanced guide** for namespaces.


Do you want me to add that next-step exercise?
