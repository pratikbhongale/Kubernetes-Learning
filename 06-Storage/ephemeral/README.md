# Kubernetes Ephemeral Storage

## Overview

In **Kubernetes**, containers are designed to be **ephemeral**. This means that when a container or Pod is deleted or restarted, the container filesystem is recreated from the image and **any runtime data is lost**.

To handle **temporary runtime data**, Kubernetes provides **Ephemeral Storage**.

Ephemeral storage is **temporary storage that exists only during the lifecycle of a Pod**. When the Pod is deleted, the storage and all data inside it are automatically removed.

Ephemeral storage is commonly used for:

* Temporary files
* Application caches
* Data sharing between containers in the same Pod
* Intermediate processing data
* Log buffering

---

# What is Ephemeral Storage?

Ephemeral storage refers to **non-persistent storage tied to the lifetime of a Pod**.

Lifecycle:

```
Pod Created
     ↓
Volume Created
     ↓
Containers use the volume
     ↓
Pod Deleted
     ↓
Volume Deleted
```

Data stored in ephemeral storage **does not survive Pod deletion**.

---

# Why Learn Ephemeral Storage?

Understanding ephemeral storage is important because:

### 1. Containers are Stateless

Most cloud-native applications avoid storing permanent data inside containers.

### 2. Performance Optimization

Temporary storage is useful for caching, buffering, and processing files.

### 3. Multi-Container Collaboration

Containers in the same Pod can share files.

### 4. Foundation for Persistent Storage

Understanding ephemeral storage makes **Persistent Volumes (PV)** and **Persistent Volume Claims (PVC)** easier to learn.

### 5. Common in Production Systems

Real systems often use ephemeral storage for:

* log aggregation
* caching
* batch processing
* temporary build artifacts

---

# Types of Ephemeral Storage in Kubernetes

Kubernetes supports several ephemeral storage sources:

| Type              | Purpose                               |
| ----------------- | ------------------------------------- |
| emptyDir          | Temporary shared storage inside a Pod |
| configMap         | Configuration files                   |
| secret            | Sensitive data                        |
| downwardAPI       | Pod metadata                          |
| ephemeral volumes | Dynamic temporary volumes             |

For beginners, the **most important type is `emptyDir`**.

---

# emptyDir Volume (Core Ephemeral Storage)

`emptyDir` is the **most commonly used ephemeral storage type** in Kubernetes.

When a Pod is assigned to a node:

1. Kubernetes creates an empty directory.
2. The directory is mounted into the container.
3. All containers in the Pod can share it.
4. When the Pod is removed, the directory is deleted.

Lifecycle:

```
Pod Starts
   ↓
emptyDir created
   ↓
Containers read/write files
   ↓
Pod Deleted
   ↓
emptyDir removed
```

---

# Key Characteristics of emptyDir

* Created when Pod starts
* Deleted when Pod stops
* Shared between containers
* Stored on node disk by default
* Can optionally use RAM
* Simple to configure

---

# Basic emptyDir Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: emptydir-demo
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: cache-volume
      mountPath: /cache

  volumes:
  - name: cache-volume
    emptyDir: {}
```

Explanation:

* Kubernetes creates a temporary volume.
* It is mounted inside the container at `/cache`.

Inside the container:

```
/cache
```

acts like a normal directory.

---

# Sharing Data Between Containers

One major use case of `emptyDir` is **multi-container Pods**.

Architecture:

```
Container A → writes data
Container B → reads data
        ↓
      emptyDir
```

Example:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: shared-volume-demo
spec:
  containers:
  - name: writer
    image: busybox
    command: ["sh","-c","while true; do date >> /data/log.txt; sleep 5; done"]
    volumeMounts:
    - name: shared-volume
      mountPath: /data

  - name: reader
    image: busybox
    command: ["sh","-c","tail -f /data/log.txt"]
    volumeMounts:
    - name: shared-volume
      mountPath: /data

  volumes:
  - name: shared-volume
    emptyDir: {}
```

Behavior:

* `writer` container writes logs.
* `reader` container reads the same file.
* Both share the `emptyDir`.

---

# Memory Based emptyDir (tmpfs)

`emptyDir` can use **RAM instead of disk**.

This improves performance for caching workloads.

Example:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: memory-emptydir
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - mountPath: /cache
      name: mem-cache

  volumes:
  - name: mem-cache
    emptyDir:
      medium: Memory
```

This creates a **tmpfs filesystem backed by RAM**.

Advantages:

* extremely fast
* reduces disk IO
* good for high-speed caching

---

# Advantages of Ephemeral Storage

## 1. Easy Configuration

No cluster-level configuration required.

## 2. High Performance

Local node storage provides fast read/write speed.

## 3. Automatic Cleanup

Data is automatically removed when Pod terminates.

## 4. Ideal for Temporary Workloads

Perfect for caches and temporary files.

## 5. Supports Container Collaboration

Multiple containers in a Pod can share data.

---

# Disadvantages of Ephemeral Storage

## 1. Data Loss

```
Pod Deleted
     ↓
Volume Deleted
     ↓
Data Lost
```

Not suitable for storing important data.

---

## 2. Node Dependency

Ephemeral storage exists only on the node where the Pod runs.

If the Pod moves to another node, the data does not move.

---

## 3. Limited Storage Capacity

Nodes have limited disk space.

Heavy usage can cause **disk pressure**.

---

## 4. Not Suitable for Databases

Databases require **persistent storage**.

Examples:

* MySQL
* PostgreSQL
* MongoDB

These require PV/PVC instead.

---

# Real World Use Cases

## 1. Caching

Example architecture:

```
Application
    ↓
Cache responses
    ↓
emptyDir
```

If the Pod restarts, cache rebuilds automatically.

---

## 2. Log Processing (Sidecar Pattern)

```
App Container
     ↓
Writes logs
     ↓
emptyDir
     ↓
Log Collector Container
```

---

## 3. Data Processing Jobs

```
Download File
     ↓
Process Data
     ↓
Upload Result
```

Temporary files stored in ephemeral storage.

---

## 4. CI/CD Builds

Build systems store:

* build artifacts
* temporary compilation files
* test outputs

inside ephemeral storage.

---

# Best Practices

### 1. Never store critical data

Ephemeral storage should only hold temporary data.

### 2. Monitor disk usage

Large ephemeral volumes can fill node disks.

### 3. Use memory volumes carefully

Memory based volumes consume RAM.

### 4. Combine with persistent storage

Typical architecture:

```
Database → Persistent Storage
Cache → Ephemeral Storage
Logs → Ephemeral Storage
```

---

# When NOT to Use Ephemeral Storage

Avoid ephemeral storage when:

* data must survive Pod restarts
* data must move between nodes
* running databases
* storing user uploads

Instead use **Persistent Volumes (PV)**.

---

# Summary

Ephemeral storage provides **temporary Pod-level storage** for runtime data.

Key characteristics:

* Exists only during Pod lifecycle
* Automatically cleaned up
* Ideal for caching and temporary processing
* Not suitable for persistent data

Most common implementation:

```
emptyDir
```

Architecture example:

```
Pod
 ├── Container A
 ├── Container B
 └── emptyDir volume
```
