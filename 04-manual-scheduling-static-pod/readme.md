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
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: manual
spec:
  nodeName: <target-node-name>  # This field enables manual scheduling
  containers:
  - image: nginx
    name: manual
```

📋 Part 2: Static Pods
What are Static Pods?
Static pods are managed directly by the kubelet on each node, not by the Kubernetes API server. They're defined by manifest files in a specific directory on each node.

Hands-on Implementation
1. Access Node Containers (Kind Environment)

```bash
# List running containers
docker ps

# Access control plane
docker exec -it my-first-cluster-control-plane bash

# Access worker nodes
docker exec -it my-first-cluster-worker bash
docker exec -it my-first-cluster-worker2 bash
```
2. Static Pod Manifest Location
bash


# Standard static pod directory
/etc/kubernetes/manifests/

3. Creating Static Pods
```bash
# Generate static pod YAML
kubectl run static-pod --image=nginx -o yaml --dry-run=client > static-pod.yaml

# Copy to static pod directory on each node
cp static-pod.yaml /etc/kubernetes/manifests/
```
🔍 Static Pod Behavior
Kubelet Management: Kubelet directly manages these pods
Mirror Objects: API server creates read-only mirror objects
No kubectl Control: Cannot edit/delete via kubectl
File-based: Changes require modifying manifest files on nodes
