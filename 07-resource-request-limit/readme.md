🚀 Kubernetes Resource Requests & Limits
A comprehensive learning guide for DevOps and Cloud Engineers
__________________________________________________
📚 Table of Contents
•	✓ Introduction & Problem Statement
•	✓ Requests vs Limits
•	✓ Resource Types & Units
•	✓ How Scheduling Works
•	✓ Monitoring with kubectl top
•	✓ Practical Demos & Examples
•	✓ LimitRange Configuration
•	✓ Best Practices & Interview Tips
__________________________________________________
🎯 Why Do We Need Requests and Limits?
The Noisy Neighbor Problem
Imagine a Kubernetes cluster with two deployments running on the same nodes:
•	Deployment-1: Normal application running smoothly
•	Deployment-2: Malfunctioning pods consuming all available CPU and memory
Without requests and limits, Deployment-2 starves Deployment-1 of resources, causing performance degradation or complete failure. This is the noisy neighbor problem.
✅ Solution: Requests & Limits
•	Requests: Define minimum resources a container needs
•	Limits: Define maximum resources a container can consume
💡 Key Benefits
•	✓ Efficient Resource Allocation
•	✓ Avoidance of Resource Starvation
•	✓ Cluster Stability & Predictability
•	✓ Mitigation of Noisy Neighbor Problem
__________________________________________________
📊 Requests vs Limits - Quick Comparison
Aspect	Request	Limit
Purpose	Minimum resources guaranteed	Maximum resources allowed
Used By	Scheduler (placement decision)	Kernel (enforcement)
Enforcement	Soft (can exceed if available)	Hard (strictly enforced)
CPU Behavior	Reserve capacity	Throttle if exceeded
Memory Behavior	Reserve capacity	OOM kill if exceeded
__________________________________________________
📏 Resource Types & Units
Resource	Unit	Example	Notes
CPU	Cores (millicores)	500m, 1, 2	1000m = 1 core
Memory	Bytes (Mi/Gi)	256Mi, 1Gi	Mi = Mebibytes, Gi = Gibibytes
Ephemeral Storage	Bytes (Mi/Gi)	500Mi, 2Gi	Temporary storage per pod
GPU	Vendor-specific	nvidia.com/gpu: 1	Discrete accelerators
HugePages	Bytes	hugepages-2Mi: 512Mi	Large page sizes
⚠️ Important:
Requests and limits must be defined
per container
inside a Pod, not at the Pod level.
__________________________________________________
🔧 Practical Example: Understanding Node Capacity
Scenario: Node with 6 vCPUs and 24GB memory
Container specifications:
requests:

cpu: "1"

memory: "2Gi"

limits:

cpu: "2"

memory: "4Gi"
Calculation:
•	Maximum containers on node = 6 vCPUs ÷ 1 vCPU request = 6 containers
•	Memory is still available (12GB), but CPU is fully allocated
•	No more workloads can be scheduled due to CPU exhaustion
__________________________________________________
⚙️ How Requests Affect Scheduling
Step 1: Pod Submission
When a Pod is created with resource requests, the scheduler receives the request.
Step 2: Node Selection
The scheduler finds nodes with sufficient available resources:
Pod requests 500m CPU + 256Mi memory

Scheduler finds node with ≥ 500m CPU + 256Mi memory available
Step 3: Pod Placement
If no suitable node exists → Pod stays in Pending state.
Example:

Container requests 100Mi memory but node has 24GB free

→ Container can use more than 100Mi if available

