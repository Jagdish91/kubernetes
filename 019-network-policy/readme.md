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
