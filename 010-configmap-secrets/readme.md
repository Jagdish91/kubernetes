# Introduction

As modern applications grow in complexity, separating configuration and secrets from application logic becomes essential for portability, maintainability, and security. Kubernetes provides native support for this through ConfigMaps and Secrets. ConfigMaps help manage non-sensitive configuration data, while Secrets handle sensitive data like passwords and API keys.

In this session, we will learn how to use ConfigMaps and Secrets effectively in Kubernetes. Through hands-on demos, you’ll explore injecting configurations as environment variables, command-line arguments, and files, and understand how Kubernetes handles updates to these objects at runtime. You’ll also learn the key differences between ConfigMaps and Secrets, and best practices for secure and scalable usage in real-world environments.

## ConfigMaps

### Why Do We Need ConfigMaps?
In real-world Kubernetes deployments:
- Maintaining a large number of hardcoded environment variables in Pod specs can become unmanageable.
- You may want to update configuration without rebuilding your container image.
- You might need to inject configuration data via environment variables, CLI arguments, or mounted files.

ConfigMaps allow you to decouple configuration from your container images, making your applications more portable and easier to manage.

### What is a ConfigMap?
A ConfigMap is a Kubernetes API object used to store non-confidential key-value configuration data.

Pods can consume ConfigMaps in three primary ways:
1. As environment variables
2. As command-line arguments (Less Common)
3. As configuration files via mounted volumes
**Important:** ConfigMaps do not provide encoding. For sensitive information, use a Secret instead.

## What are Environment Variables?

Environment variables are key-value pairs used by the operating system and applications to store configuration settings. They help control how processes behave without modifying the underlying code.

### Example on a Linux system:
```bash
export APP_NAME=frontend
export ENVIRONMENT=production
```
You can access these values using the echo command:
```bash
echo $APP_NAME     # Output: frontend
echo $ENVIRONMENT  # Output: production
```
These variables are commonly used in shell scripts or passed to applications during execution to configure them dynamically.

## Without ConfigMap: Hardcoded Environment Variables
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
        - name: frontend-container
          image: nginx
          env:
            - name: APP
              value: frontend
```

# Verification

## Pick a Pod from the Deployment:

```bash
kubectl get pods -l app=frontend
```

## Then exec into one of them:

```bash
kubectl exec -it <pod-name> -- printenv | grep -E 'APP|ENVIRONMENT'
```

## Expected Output:

```
APP=frontend
ENVIRONMENT=production
```

# Creating ConfigMaps in Kubernetes

You can also create a **ConfigMap** imperatively using the `kubectl create configmap` command.

## Example:
```bash
kubectl create configmap <name-of-configmap> --from-literal=key1=value1 --from-literal=key2=value2
```

## Real-world example:
```bash
kubectl create configmap frontend-cm --from-literal=APP=frontend --from-literal=ENVIRONMENT=production
```

This method is quick and useful for creating simple ConfigMaps imperatively from the command line, without needing to write and apply a YAML manifest.

---

# When to Use Imperative vs Declarative Approach

## Imperative Commands (`kubectl create configmap`) are best when you:
- Need to quickly create a ConfigMap for testing or small demos.
- Are experimenting or working interactively.
- Don't need to track the resource in version control (like Git).

## Declarative Approach (YAML manifests + `kubectl apply -f`) is preferred when you:
- Are working in production environments.
- Want your ConfigMaps (and all Kubernetes resources) to be version-controlled.
- Need better team collaboration, auditing, and repeatable deployments.

---

# Rule of Thumb:
- Use imperative for quick, temporary tasks.
- Use declarative for production-grade, repeatable, and auditable setups.

# Step 2: Mount ConfigMap Volume in Deployment

Understand that ConfigMaps are mounted as directories in Kubernetes. If you try to mount a ConfigMap at a path that is already a file (like `/usr/share/nginx/html/index.html`), the container will fail to start because Kubernetes cannot mount a directory over an existing file.

That’s why we only use `/usr/share/nginx/html` as the `mountPath`, which is a directory, not a file.

## Omitting the `items:` Section
If you omit the `items:` section, like this:

```yaml
volumes:
  - name: html-volume
    configMap:
      name: frontend-cm