→ Kubernetes reserves 100Mi for this container's future use
__________________________________________________
⚡ How Limits Affect Running Containers
CPU Limits → Throttling
•	Enforced using CPU throttling
•	Containers receive reduced CPU cycles
•	No failure; just slowdown
•	CPU is a compressible resource
Memory Limits → OOM Kill
•	Enforced using Out-of-Memory (OOM) killer
•	Containers exceeding memory may be terminated
•	Memory is an incompressible resource
•	Termination happens via cgroups enforcement
Scenario	CPU Behavior	Memory Behavior
Exceeds Limit	Throttled	OOM Killed
Exceeds Request	Uses available CPU	Uses available memory
Node has free resources	Can burst above request	Can use available memory
⚠️ Critical:
Memory limit OOM kills happen
regardless of node memory availability
. The limit is per-container, enforced by Linux cgroups.
__________________________________________________
📡 Monitoring Resources with Metrics Server
Why kubectl top Fails Without Metrics Server
By default, Kubernetes doesn't collect metrics. The Metrics Server must be installed to provide resource usage data to the kubectl top command.
How Kubernetes Collects Metrics
1. cAdvisor (inside kubelet)

Collects CPU, memory, filesystem, and network metrics


2. Kubelet Exposes Metrics

Aggregates stats and exposes via HTTPS (port 10250)


3. Metrics Server Scrapes

Periodically pulls metrics from /metrics/resource endpoint


4. API Aggregation

kubectl top calls aggregated API at metrics.k8s.io/
Installing Metrics Server
Step 1: Download latest release
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.7.2/components.yaml
Step 2: For KIND clusters, add --kubelet-insecure-tls flag
spec:
  containers:
  - args:
    - --cert-dir=/tmp
    - --secure-port=443
    - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
    - --kubelet-insecure-tls
Step 3: Verify installation
kubectl get pods -n kube-system

kubectl top nodes
__________________________________________________
🔨 Hands-On Commands from Your Session
Command	Purpose
k top nodes	View CPU and memory usage of all nodes
k top pods	View resource usage of pods in current namespace
k get pods -n kube-system	List pods in kube-system namespace (Metrics Server location)
k describe node <node-name>	View allocated resources on a specific node
k describe pod <pod-name>	View pod details including requests and limits
k apply -f <yaml-file>	Deploy pod with specified resource constraints
k delete pod <pod-name>	Remove a pod
__________________________________________________
💻 Demo 1: Memory Requests & Limits
Pod Definition
apiVersion: v1
kind: Pod
metadata:
  name: memory-demo
spec:
  containers:
    - name: memory-demo-ctr
      image: polinux/stress
      resources:
        requests:
          memory: "100Mi"
        limits:
          memory: "200Mi"
      command: ["stress"]
      args: ["--vm", "1", "--vm-bytes", "190M", "--vm-hang", "1"]
Explanation
•	requests.memory: "100Mi" → Guaranteed minimum of 100MiB
•	limits.memory: "200Mi" → Cannot exceed 200MiB
•	--vm-bytes 190M → Allocates 190MiB (within limit)
•	Result: Pod runs successfully ✓
Exceeding Memory Limit
args: ["--vm", "1", "--vm-bytes", "250M", "--vm-hang", "1"]
•	Tries to allocate 250MiB (exceeds 200MiB limit)
•	Kernel triggers OOM kill
•	Pod is terminated with OOMKilled status
•	Result: Pod fails ✗
Verification Commands
kubectl get pods -o wide

kubectl describe pod memory-demo

kubectl top pods
__________________________________________________
💻 Demo 2: CPU Requests & Limits
Pod Definition
apiVersion: v1
kind: Pod
metadata:
  name: cpu-demo
spec:
  containers:
    - name: cpu-container
      image: vish/stress
      resources:
        requests:
          cpu: "500m"
        limits:
          cpu: "900m"
      args:
        - -cpus
        - "1"
Explanation
•	1 CPU = 1000m (millicores)
•	500m = 0.5 CPUs (request)
•	900m = 0.9 CPUs (limit)
•	Container tries to use 1 full CPU but gets throttled to 900m
•	Result: Pod runs but is CPU throttled ⚡
Verification Commands
kubectl apply -f cpu-demo.yaml

kubectl top pods

kubectl describe pod cpu-demo
__________________________________________________
🔐 LimitRange: Default Requests & Limits
Administrators can enforce default requests and limits for containers in a namespace using LimitRange.
Complete LimitRange YAML
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-resource-constraint
  namespace: default
