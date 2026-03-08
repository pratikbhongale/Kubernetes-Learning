
# Kubernetes Storage Guide: Persistent Volumes (PV) & Persistent Volume Claims (PVC)

This guide explains **Kubernetes Persistent Storage** with **clear theory, YAML examples, commands, exercises, and interview scenarios**.

By the end of this guide, you should understand:

* Why Kubernetes needs persistent storage
* How **PersistentVolume (PV)** works
* How **PersistentVolumeClaim (PVC)** works
* How pods use storage
* How to debug storage issues
* How to answer **real interview scenario questions**

This guide focuses on **strong fundamentals + hands-on learning**.

---

# 1. The Storage Problem in Kubernetes

Containers are **ephemeral**.

This means:

* Containers can stop
* Pods can restart
* Pods can be rescheduled to another node

When this happens, **data inside the container is lost**.

Example:

```bash
kubectl exec -it mypod -- bash
echo "hello" > /data/file.txt
```

If the pod is deleted:

```bash
kubectl delete pod mypod
```

The file **disappears**.

This is a big problem for applications like:

* Databases
* Logging systems
* User uploads
* Application state

To solve this, Kubernetes provides **Persistent Storage**.

---

# 2. Kubernetes Storage Architecture

Kubernetes separates **storage resource** and **storage request**.

Architecture:

```
Pod → PVC → PV → Physical Storage
```

Component explanation:

| Component                   | Description             |
| --------------------------- | ----------------------- |
| PersistentVolume (PV)       | Actual storage resource |
| PersistentVolumeClaim (PVC) | Request for storage     |
| Pod                         | Uses the storage        |

Think of it like:

```
PV = Hard disk
PVC = Storage request form
Pod = Application using the disk
```

---

# 3. Persistent Volume (PV)

A **Persistent Volume** is a storage resource in the cluster.

It is created by **cluster administrators**.

Example storage sources:

* Local disk
* NFS
* Cloud storage (AWS EBS, Azure Disk)
* Distributed storage

For learning we use **hostPath**.

---

# 4. PV YAML Example

Create file:

```
pv.yaml
```

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: demo-pv
spec:
  capacity:
    storage: 1Gi

  accessModes:
    - ReadWriteOnce

  persistentVolumeReclaimPolicy: Retain

  hostPath:
    path: /mnt/data
```

Apply it:

```bash
kubectl apply -f pv.yaml
```

Check:

```bash
kubectl get pv
```

Example output:

```
NAME      CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS
demo-pv   1Gi        RWO            Retain           Available
```

---

# 5. PV Important Fields

## Capacity

```
capacity:
  storage: 1Gi
```

Storage size.

Kubernetes uses binary units:

| Unit | Meaning    |
| ---- | ---------- |
| Ki   | 1024 bytes |
| Mi   | 1024 Ki    |
| Gi   | 1024 Mi    |

---

## Access Modes

```
accessModes:
- ReadWriteOnce
```

Modes:

| Mode                | Meaning               |
| ------------------- | --------------------- |
| ReadWriteOnce (RWO) | One node read/write   |
| ReadOnlyMany (ROX)  | Many nodes read only  |
| ReadWriteMany (RWX) | Many nodes read/write |

---

## Reclaim Policy

```
persistentVolumeReclaimPolicy: Retain
```

Controls what happens when PVC is deleted.

Policies:

| Policy  | Behavior       |
| ------- | -------------- |
| Retain  | Keep storage   |
| Delete  | Delete storage |
| Recycle | Deprecated     |

---

# 6. hostPath Storage

```
hostPath:
  path: /mnt/data
```

This means storage is coming from **node filesystem**.

Example:

```
Node filesystem
/mnt/data
```

Data written by the pod will be stored here.

Example file created inside container:

```
/usr/share/nginx/html/test.txt
```

Actually stored at:

```
/mnt/data/test.txt
```

hostPath is mostly used for:

* learning
* local clusters
* development

Not recommended for production.

---

# 7. Persistent Volume Claim (PVC)

PVC is a **request for storage**.

Pods do not directly use PV.

They request storage using **PVC**.

---

# 8. PVC YAML Example

Create:

```
pvc.yaml
```

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: demo-pvc
spec:
  accessModes:
    - ReadWriteOnce

  resources:
    requests:
      storage: 500Mi
```

Apply:

```bash
kubectl apply -f pvc.yaml
```

Check:

```bash
kubectl get pvc
```

Example:

```
NAME       STATUS   VOLUME
demo-pvc   Bound    demo-pv
```

