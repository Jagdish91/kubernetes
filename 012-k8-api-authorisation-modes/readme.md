# Kubernetes API: The Engine Behind Everything

## What Is the Kubernetes API?

The Kubernetes API is the primary interface to your cluster. Whether you're creating a Pod, scaling a Deployment, or checking the status of a resource, you're interacting with this API.

It’s a RESTful, resource-based interface that exposes core Kubernetes objects like Pods, Services, Deployments, ConfigMaps, and more. These objects are accessible through structured URLs like `/api/v1` and `/apis/apps/v1`, and they respond to standard HTTP verbs like:

- **GET** – retrieve a resource
- **POST** – create a resource
- **PUT/PATCH** – update a resource
- **DELETE** – remove a resource

Anytime you use a tool like `kubectl`, it’s acting as a REST client that translates your CLI command into an API call.

### Example:
```bash
kubectl get pods
```
translates to:
```http
GET /api/v1/namespaces/default/pods
```
This request is sent over HTTPS, authenticated using your kubeconfig, and authorized using cluster policies like RBAC or Webhook authorization.

# Kubernetes API vs. API Server

Since the beginning of the course, we've said that the API server is the central hub of all Kubernetes communication — and that’s true. But it’s worth clarifying that the Kubernetes API is just one part of what the API server does.

## The Kubernetes API
Refers specifically to the set of RESTful endpoints (like `/api`, `/apis`, `/healthz`, etc.) that expose cluster resources. It’s the interface used by internal components and external clients to read or modify the state of the cluster.

## The API Server's Responsibilities
The API server itself does much more than just serve API endpoints. It’s responsible for:
- Authentication (verifying who is making the request)
- Authorization (checking if the action is allowed)
- Schema validation (ensuring requests match expected formats)
- Admission control (enforcing cluster policies)
- And much more...

## Summary
So, in short:
- **The Kubernetes API** is a *logical interface* — the set of endpoints that represent your cluster's state.
- **The API server** is the *physical component* that hosts that API and applies all the control mechanisms around it.

This distinction helps frame the API server not just as a gateway to the cluster, but also as a gatekeeper, enforcing critical checks before anything gets stored in `etcd`.

## Key Pointers
To recap:
1. The **Kubernetes API** is the interface, exposing your cluster's resources.
2. The **API server** is the component that hosts this API and applies all access controls around it.
3. Every request goes through a well-defined pipeline: **authentication → authorization → validation → execution**.
4. Tools like `kubectl`, controllers, and even kubelets all communicate with your cluster through this same pathway.

# Kubernetes API Endpoints Overview

At a high level, the API server serves multiple paths. Each path corresponds to a different kind of functionality:

| Endpoint | Description |
| --- | --- |
| `/version` | Returns the version of the Kubernetes API server. Unauthenticated and publicly accessible. |
| `/healthz`, `/livez`, `/readyz` | Health check endpoints for the API server itself, used by monitoring tools. `/livez` and `/readyz` are preferred in newer Kubernetes versions. Unauthenticated and publicly accessible. |
| `/api` | Root path for the core API group (`""` group), which includes legacy and foundational resources like Pods, Services, ConfigMaps, etc. |
| `/apis` | Root path for all named API groups (e.g., `apps`, `rbac.authorization.k8s.io`, `networking.k8s.io`). Most modern resources live here. |
| `/metrics` | Exposes Prometheus-format metrics that can be scraped by Prometheus or any monitoring system that supports the Prometheus exposition format. |
| `/logs` | ❌ Not a standalone API endpoint. Accessing logs via `kubectl logs` invokes the kubelet’s `/containerLogs/` endpoint, not `/logs`. Used by the API server to expose its own logs only when certain flags are enabled (e.g., `--logtostderr=false`, `--log-dir`). |

## API Endpoints: Resource vs Non-Resource

The Kubernetes API is split across multiple HTTPS endpoints, each serving a different purpose:

### Resource endpoints
These expose Kubernetes objects (Pods, Services, Deployments, etc.) and allow CRUD operations using standard HTTP verbs (GET, POST, PUT, DELETE).

- `/api` → for core group resources (e.g., Pods, ConfigMaps, Services).
- `/apis` → for named group resources (e.g., Deployments, Ingresses, Roles).

### Non-resource endpoints
These do not expose Kubernetes objects but provide metadata or cluster-level utilities:

- `/version` – Returns API server version.
- `/healthz`, `/livez`, `/readyz` – Health check endpoints for the API server itself.
- `/metrics` – Exposes API server metrics in Prometheus format.
- `/logs` – Not a true API endpoint; `kubectl logs` reaches the kubelet, not the API server.

## API Groups: Organizing the Kubernetes API

Kubernetes organizes its API using **API groups**, enabling modular feature development and independent versioning. These are split into the **core group** and various **named groups**.

### 1. Core Group (`/api`) 
The core group—also called the legacy group—has no explicit group name and is served at /api/v1. It includes foundational objects essential to nearly every workload:

