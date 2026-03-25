# CKA Study Session: Manual Scheduling & Static Pods

## 📚 Overview
Today I explored two fundamental Kubernetes concepts that are crucial for the CKA exam:
- **Manual Scheduling**: Direct pod placement on nodes without the default scheduler
- **Static Pods**: Kubelet-managed pods that bypass the Kubernetes API for creation

## 🎯 Learning Objectives
- [x] Understand manual pod scheduling using `nodeName`
- [x] Create and manage static pods via kubelet
- [x] Explore the relationship between static pods and API server mirror objects
- [x] Practice with Kind cluster environment

---

## 🔧 Lab Environment
- **Cluster Type**: Kind (Kubernetes in Docker)
- **Nodes**: 1 Control Plane + 2 Worker Nodes
- **Tools Used**: kubectl, docker, vim

---

## 📋 Part 1: Manual Scheduling

### What is Manual Scheduling?
Manual scheduling allows you to bypass the Kubernetes scheduler and directly assign pods to specific nodes using the `nodeName` field in the pod specification.

### Hands-on Commands
```bash
# Generate a basic pod YAML
kubectl run manual --image=nginx --dry-run=client -o yaml > manual.yaml

# View current nodes
kubectl get nodes

# Check pod placement
kubectl get pods -o wide

```
Key Configuration

apiVersion: v1
kind: Pod
metadata:
  name: manual
spec:
  nodeName: <target-node-name>  # This field enables manual scheduling
  containers:
  - image: nginx
    name: manual
