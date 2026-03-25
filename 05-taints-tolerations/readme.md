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

```
Verifying Taints

```bash
kubectl describe node <node-name> | grep -i taints
```
3. Implementing Tolerations (The Key)
To allow a pod to run on the tainted worker, I added the following to the Pod/Deployment spec:
```yaml
spec:
  tolerations:
  - key: "storage"
    operator: "Equal"
    value: "ssd"
    effect: "NoSchedule"
```
🧠 Key Technical Concepts
Taint Effects



Effect	Behavior
NoSchedule	Strict: No new pods allowed without toleration.
PreferNoSchedule	Soft: Try to avoid, but schedule here if nowhere else is available.
NoExecute	Eviction: Kicks out existing pods that don't have the toleration.

Operators
Equal (Default): Requires both Key and Value to match exactly.
Exists: Matches if the Key exists, regardless of the Value.

