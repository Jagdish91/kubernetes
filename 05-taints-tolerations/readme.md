# CKA Study: Mastering Taints and Tolerations 🛡️🔑

## 📖 Overview
Taints and Tolerations are the "Lock and Key" mechanism of Kubernetes scheduling. They allow you to repel pods from certain nodes unless those pods have a matching "key" (Toleration).

## 🎯 Learning Objectives
- [x] Understand the "Lock & Key" analogy for node scheduling.
- [x] Apply Taints to nodes with different effects (`NoSchedule`, `PreferNoSchedule`, `NoExecute`).
- [x] Configure Pod Tolerations using `Equal` and `Exists` operators.
- [x] Understand why `nodeName` (Manual Scheduling) bypasses these restrictions.
- [x] Implement real-world isolation scenarios (Dedicated Nodes, GPU workloads).

---

## 🛠️ Hands-on Lab: Commands & Implementation

### 1. Applying Taints to Nodes
I used a Kind cluster with three nodes: `control-plane`, `worker`, and `worker2`.

```bash
# Apply a 'Lock' to worker 1
kubectl taint nodes my-second-cluster-worker storage=ssd:NoSchedule

# Apply a 'Lock' to worker 2
kubectl taint nodes my-second-cluster-worker2 storage=hdd:NoSchedule
