# CKA Study: Node Selector & Node Affinity 🎯🧲

## 📖 Overview
Today's session focused on **Node Assignment**—controlling exactly where pods land using labels. While Taints *repel* pods, Node Selectors and Affinity *attract* them.

## 🎯 Learning Objectives
- [x] Master simple scheduling with `nodeSelector`.
- [x] Implement advanced scheduling with `nodeAffinity`.
- [x] Understand Hard (`required`) vs. Soft (`preferred`) constraints.
- [x] Use advanced operators: `In`, `NotIn`, `Exists`, `Gt`, `Lt`.
- [x] Learn how to implement logic (AND/OR) in scheduling rules.

---

## ⚖️ Comparison: Node Selector vs. Node Affinity

| Feature | Node Selector | Node Affinity |
| :--- | :--- | :--- |
| **Complexity** | Simple (`key: value`) | Advanced (Set-based) |
| **Logic** | Exact Match only | `In`, `NotIn`, `Exists`, `Gt`, `Lt` |
| **Flexibility** | Hard rule only | Hard (`required`) & Soft (`preferred`) |
| **Usage** | Basic placement | Complex topology & preferences |

---

## 🛠️ Hands-on Lab: Implementation

### 1. The Basic Approach: Node Selector
The simplest way to pin a pod to a node.

```bash
# Label the node
kubectl label nodes my-second-cluster-worker storage=ssd

# Pod configuration
spec:
  nodeSelector:
    storage: ssd
```

The Advanced Approach: Node Affinity
A. Hard Rules (Required)
The pod MUST run on a matching node, or it will not schedule.
```yaml
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: storage
            operator: In
            values: ["ssd", "hdd"] # OR logic (SSD or HDD)
```


