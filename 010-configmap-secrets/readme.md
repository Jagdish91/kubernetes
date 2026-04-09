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

