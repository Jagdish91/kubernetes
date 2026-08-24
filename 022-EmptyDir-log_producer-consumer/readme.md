# Kubernetes Multi-Container Pod with Shared Storage - Log Demo

A hands-on Kubernetes project demonstrating **ephemeral storage (emptyDir)**, **multi-container pods**, **volume sharing**, and **container health probes**.

## 🎯 Project Overview

This project implements a real-world scenario where two containers in the same Kubernetes Pod share temporary storage to process logs:

- **log-producer**: Writes application logs to a shared volume every 10 seconds
- **log-aggregator**: Reads logs from the shared volume, counts entries, and writes summaries every 30 seconds

This demonstrates core Kubernetes concepts essential for DevOps and Cloud Engineers.

## 📚 Learning Objectives

After working through this project, you will understand:

✅ **emptyDir volumes** — temporary storage that lives only as long as the Pod  
✅ **Multi-container Pods** — designing Pods with multiple app containers  
✅ **Volume mounts** — sharing storage between containers  
✅ **Container startup coordination** — ensuring dependent containers wait properly  
✅ **Startup probes** — health checks during container initialization  
✅ **kubectl debugging** — examining logs and verifying shared state  

🔍 Key Concepts Explained
-------------------------

### **emptyDir Volume**

*   Created when Pod is created
    
*   Deleted when Pod is deleted
    
*   Shared by all containers in the Pod
    
*   Lives on the node's disk
    
*   sizeLimit: 50Mi prevents unbounded growth
    

### **Multi-Container Pod**

*   Containers in the same Pod share:
    
    *   Network namespace (same IP, can reach via localhost)
        
    *   IPC namespace
        
    *   Shared volumes
        
*   Containers start in parallel (no built-in ordering)
    
*   All containers must be healthy for Pod to be "Ready"
    

### **Volume Mounts**

*   Each container specifies where it wants the volume mounted
    
*   Same volume mounted to different paths in different containers
    
*   Changes in /shared/logs by producer are immediately visible to aggregator
    

### **Startup Probe**

```startupProbe:
  exec:
    command: [sh, -c, "test -s /shared/logs/app.log"]
  periodSeconds: 2
  failureThreshold: 15
```

*   Checks if file exists and is non-empty (-s flag)
    
*   Runs every 2 seconds
    
*   Pod fails if all 15 attempts fail (30 seconds total)
    
*   Once successful, probe stops running
    

### **Container Startup Coordination**

Unlike init containers, regular app containers start in parallel. To ensure aggregator waits for producer:

```until [ -s /shared/logs/app.log ]; do
  echo "waiting for app.log to have content..."
  sleep 2
done
```