spec:
  limits:
  - type: Container
    default:
      cpu: "2"
      memory: "4Gi"
    defaultRequest:
      cpu: "1"
      memory: "2Gi"
    max:
      cpu: "4"
      memory: "8Gi"
    min:
      cpu: "500m"
      memory: "512Mi"
Field Descriptions
Field	Purpose	Example
default	Applied if no limits specified	2 CPU, 4Gi memory max
defaultRequest	Applied if no requests specified	1 CPU, 2Gi memory min
max	Cannot exceed these values	4 CPU, 8Gi memory max
min	Cannot be lower than these values	500m CPU, 512Mi memory min
Applying and Verifying
kubectl apply -f limit-range.yaml

kubectl describe limitrange cpu-resource-constraint
Behavior After LimitRange
When you create a pod without requests/limits:


✓ Default limits applied: 2 CPU, 4Gi memory

✓ Default requests applied: 1 CPU, 2Gi memory

✓ Scheduler considers 1 CPU, 2Gi memory availability

✓ Container capped at 2 CPU, 4Gi memory at runtime
__________________________________________________
🎓 Best Practices for Production
1. Always Define Requests & Limits
❌ DON'T:
resources: {}
✅ DO:
resources:
  requests:
    cpu: "250m"
    memory: "256Mi"
  limits:
    cpu: "500m"
    memory: "512Mi"
2. Request-to-Limit Ratios
•	CPU: Limit typically 1.5x - 2x the request
•	Memory: Limit should match or slightly exceed request
3. Monitor Resource Usage
kubectl top nodes

kubectl top pods

kubectl describe node <node-name>
4. Use LimitRange for Namespace Defaults
Prevent uncontrolled resource consumption in shared namespaces.
5. Set Pod Disruption Budgets (PDB)
Ensure minimum availability during cluster maintenance.
__________________________________________________
💼 Interview Questions & Answers
Q1: What's the difference between requests and limits?
Answer: Requests define minimum guaranteed resources used by the scheduler for placement. Limits define maximum resources enforced by the kernel—CPU is throttled, memory triggers OOM kill.
Q2: Why use millicores instead of actual cores?
Answer: Millicores allow fractional CPU allocation. 500m = 0.5 cores, enabling granular resource distribution across many containers.
Q3: Can a container exceed its memory limit?
Answer: No. The Linux kernel cgroups enforce memory limits. Exceeding triggers OOM kill, even if the node has free memory. The limit is per-container, not cluster-wide.
Q4: What's the noisy neighbor problem?
Answer: One misbehaving pod consuming all node resources, starving other pods. Solved by enforcing requests and limits.
Q5: When would you use LimitRange vs explicit Pod limits?
Answer: Use LimitRange for namespace-level defaults and enforcement. Use explicit Pod limits for workload-specific requirements. Both work together.
__________________________________________________
📋 Your Command History Breakdown
What you accomplished in this session:
•	✓ Installed Metrics Server (components.yaml)
•	✓ Monitored node and pod resource usage (k top commands)
•	✓ Created memory stress test pods
•	✓ Created CPU stress test pods
•	✓ Observed OOM kills and CPU throttling
•	✓ Applied and tested LimitRange constraints
•	✓ Verified resource allocation at node and pod levels
__________________________________________________
🚀 Next Steps for Your DevOps Journey
•	📌 Practice Vertical Pod Autoscaler (VPA) for rightsizing
•	📌 Learn Horizontal Pod Autoscaler (HPA) based on metrics
•	📌 Explore Kyverno for policy enforcement
•	📌 Study Pod Disruption Budgets (PDB)
•	📌 Practice with Terraform to define resources as code
•	📌 Integrate Prometheus for custom metrics
•	📌 Set up alerts for OOM kills and CPU throttling
__________________________________________________
🎯 Key Takeaway
Resource Requests & Limits are fundamental to Kubernetes stability and cost optimization. Master these concepts to stand out as a DevOps engineer.
Created as a comprehensive learning resource for DevOps & Cloud Engineering
