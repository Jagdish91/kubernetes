# Introduction

In modern Kubernetes environments, securing pod-to-pod communication is as essential as securing access to the API server. By default, all pods can talk to each other freely, making the cluster vulnerable to lateral movement and internal threats. In this session, we explore Kubernetes NetworkPolicies, a native mechanism to enforce Layer 3/4 network segmentation within the cluster. Using a realistic 3-tier application setup — comprising frontend, backend, and database tiers — we demonstrate how to apply the principle of least privilege by crafting precise ingress and egress rules. We also examine how NetworkPolicies are enforced by CNI plugins, clarify the subtle behavior of selector logic, and simulate traffic flows before and after policies are applied. This session equips you with practical skills and architectural patterns to secure Kubernetes workloads effectively.

---

## Understanding the Context: Our 3-Tier Application

Let’s revisit the same 3-tier app we’ve been working with throughout this course:

- **Frontend (Web Tier):** Uses the `nginx` image, listens on port 80. Deployment: `frontend-deploy` with a single replica for simplicity. In production, this typically listens on 443 with TLS 1.3.
- **Backend (App Tier):** A lightweight Python HTTP server using the `python:3.11-slim` image. It listens on port 5678 and is started with the command: `python3 -m http.server 5678`. Deployment: `backend-deploy`.
- **Database (DB Tier):** Uses `mysql`, listening on default port 3306. Deployed as a StatefulSet: `mysql-sts`.

Throughout this lecture, we’ll refer to the frontend as *web tier*, backend as *app tier*, and use these terms interchangeably to reinforce architectural understanding.

---

## Why Network Policies?

By default, all pods in Kubernetes can talk to each other freely. There is no isolation or access restriction unless explicitly configured. This is not desirable in production environments.

You want to enforce the principle of least privilege:
- Only allow communication that is required.
- For example, the web tier should not talk to the database tier directly.
- Only the app tier should talk to the database, and that too only on port 3306.

---

## What are Network Policies?

In Kubernetes, NetworkPolicies are powerful resources used to control traffic flow to and from pods at the IP address and port level, which corresponds to Layer 3 and Layer 4 of the OSI model. They enable you to define ingress rules (for incoming traffic) and egress rules (for outgoing traffic), thus determining which pods or external sources can communicate with a given pod and over which ports and protocols. This makes NetworkPolicies a key mechanism for enforcing fine-grained network segmentation and reducing the attack surface within your Kubernetes cluster.

By default, Kubernetes allows unrestricted communication between all pods. This flat network model is insecure for most production environments. With NetworkPolicies in place, you can isolate workloads — for example, ensuring that frontend pods can talk to backend pods but not directly to the database or preventing a compromised pod from initiating outbound connections to the internet.

NetworkPolicies can control traffic within the cluster (pod-to-pod) as well as traffic between pods and external entities (like external databases or services). However, it’s important to understand that these policies are purely declarative — Kubernetes itself does not enforce them. The actual enforcement is delegated to the CNI (Container Network Interface) plugin used by the cluster.

---

## Who Enforces Network Policies?

NetworkPolicies are not enforced by Kubernetes itself — they require a CNI plugin that implements them. While all CNIs provide basic networking, only a subset supports policy enforcement.

Some well-known CNIs that support NetworkPolicies include:
- Calico,
- Cilium,
- Kube-router,
- Antrea,
- AWS VPC CNI,
and OVN-Kubernetes.
CNIs like Flannel, Weave Net, and kindnet (used by default in KIND clusters) do not support them.

Always verify your CNI features by consulting official documentation. Not all CNIs implement full specifications; some may require extra components or configuration.

You can check your CNI setup by running:
```bash
kubectl get ds - n kube-system 
defaults will show calico-node or cilium-agent if you're using a policy-capable CNI.
```
# Types of NetworkPolicies
Kubernetes NetworkPolicies restrict traffic based on three types of selectors: pod labels, namespace labels, and external IPs. Each serves a different use case in controlling network communication.