- Pods,
- Services,
- ConfigMaps,
- Secrets,
- Namespaces,
- PersistentVolumes (PVs),
- PersistentVolumeClaims (PVCs),
- ReplicationControllers.

When you use `apiVersion: v1` in a manifest, you're referencing this group. Example: Pods can be listed via /api/v1/pods.

### 2. Named Groups (`/apis`) 
To prevent the core from becoming bloated, Kubernetes introduced named API groups, each representing a specific domain or extension. These are served at /apis/GROUP/VERSION.

Examples of named groups and associated resources:

| Group & Version | Resources |
| --- | --- |
| `apps/v1` | Deployments, StatefulSets, ReplicaSets |
| `batch/v1` | Jobs, CronJobs |
| `rbac.authorization.k8s.io/v1` | Roles, ClusterRoles, RoleBindings |
| `autoscaling/v2` | HorizontalPodAutoscalers (HPA) with scaling policies |
| `networking.k8s.io/v1` | Ingresses, NetworkPolicies |
| `policy/v1` | PodDisruptionBudgets, PodSecurityPolicies *(deprecated)* |
d| `certificates.k8s.io/v1` | CertificateSigningRequests (CSRs) |
d| `admissionregistration.k8s.io/v1` | ValidatingWebhookConfiguration & MutatingWebhookConfiguration |
d| `apiextensions.k8s.io/v1` | CustomResourceDefinitions (CRDs) |
d| `storage.k8s.io/v1` | StorageClasses, VolumeAttachments & CSI drivers


# Authorization in Kubernetes

**Authentication** verifies who you are.
**Authorization** determines what you are allowed to do.

Once a user, group, or service account (SA) is authenticated, Kubernetes must determine whether that identity is allowed to perform the requested action. This process is called **authorization**.

## Authorization questions include:
- Can this user read pods in the default namespace?
- Is this service account allowed to delete a deployment in production?
- Can this user create roles at the cluster level?

Authorization decisions consider not only the user but also the groups they belong to. In Kubernetes, a user or service account can be a member of one or more groups, and permissions may be granted based on either individual identity or group membership.

## Authorization is Context-Aware
A typical Kubernetes authorization decision considers:
- User identity (from authentication)
- Requested verb (get, create, delete, patch, etc.)
- Resource type (pods, deployments, services, etc.)
- Resource name (optional)
- Namespace (if namespaced)
- API group
- Non-resource URLs (for endpoints like /metrics, /healthz)

## Types of Authorization Modes in Kubernetes
Kubernetes supports multiple pluggable authorizers evaluated in order. The first definitive decision (allow/deny) halts the chain.

### 1. RBAC (Role-Based Access Control)
RBAC is the most widely used authorization mode in Kubernetes, especially in production environments.
It works by defining who (user, group, or service account) can perform what actions on which resources.

#### Key Concepts:
| Concept | Description |
|---------|--------------|
| Roles | Define a set of permissions within a namespace |
| RoleBindings | Associate a Role or ClusterRole with specific users, groups, or service accounts within that namespace |
| ClusterRoles | Define permissions that can apply cluster-wide or be reused across multiple namespaces |
| ClusterRoleBindings | Bind a ClusterRole to users, groups, or service accounts at the cluster level |

> **Note:** A RoleBinding can reference a ClusterRole for reusing cluster-defined permissions within a specific namespace. The ClusterRole's rules apply only within that namespace.

### 2. Webhook Authorization
Webhook authorization outsources decision-making to an external service that receives request details and responds with allow/deny.
Common tools:
- OPA (Open Policy Agent)
- Gatekeeper
- Kyverno
- Custom in-house services

#### Why Use Webhooks?
Use when:
eed fine-grained control beyond RBAC,
policies depend on dynamic attributes,
or enforce business-specific rules such as:
deletion restrictions,
deployment constraints,
signed image policies.
> **Note:** Configure with `--authorization-mode=Webhook` and specify webhook config file.
Webhook offers flexibility but adds dependencies and latency. Used mainly in security-sensitive environments.
Note: Validating and mutating admission webhooks are different mechanisms for policy enforcement during object creation/update.

### 3. Node Authorization
Purpose: Restricts kubelet actions based on node-specific permissions.
How It Works: Kubelets authenticate via tokens/certificates; Node Authorization enforces scope limits automatically using node identity.
e.g., kubelet on node1 can only access resources related to node1.
'this enhances security by preventing cross-node access and minimizing impact if compromised.
'this mode is enabled automatically; no manual configuration needed.'
table:
since it’s easier to understand visually:
description | explanation |
before bootstrap token | Kubelet authenticates to API server |
after Node Authorization | Kubelet can only access its own node resources |
the method ensures secure scoped access control per node.

### 4. AlwaysAllow / AlwaysDeny Modes
These are simple modes:
either permit all requests (`AlwaysAllow`) nor deny all requests (`AlwaysDeny`).
definitions & use cases follow:
a) **AlwaysAllow:**
yields full access regardless of user/action — useful for testing/demos but insecure for production.
b) **AlwaysDeny:**
denies all requests — useful for testing failure scenarios but impractical for regular use.