```

Then all the keys in the ConfigMap (`APP`, `ENVIRONMENT`, `index.html`) will be mounted as individual files inside `/usr/share/nginx/html`.

### Inside the container, you’d get:
- `/usr/share/nginx/html/APP`
- `/usr/share/nginx/html/ENVIRONMENT`
- `/usr/share/nginx/html/index.html`

## Mounting Only Specific Files with `items:`
If you only want `index.html` to be mounted (and not `APP`, `ENVIRONMENT`), you should use the `items:` section like this:

```yaml
volumes:
  - name: html-volume
    configMap:
      name: frontend-cm
      items:
        - key: index.html
          path: index.html
```
# Kubernetes Secrets

## Why Do We Need Secrets in Kubernetes?

In production environments, applications often require access to sensitive data such as:

- **Database credentials:** Used by applications to authenticate securely with backend databases.
- **API tokens:** Serve as secure keys to authorize and access APIs or third-party services.
- **SSH private keys:** Enable secure, encrypted access to remote systems over SSH.
- **TLS certificates:** Provide encryption and identity verification for secure network communication (e.g., HTTPS).

Storing these directly in your container image or Kubernetes manifests as plain text is insecure. Kubernetes Secrets provide a way to manage this data securely.

## What is a Kubernetes Secret?

A Secret is a Kubernetes object used to store and manage sensitive information. Secrets are base64-encoded and can be made more secure by:

- Enabling encryption at rest
- Limiting RBAC access to Secrets
- Using external secret managers like HashiCorp Vault, AWS Secrets Manager, or Sealed Secrets

Secrets are accessible to Pods via:

- Environment variables
- Mounted volumes (as files)
- Command-line arguments (less common)

> **Note:** By default, Kubernetes stores secrets unencrypted in etcd. It is recommended to enable encryption at rest for better security.

## Important Distinction: Encoding vs. Encryption

It's crucial to understand that Kubernetes Secrets use base64 encoding, not encryption.
This means the data is obfuscated but not secured. Anyone who gains access to the Secret object can easily decode it.

### Why Use Encoding (e.g., base64)?

Encoding is useful when you want to hide the data from casual observation, such as:

- Preventing someone looking over your shoulder from instantly seeing a password.
- Making binary data safe to transmit in systems that expect text.

However, encoding is not encryption. It's not secure by itself.
Anyone who has access to your encoded data can easily decode it.

For example, base64 is reversible using a simple decoding command.
If you need to protect sensitive data, you should use encryption or a Kubernetes Secret, which at least provides better handling and access controls.

We'll see how to encode and decode in the demo section.

## Encoding vs. Encryption Comparison Table:
| Feature | Encoding | Encryption |
|---------|----------|------------|
| Purpose | Data formatting for safe transport | Data protection and confidentiality |
| Reversible | Yes (easily reversible) | Yes (only with the correct key) |
| Security | Not secure | Secure |
| Use Case | Data transmission/storage compatibility | Protect sensitive data (passwords, tokens) |
| Example | Base64, URL encoding | AES, RSA, TLS |
| Tool Needed to Decode | None (any base64 tool) | Requires decryption key |
> **Note:** If you need to store sensitive data securely, consider enabling encryption at rest for Secrets in Kubernetes and restrict access using RBAC.
# Best Practices for Using Kubernetes Secrets

- Avoid storing secrets in Git repositories.
- Use `kubectl create secret` or Helm to avoid manually encoding data.
- Enable encryption at rest in etcd.
- Use Role-Based Access Control (RBAC) to restrict access to secrets.
- Use external secret managers for enhanced security and auditability.
- Consider using `subPath` when mounting specific keys from a Secret as individual files.

# Understanding Dynamic Updates with ConfigMaps and Secrets

- Environment variables defined via `env.valueFrom.configMapKeyRef` or `secretKeyRef` are evaluated only once when the pod starts.
- Updating the underlying ConfigMap or Secret does not affect the values already injected as environment variables in a running container.
- ConfigMaps or Secrets mounted as volumes (without `subPath`) do reflect updates dynamically inside the container.
- Kubernetes handles dynamic updates using symlinks to new file versions, but the application must re-read the files to detect changes.
- When mounting individual keys using `subPath`, the file is copied, not symlinked, so updates to the ConfigMap or Secret will not propagate.

**To enable live updates without restarting pods:**

- Prefer volume mounts without `subPath`
- Ensure the application supports hot reloading or use a `config-reloader`."} }```
