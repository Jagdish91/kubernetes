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

# What is RBAC?

Let’s start with the basics.

RBAC stands for **Role-Based Access Control**. It’s a method to regulate access based on a user's role within an organization.

## In Kubernetes:

- **Roles:** Define a set of permissions within a namespace.
- **RoleBindings:** Associate a Role or ClusterRole with specific users, groups, or service accounts within that namespace.
- **ClusterRoles:** Like roles, but define permissions that can apply cluster-wide or be reused across multiple namespaces.
- **ClusterRoleBindings:** Bind a ClusterRole to users, groups, or service accounts at the cluster level, granting access across the entire cluster.

> **Note:** A RoleBinding can reference a ClusterRole, allowing you to reuse cluster-defined permissions in a specific namespace. The ClusterRole's rules will only apply within the namespace where the RoleBinding exists.

## RBAC answers these questions:

- Who is the user?
- What resource do they want to access?
- Which operation do they want to perform?
- Where?

# Starting with Role and RoleBinding (Namespace-Scoped Permissions)

## Important Limitation: Roles Are Strictly Namespace-Scoped

Unlike ClusterRoles, Roles are strictly limited to namespace-scoped resources.

### Here’s why:

- A Role must always be created within a specific namespace.
- If you don’t explicitly set a namespace, it will default to the "default" namespace.
- Therefore, Roles cannot be used to grant access to cluster-scoped resources like Nodes, PersistentVolumes, or Namespaces themselves.

### For example:

You cannot create a Role that allows access to nodes, because nodes are cluster-scoped. Only a ClusterRole can define such permissions.

This limitation makes Roles suitable for fine-grained access control within a single namespace, but not for managing cluster-wide access.

# What Happens When Jag Runs a Command?

When Jag executes:

```bash
kubectl create deployment nginx --image=nginx --namespace=dev
```

Here’s what Kubernetes does behind the scenes:

## Authentication
- The API server verifies that the user is **Seema**.

## Authorization
- The API server looks for:
  - RoleBindings in the **dev** namespace.
  - Finds `bind-workload-editor` that grants the `workload-editor` Role.
  - Confirms that **Jag** is listed in the subjects.
  - Checks the Role has permission to create deployments.

## Execution
- If all checks pass, the deployment is created.

## Key Pointers:
- Roles are used to define namespaced permissions and can have multiple rules.
- RoleBindings assign those Roles to users, groups, or service accounts, and can list multiple subjects.
- Use comments inside YAML to make your manifests clearer — it helps with collaboration and reviews.
- We used **Jag** and a group (**junior-admins**) to show how real-world teams might be structured.

# ClusterRole and ClusterRoleBinding (Cluster-Scoped Permissions)

## Important Clarification: ClusterRoles Aren’t Just for Cluster-Scoped Resources

It’s a common misconception that **ClusterRoles** and **ClusterRoleBindings** are only used for cluster-scoped resources like Nodes or PersistentVolumes.

In reality, you can use them to grant access across all namespaces for resources that are normally namespace-scoped.

### Example:

- Deployments are namespace-scoped resources.
- If you define a **ClusterRole** that allows actions on Deployments, and bind it using a **ClusterRoleBinding**, then the subject (user, group, or service account) will gain those permissions in every namespace.

This is useful when a platform admin or automation service needs consistent access across the entire cluster, without having to create duplicate Roles and RoleBindings in each namespace.

## Quick Recap: RBAC Building Blocks
| Resource | Scope | Purpose | Binds to |
|---|---|---|---|
| Role | Namespace | Define permissions inside a namespace | Used in a RoleBinding |
| RoleBinding | Namespace | Assign Role (or ClusterRole) to subject | User / Group / ServiceAccount |
| ClusterRole | Cluster-wide | Define permissions for all or across namespaces | Used in RoleBinding or ClusterRoleBinding |
| ClusterRoleBinding | Cluster-wide | Assign ClusterRole to subject globally | User / Group / ServiceAccount |

## Real-World Advice & Best Practices
- Always follow the principle of least privilege — grant only what is absolutely needed.
- Use **Role + RoleBinding** for namespace-specific permissions.
- Use **ClusterRole + ClusterRoleBinding** sparingly — only for truly cluster-wide access.
- Prefer binding to groups or service accounts for scalable, manageable access control.
- When using service accounts, remember that their `apiGroup` must be `""` (empty).
- Reuse **ClusterRoles** with **RoleBindings** in namespaces to avoid duplication.
- Regularly audit and review your RBAC setup. Use `kubectl auth can-i` to validate access.
