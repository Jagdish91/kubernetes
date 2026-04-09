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
defaults;
the rest of the YAML content continues...