Bound means **PVC connected to PV**.

---

# 9. Using PVC in a Pod

Now we attach storage to a pod.

Create:

```
pod-storage.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: storage-pod
spec:
  containers:
  - name: nginx
    image: nginx

    volumeMounts:
    - name: storage-volume
      mountPath: /usr/share/nginx/html

  volumes:
  - name: storage-volume
    persistentVolumeClaim:
      claimName: demo-pvc
```

Apply:

```bash
kubectl apply -f pod-storage.yaml
```

---

# 10. What is Mounting?

Mounting means **attaching storage to a container directory**.

Example:

```
volumeMounts:
- name: storage-volume
  mountPath: /usr/share/nginx/html
```

This means storage will appear inside container at:

```
/usr/share/nginx/html
```

---

# 11. Testing Persistence

Enter pod:

```bash
kubectl exec -it storage-pod -- bash
```

Create file:

```bash
echo "hello storage" > /usr/share/nginx/html/test.txt
```

Delete pod:

```bash
kubectl delete pod storage-pod
```

Recreate pod.

Check file again:

```bash
kubectl exec -it storage-pod -- cat /usr/share/nginx/html/test.txt
```

The file **still exists**.

This proves persistence.

---

# 12. Storage Lifecycle

```
PV Created
↓
PVC Created
↓
PV Bound to PVC
↓
Pod uses PVC
↓
Data stored
```

---

# 13. Debugging Commands

Check PV

```bash
kubectl get pv
```

Check PVC

```bash
kubectl get pvc
```

Check pod volumes

```bash
kubectl describe pod <pod-name>
```

Check binding events

```bash
kubectl describe pvc <pvc-name>
```

---

# 14. Common Problems

## PVC stuck in Pending

Reason:

* No matching PV
* Storage class mismatch
* Access mode mismatch

---

## Pod fails to mount volume

Check:

```
kubectl describe pod
```

Look for:

```
Unable to attach or mount volumes
```

---

# 15. Beginner Exercises

## Exercise 1

Create:

* PV with **2Gi**
* PVC requesting **1Gi**

Verify binding.

---

## Exercise 2

Mount PVC into nginx pod.

Create file:

```
hello.txt
```

Delete pod.

Verify persistence.

---

## Exercise 3

Create **two pods using same PVC**.

Observe behavior with `ReadWriteOnce`.

---

# 16. Interview Questions

## Question 1

What is the difference between PV and PVC?

Answer:

```
PV = Storage resource
PVC = Storage request
```

PVC binds to available PV.

---

## Question 2

What happens if PVC is deleted?

Answer:

Depends on reclaim policy.

Example:

```
Retain → data kept
Delete → storage deleted
```

---

## Question 3

What happens if PV is deleted?

Answer:

PVC becomes:

```
Pending
```

Pods using that storage may fail.

---

# 17. Scenario Based Interview Question

Scenario:

A developer created PVC but it is stuck in **Pending**.

What will you check?

Answer steps:

1. Check PVC

```
kubectl get pvc
```

2. Describe PVC

```
kubectl describe pvc <name>
```

3. Check PV

```
kubectl get pv
```

4. Verify

* storage size
* access mode
* storage class

---

# 18. Real World Use Cases

Persistent storage is required for:

Databases

* MySQL
* PostgreSQL
* MongoDB

Logging

* Elasticsearch
* Loki

File storage

* WordPress uploads
* user documents

---

# 19. Strategies & Tricks for Learning

### Trick 1

Always visualize storage like this:

```
Pod
↓
PVC
↓
PV
↓
Disk
```

---

### Trick 2

Debug order:

```
Pod
↓
PVC
↓
PV
```

---

### Trick 3

Remember binding rule:

```
PV capacity ≥ PVC request
```

---

# 20. Important Concepts to Remember

| Concept        | Meaning                       |
| -------------- | ----------------------------- |
| PV             | Storage resource              |
| PVC            | Storage request               |
| Mount          | Attach storage to container   |
| hostPath       | Node filesystem storage       |
| Reclaim Policy | What happens when PVC deleted |

---

# 21. Final Summary

```
Persistent Volume (PV)
    ↓
Persistent Volume Claim (PVC)
    ↓
Pod Volume Mount
    ↓
Application uses storage
```

Persistent storage allows Kubernetes workloads to **survive pod restarts and failures**.

Mastering PV and PVC is essential for:

* DevOps engineers
* Kubernetes administrators
* Cloud engineers

---

End of Guide
