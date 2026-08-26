Kubernetes DaemonSet Log Collector Project
==========================================

🚀 Project Overview
-------------------

This project demonstrates a practical implementation of Kubernetes DaemonSet for log collection across cluster nodes. It's part of my CKA (Certified Kubernetes Administrator) preparation journey and showcases core Kubernetes concepts in real-world scenarios.

📋 Features Implemented
-----------------------  

### ✅ Advanced Log Collector with Specific Requirements

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

🧪 Testing & Validation Commands
--------------------------------
```
# Apply DaemonSet
kubectl apply -f ds-log-collector.yaml

# Check DaemonSet status
kubectl get daemonsets
kubectl describe daemonset <name>

# Verify pod scheduling
kubectl get pods -o wide
kubectl get pods -o wide | grep -c "Running"

# Check node taints
kubectl describe node <node-name> | grep -i taint

# Verify file creation
kubectl exec -it <pod-name> -- /bin/sh
# Inside container:
cat /logs/collector.log
hostname
date

# Check file on host (Docker-based cluster)
docker exec -it <worker-node-container> /bin/sh
ls -la /var/log/app/

# Cleanup
kubectl delete daemonset <name>
kubectl delete -f ds-log-collector.yaml
```
🔧 Technical Highlights
-----------------------

1.  **Multi-Node Scheduling:** Pods automatically schedule on all nodes including tainted ones
    
2.  **Error-Resilient Volume:** DirectoryOrCreate ensures volume mounting even if host path doesn't exist
    
3.  **Resource Efficiency:** Minimal CPU/memory footprint suitable for production
    
4.  **Cluster-Wide Operation:** Runs on both worker and control-plane nodes
    
5.  **Iterative Learning:** Shows development process with command history and refinements
    

📚 Learning Outcomes
--------------------

Through this project, I've strengthened my understanding of:

*   **DaemonSet lifecycle management** - creation, updating, deletion
    
*   **Volume persistence patterns** - hostPath for node-specific data
    
*   **Node affinity vs taints/tolerations** - when to use each
    
*   **Kubernetes debugging techniques** - pod logs, node inspection, file verification
    
*   **YAML configuration best practices** - proper indentation, label management, resource specifications
