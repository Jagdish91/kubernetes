# Namespaced vs Cluster-Scoped Resources and RBAC Verbs

Before we start the discussion on RBAC authorization, let me remind you of a few important concepts:

- Kubernetes resources can be either **namespaced** or **non-namespaced** (cluster-wide).

- **Namespaced resources** exist within a specific namespace (e.g., Pods, Deployments).

- **Non-namespaced resources** are cluster-scoped (e.g., Nodes, PersistentVolumes).

You can check resource names and whether they are namespaced by running:

```bash
kubectl api-resources
```

All Kubernetes resources have **verbs** (actions) associated with them.

These verbs define what operations can be performed on the resources (e.g., create, delete, get).

When we create Roles or ClusterRoles, we use these verbs to specify permissions.

To see the available resources along with their verbs, use:

```bash
kubectl api-resources -o wide
```

## Common RBAC Verbs and Their Descriptions
| Verb | Description |
|--------|--------------|
| create | Create a new resource (e.g., `kubectl create deployment nginx`) |
| delete | Remove a specific resource (e.g., `kubectl delete pod nginx`) |
| deletecollection | Remove multiple resources at once (e.g., `kubectl delete pods --all`) |
| get | Retrieve details of a resource (e.g., `kubectl get pod nginx -o yaml`) |
| list | List resources of a certain type (e.g., `kubectl get pods`) |
| patch | Modify part of an existing resource (e.g., `kubectl patch deployment nginx`) |
| update | Apply changes to an existing resource (e.g., `kubectl apply -f deployment.yaml`) |
| watch | Continuously observe changes in resources (e.g., `kubectl get pods --watch`) |
