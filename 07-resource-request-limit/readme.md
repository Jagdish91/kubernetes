# 🚀 Kubernetes Resource Requests & Limits

A comprehensive learning guide for DevOps and Cloud Engineers

\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_\_

## 📚 Table of Contents

- ✓ Introduction & Problem Statement
- ✓ Requests vs Limits
- ✓ Resource Types & Units
- ✓ How Scheduling Works
- ✓ Monitoring with kubectl top
- ✓ Practical Demos & Examples
- ✓ LimitRange Configuration
- ✓ Best Practices & Interview Tips

\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_\_

 Requests & Limits

- Requests: Define minimum resources a container needs
- Limits: Define maximum resources a container can consume

### 💡 Key Benefits

- ✓ Efficient Resource Allocation
- ✓ Avoidance of Resource Starvation
- ✓ Cluster Stability & Predictability
- ✓ Mitigation of Noisy Neighbor Problem

\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_\_

## 📊 Requests vs Limits - Quick Comparison

| Aspect          | Request                        | Limit                     |
| --------------- | ------------------------------ | ------------------------- |
| Purpose         | Minimum resources guaranteed   | Maximum resources allowed |
| Used By         | Scheduler (placement decision) | Kernel (enforcement)      |
| Enforcement     | Soft (can exceed if available) | Hard (strictly enforced)  |
| CPU Behavior    | Reserve capacity               | Throttle if exceeded      |
| Memory Behavior | Reserve capacity               | OOM kill if exceeded      |

\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_\_

## 📏 Resource Types & Units

| Resource          | Unit               | Example              | Notes                          |
| ----------------- | ------------------ | -------------------- | ------------------------------ |
| CPU               | Cores (millicores) | 500m, 1, 2           | 1000m = 1 core                 |
| Memory            | Bytes (Mi/Gi)      | 256Mi, 1Gi           | Mi = Mebibytes, Gi = Gibibytes |
| Ephemeral Storage | Bytes (Mi/Gi)      | 500Mi, 2Gi           | Temporary storage per pod      |
| GPU               | Vendor-specific    | nvidia.com/gpu: 1    | Discrete accelerators          |
| HugePages         | Bytes              | hugepages-2Mi: 512Mi | Large page sizes               |

⚠️ Important:

Requests and limits must be defined per container inside a Pod, not at the Pod level.

\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_\_

## 🔧 Practical Example: Understanding Node Capacity

Scenario: Node with 6 vCPUs and 24GB memory

Container specifications:

requests:

cpu: "1"

memory: "2Gi"

limits:

cpu: "2"

memory: "4Gi"

Calculation:

- Maximum containers on node = 6 vCPUs ÷ 1 vCPU request = 6 containers
- Memory is still available (12GB), but CPU is fully allocated
- No more workloads can be scheduled due to CPU exhaustion

\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_\_

## ⚙️ How Requests Affect Scheduling

### Step 1: Pod Submission

When a Pod is created with resource requests, the scheduler receives the request.

### Step 2: Node Selection

The scheduler finds nodes with sufficient available resources:

Pod requests 500m CPU + 256Mi memory

Scheduler finds node with ≥ 500m CPU + 256Mi memory available

### Step 3: Pod Placement

If no suitable node exists → Pod stays in Pending state.

Example:

Container requests 100Mi memory but node has 24GB free

→ Container can use more than 100Mi if available

→ Kubernetes reserves 100Mi for this container's future use

\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_\_

## ⚡ How Limits Affect Running Containers

### CPU Limits → Throttling

- Enforced using CPU throttling
- Containers receive reduced CPU cycles
- No failure; just slowdown
- CPU is a compressible resource

### Memory Limits → OOM Kill

- Enforced using Out-of-Memory (OOM) killer
- Containers exceeding memory may be terminated
- Memory is an incompressible resource
- Termination happens via cgroups enforcement

| Scenario                | CPU Behavior            | Memory Behavior          |
| ----------------------- | ----------------------- | ------------------------ |
| Exceeds Limit           | Throttled               | OOM Killed               |
| Exceeds Request         | Uses available CPU      | Uses available memory    |
| Node has free resources | Can burst above request | Can use available memory |

⚠️ Critical:

Memory limit OOM kills happen regardless of node memory availability . The limit is per-container, enforced by Linux cgroups.

\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_\_

## 📡 Monitoring Resources with Metrics Server

### Installing Metrics Server

Step 1: Download latest release

kubectl apply -f <https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.7.2/components.yaml>

Step 2: For KIND clusters, add --kubelet-insecure-tls flag

spec:  
containers:  
\- args:  
\- --cert-dir=/tmp  
\- --secure-port=443  
\- --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname  
\- --kubelet-insecure-tls

Step 3: Verify installation

kubectl get pods -n kube-system

kubectl top nodes

\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_\_

## 🔨 Hands-On Commands from Your Session

| Command                           | Purpose                                                      |
| --------------------------------- | ------------------------------------------------------------ |
| k top nodes                       | View CPU and memory usage of all nodes                       |
| k top pods                        | View resource usage of pods in current namespace             |
| k get pods -n kube-system         | List pods in kube-system namespace (Metrics Server location) |
| k describe node &lt;node-name&gt; | View allocated resources on a specific node                  |
| k describe pod &lt;pod-name&gt;   | View pod details including requests and limits               |
| k apply -f &lt;yaml-file&gt;      | Deploy pod with specified resource constraints               |
| k delete pod &lt;pod-name&gt;     | Remove a pod                                                 |

\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_\_



### Verification Commands

kubectl get pods -o wide

kubectl describe pod memory-pod

kubectl top pods

\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_\_


### Q1: What's the difference between requests and limits?

Answer: Requests define minimum guaranteed resources used by the scheduler for placement. Limits define maximum resources enforced by the kernel-CPU is throttled, memory triggers OOM kill.

### Q2: Why use millicores instead of actual cores?

Answer: Millicores allow fractional CPU allocation. 500m = 0.5 cores, enabling granular resource distribution across many containers.

### Q3: Can a container exceed its memory limit?

Answer: No. The Linux kernel cgroups enforce memory limits. Exceeding triggers OOM kill, even if the node has free memory. The limit is per-container, not cluster-wide.

### Q4: What's the noisy neighbor problem?

Answer: One misbehaving pod consuming all node resources, starving other pods. Solved by enforcing requests and limits.

### Q5: When would you use LimitRange vs explicit Pod limits?

Answer: Use LimitRange for namespace-level defaults and enforcement. Use explicit Pod limits for workload-specific requirements. Both work together.

\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_**\_\_

