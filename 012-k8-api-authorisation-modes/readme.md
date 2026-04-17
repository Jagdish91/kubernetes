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