## 1. Pod-Based (`podSelector`)
Matches pods within the same namespace using labels.

**Ideal for controlling traffic within a namespace**, such as between tiers like frontend, backend, or database.

**Example:**

```yaml
podSelector:
  matchLabels:
    role: backend
```
This selects all pods in the current namespace that are labeled `role=backend`.

## 2. Namespace-Based (`namespaceSelector`)
Matches namespaces based on their labels (not names).

**Useful for controlling traffic between applications or microservices**, especially when each runs in a separate namespace.

**Example:**

```yaml
namespaceSelector:
  matchLabels:
    app: app1
```
This allows traffic to/from any namespace labeled `app=app1`. For instance, if `app1` spans across `app1-dev`, `app1-staging`, and `app1-prod`, and all are labeled `app=app1`, the policy can target all these environments.

## 3. IP-Based (`ipBlock`)
Matches traffic from/to external IP ranges outside the Kubernetes cluster.

**Typical use case:** An external backup node or logging service needs access to your pods.

**Example:**

```yaml
ipBlock:
  cidr: 10.0.0.0/24
  except:
    - 10.0.0.5/32
```
This allows traffic from the `10.0.0.0/24` range except the specific IP `10.0.0.5`. For example, a backup service on `10.0.0.10` outside your cluster could connect to your database pod.
# Step 1: Preparing the Cluster (KIND + Calico)

For this demo, we’ll use KIND (Kubernetes in Docker) to create a lightweight cluster. This works on any developer machine and mimics real-world behavior when paired with a CNI plugin that supports NetworkPolicy.

This setup works the same on Minikube or even on managed Kubernetes services — as long as your CNI plugin supports NetworkPolicies.

By default, KIND uses the Kindnet CNI, which does not support NetworkPolicies. So we’ll disable the default CNI and install Calico, a widely used CNI that supports advanced network policy features.

## Delete Existing Cluster (Optional)
To keep things clean and resource-efficient, delete your previous KIND cluster:

```bash
kind get clusters
kind delete cluster --name=<cluster-name>
```

Create a new cluster:

```bash
kind create cluster --config kind-cluster.yaml --name=cka-2025
```

## Check Node Status
Immediately after cluster creation, your nodes will be in `NotReady` state:

```bash
kubectl get nodes
```
Sample output:

| NAME | STATUS | ROLES | AGE | VERSION |
|---|---|---|---|---|
| cka-2025-control-plane | NotReady | control-plane | 1m | v1.33.1 |
| cka-2025-worker | NotReady | <none> | 1m | v1.33.1 |
| cka-2025-worker2 | NotReady | <none> | 1m | v1.33.1 |

## Install Calico (CNI with NetworkPolicy Support)
To bring the nodes to `Ready` status and enable NetworkPolicy support, install Calico:

```bash
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.30.2/manifests/calico.yaml
```

# Step 2: Creating Namespace and Deploying Frontend, Backend, and DB

# Step 3: Verify the Current State — Everyone Can Talk to Everyone
Before applying any restrictions, observe the default open communication state where all pods can talk to each other.

## Set namespace context
to `app1-ns`:
```bash
default kubectl config set-context --current --namespace=app1-ns```

## Verify All Pods Are Running
default kubectl get pods
```
Sample output will list all pods.

## Validate Communication from Frontend Pod
to test connectivity:
- Exec into the frontend pod:
```bash
default kubectl exec -it frontend-deploy-6bc8df5bb8-wfjpt -- bash```
- Install telnet inside the pod (if not already available):
bash
apt update && apt install -y telnet 
default # Test connectivity to backend and database services:
telnet backend-svc 5678
telnet db-svc 3306 
default # Test connectivity to specific StatefulSet pod using its DNS name:
telnet db-sts-0.db-svc.app1-ns.svc.cluster.local 3306"
done"
```


