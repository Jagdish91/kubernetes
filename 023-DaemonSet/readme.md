Kubernetes DaemonSet Log Collector Project
==========================================

🚀 Project Overview
-------------------

This project demonstrates a practical implementation of Kubernetes DaemonSet for log collection across cluster nodes. It's part of my CKA (Certified Kubernetes Administrator) preparation journey and showcases core Kubernetes concepts in real-world scenarios.

📋 Features Implemented
-----------------------

### ✅ Completed Task 1: Basic Log Collector DaemonSet

*   **DaemonSet Configuration:** Properly configured DaemonSet with label selectors and pod templates
    
*   **HostPath Volumes:** Mounted host directories for log collection (/var/log → container path)
    
*   **Tolerations:** Implemented tolerations for control-plane and custom node taints
    
*   **Resource Management:** Set CPU/memory requests and limits
    
*   **Log Collection Script:** Container continuously writes logs with timestamps and hostnames
    

### ✅ Completed Task 2: Advanced Log Collector with Specific Requirements

*   **Multi-toleration Support:** Handles both control-plane and custom infra node taints
    
*   **Directory Creation:** Uses DirectoryOrCreate type for non-existent host directories
    
*   **Verification Workflow:** Includes comprehensive testing commands and validation steps
    
*   **Error Handling:** Iterative development with delete-apply-test cycles
    

🛠 Key Kubernetes Concepts Demonstrated
---------------------------------------

### **DaemonSet Fundamentals**

*   One pod-per-node scheduling pattern
    
*   Suitable for cluster-wide monitoring, logging, and storage daemons
    
*   selector.matchLabels ↔ template.metadata.labels alignment
    

### **Volumes and Storage**

*   hostPath volume types: Directory, DirectoryOrCreate
    
*   Volume mounting at different container paths
    
*   File persistence across container restarts
    

### **Node Scheduling & Taints/Tolerations**

*   Control-plane taint: node-role.kubernetes.io/control-plane:NoSchedule
    
*   Custom node taints and how to tolerate them
    
*   Exists vs Equal operators for tolerations
    

### **Resource Management**

*   CPU requests (50m) and limits (100m)
    
*   Memory requests (64Mi) and limits (128Mi)
    
*   Best practices for resource constraints
    

### **DaemonSet Update Strategies**

*   RollingUpdate (default) - automatic rollout across nodes
    
*   OnDelete - manual control of pod updates
