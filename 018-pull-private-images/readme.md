# Introduction

In real-world Kubernetes deployments, not all images live in public registries. Organizations often rely on private container registries to securely store production-ready images, internal tools, and microservices.

This lecture focuses on how Kubernetes interacts with private registries, how to configure authentication using `imagePullSecrets`, and the role of ServiceAccounts in making this process seamless and secure.

## Defaulting Behavior in Kubernetes
Kubernetes (and Docker) provide default fallbacks:

- If the registry is omitted: it defaults to `docker.io`
- If the namespace is omitted: it defaults to `library/` for official images
- If the tag is omitted: it defaults to `latest`

So, this:

```yaml
image: nginx
```

Is interpreted as:

```plaintext
docker.io/library/nginx:latest
```

## Explicit Examples
For public images under a user or organization:

```yaml
image: docker.io/cloudwithvarjosh/my-python-image:v1
```

## How Kubernetes Authenticates to Private Image Registries
Kubernetes itself does not pull container images. This responsibility is delegated to the Container Runtime Interface (CRI) — such as containerd or CRI-O — running on each node.

To authenticate to private registries, Kubernetes uses `imagePullSecrets`, which store credentials in a format compatible with Docker (`kubernetes.io/dockerconfigjson`). These credentials typically include:
- Registry URL
- Username and password
- Base64-encoded auth token

When a pod is created, the kubelet on the node:
1. Receives the pod spec and identifies the image to be pulled.
2. Fetches any referenced `imagePullSecrets` — either directly from the pod spec or indirectly from the associated ServiceAccount.
3. Passes the credentials to the CRI (e.g., containerd), which uses them to authenticate with the image registry and pull the image.

## Default Behavior: ServiceAccount and Secret Mounting
default, every namespace in Kubernetes comes with a ServiceAccount named `default`, and any pod created without explicitly specifying a `serviceAccountName` will be automatically associated with this default account.
This default ServiceAccount is also automatically mounted into the pod at runtime via a projected volume under:
`/var/run/secrets/kubernetes.io/serviceaccount/`

## Configuring Registry Access
two supported ways to provide private registry credentials:
1. **Pod-level** `imagePullSecrets`: Add the secret explicitly in the pod spec. This gives control per workload.
2. **Namespace-level** via ServiceAccount: Attach the secret to a ServiceAccount (e.g., default) so that all pods using that SA automatically inherit registry credentials.
This approach is cleaner for multi-pod setups in a namespace where the same registry credentials apply.
Whether using the default or a custom ServiceAccount, Kubernetes ensures that credentials are used only at image pull time, and they are never mounted into containers or available as environment variables inside pods.

# Demo 1: Authenticate to a Private Registry Using `imagePullSecrets`

In this demo, we'll:

- Create a Docker registry secret with private credentials
- Use that secret in a pod
- Decode and inspect the secret
- Understand why this must be handled carefully

## Step 1: Create a Docker Registry Secret
Use the following command to create a Kubernetes secret for authenticating with Docker Hub:

```bash
kubectl create secret docker-registry dockerhub-secret \
  --docker-server=https://index.docker.io/v1/ \
  --docker-username=youprivateregistry \
  --docker-password=YOUR_PASSWORD \
  --docker-email=your-email
```
Verify the secret:

```bash
kubectl get secret dockerhub-secret -o yaml
```
This secret is of type `kubernetes.io/dockerconfigjson` and stores a base64-encoded `.dockerconfigjson`.

# Demo 2: Configure Default ServiceAccount with Pull Secret (Namespace-Wide)
In this demo, we eliminate the need to add `imagePullSecrets` to every pod manifest by configuring the default ServiceAccount in a namespace to use the pull secret automatically.

## Step 1: Create Namespace and Confirm Default ServiceAccount
Every new namespace in Kubernetes automatically includes a default ServiceAccount named `default`.

```bash
kubectl create ns app1-ns
tkubectl get sa -n app1-ns
```

## Step 2: Create the Pull Secret in the New Namespace
*This step involves creating the pull secret within your namespace, similar to how it was created in Demo 1.*

## Step 3: Patch the Default ServiceAccount (Option 1: CLI)
Attach the `dockerhub-secret` to the default SA in the new namespace:

```bash
ykubectl patch serviceaccount default \
  -n app1-ns \
  -p '{"imagePullSecrets": [{"name": "dockerhub-secret"}]}'
```
