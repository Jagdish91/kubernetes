# Kubernetes Ephemeral Storage & Downward API 

## 📖 Overview
This repository contains comprehensive study materials for **Kubernetes Ephemeral Storage** and the **Downward API** 

## 🎯 Learning Objectives
- How `emptyDir` volumes facilitate shared scratch space within Pods.
- How the Downward API provides runtime metadata to containers.
- Practical use cases for ephemeral storage in production.
- Container layer architecture and shared writable space concepts.

---

## 🗄️ Ephemeral Storage
**Ephemeral storage** refers to temporary storage that exists only during the Pod's lifetime. When the Pod is deleted, all data is permanently lost.

### 📁 EmptyDir Volume
- **Temporary Directory:** Created when a Pod is assigned to a node.
- **Pod Lifetime:** Exists only as long as the Pod is running.
- **Shared Access:** All containers within the same Pod can access the same emptyDir volume.

### 🏗️ Container Layers Recap
- **Read-Only Layers:** Shared across containers using the same image.
- **Writable Layer:** Each container gets its own isolated writable layer.
- **EmptyDir Solution:** Provides shared writable space across containers in a Pod.

### 🧪 EmptyDir Demo
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: emptydir-example
spec:
  containers:
  - name: busybox-container
    image: busybox
    command: ["/bin/sh", "-c", "sleep 3600"]
    volumeMounts:
    - mountPath: /data
      name: temp-storage
  - name: busybox-container-2
    image: busybox
    command: ["/bin/sh", "-c", "sleep 3600"]
    volumeMounts:
    - mountPath: /data
      name: temp-storage
  volumes:
  - name: temp-storage
    emptyDir: {}
```

🔍 Downward API
The Downward API enables Pods to access metadata about themselves (name, namespace, labels, etc.) without calling the API server.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: downwardapi-example
spec:
  containers:
  - name: metadata-container
    image: busybox
    command: ["/bin/sh", "-c", "env && sleep 3600"]
    env:
    - name: POD_NAME
      valueFrom:
        fieldRef:
          fieldPath: metadata.name
```

Volume file Demo
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: downwardapi-volume
  labels:
    app: monitor
spec:
  containers:
  - name: metadata-container
    image: busybox
    volumeMounts:
    - name: downwardapi-volume
      mountPath: /etc/podinfo
  volumes:
  - name: downwardapi-volume
    downwardAPI:
      items:
      - path: "labels"
        fieldRef:
          fieldPath: metadata.labels
```



