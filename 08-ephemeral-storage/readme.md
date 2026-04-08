# Kubernetes Ephemeral Storage & Downward API - CKA Study Guide 🚀

[![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)](https://kubernetes.io/)
[![CKA](https://img.shields.io/badge/CKA-Certified-green?style=for-the-badge)](https://www.cncf.io/certification/cka/)
[![DevOps](https://img.shields.io/badge/DevOps-Learning-blue?style=for-the-badge)](https://github.com/yourusername)

## 📖 Overview
This repository contains comprehensive study materials for **Kubernetes Ephemeral Storage** and the **Downward API** - essential topics for the **Certified Kubernetes Administrator (CKA)** certification. 

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
