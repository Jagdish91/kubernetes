CKA Scheduling Challenge - Team Blue
====================================

Task Overview
-------------

This challenge tests your ability to combine multiple Kubernetes scheduling concepts:

*   Namespace isolation
    
*   Node taints and tolerations
    
*   Node labels and selectors
    
*   Node affinity rules
    
*   Pod anti-affinity for high availability
    

Cluster Setup
-------------

**Nodes:**

*   my-second-cluster-worker → labeled tier=frontend, tainted workload=frontend:NoSchedule
    
*   my-second-cluster-worker2 → labeled tier=backend
    

**Namespace:**

*   team-blue
    

Challenge Requirements
----------------------

### Part 1: Node Preparation

1.  Create namespace team-blue
    
2.  Label nodes appropriately:
    
    *   worker1 → tier=frontend
        
    *   worker2 → tier=backend
        
3.  Taint worker1 with workload=frontend:NoSchedule
    

### Part 2: Frontend Application

Create frontend-app pod with specifications:

*   **Image:** nginx
    
*   **Namespace:** team-blue
    
*   **Tolerations:** Must tolerate workload=frontend:NoSchedule
    
*   **Node Affinity:** Must run only on nodes with tier=frontend
    
*   **Pod Anti-affinity:** Must not share node with any pod labeled app=backend
    
*   **Label:** app=frontend (applied later)
    

### Part 3: Backend Application

Create backend-app pod with specifications:

*   **Image:** busybox with command sleep 3600
    
*   **Namespace:** team-blue
    
*   **Node Affinity:** Must run only on nodes with tier=backend
    
*   **Pod Anti-affinity:** Must not share node with pods labeled app=frontend
    
*   **Label:** app=backend
    

Expected Outcome
----------------

After successful implementation:

*   frontend-app → Runs exclusively on my-second-cluster-worker
    
*   backend-app → Runs exclusively on my-second-cluster-worker2
    
*   Both pods should be in Running state
