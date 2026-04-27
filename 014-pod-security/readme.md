# Why Pod Security?

Kubernetes lets you run containers easily, but it does not secure them by default. If you spin up a basic pod, it often runs as the root user inside the container, which can be dangerous.

## Let’s try it out:

```bash
kubectl run debugpod --image=busybox -it --restart=Never -- sh
```

Then run:

```bash
dd
```
Output:

```plaintext
uid=0(root) gid=0(root) groups=0(root),10(wheel)
```

It means:
- **uid=0(root):** The user ID is 0, which corresponds to the root user.
- **gid=0(root):** The primary group ID is also 0, representing the root group.
- **groups=0(root),10(wheel):** The user belongs to both the root group (ID 0) and the wheel group (ID 10).

The wheel group traditionally grants users the ability to execute sudo or switch to root.

### In summary:
UID 0 is what makes a user root. It's not the username but the UID that determines privilege. The rest (username or group names) are just labels mapped from files like `/etc/passwd` or `/etc/group`.

## But, what is the risk with using the root user?

By default, this pod runs as the root user inside the container — which poses several security risks. Running containers as root is not recommended because:

- **Privilege Escalation:** If the container is compromised, an attacker could attempt to escalate privileges and break out of the container.
- **Access to Host Resources:** In less strictly isolated environments (e.g., without user namespaces or Pod Security Standards), a root container may gain elevated access to the host system.
- **Accidental Misuse:** Applications running as root can unintentionally modify critical files or perform actions that would otherwise be restricted.
- **Non-compliance:** Many regulatory and security standards require workloads to run with the least privilege possible.

## Important Note

By default, a pod may run as the root user inside the container — which can pose serious security risks. However, not all pods run as root, even if no explicit security settings are applied in the pod manifest. This is because:
- Some container images are designed to run as non-root by default. Their Dockerfiles may include instructions like `USER 1000`, which ensures the container process does not start with UID 0.
- Security-conscious base images (e.g., distroless, bitnam) often avoid running processes as root by design.

Still, relying solely on the image to enforce non-root execution is not sufficient for securing Kubernetes workloads.

### As DevOps and Infrastructure engineers, we must:
1. Explicitly define security context fields (`runAsUser`, `runAsNonRoot`, etc.) in the pod manifest to ensure pods do not run as root — regardless of how the container image was built.
2. Enforce policies using tools like Pod Security Standards, OPA Gatekeeper, or Kyverno to validate that workloads adhere to these constraints.

This ensures consistency, defense in depth, and aligns with *the principle of least privilege*, reducing potential impact of a container compromise.

# Pod Security

Hardening Kubernetes workloads begins with Pod Security — ensuring that containers run with the least privilege required. Pod security helps prevent containers from escalating privileges or accessing sensitive host resources, especially in shared or multi-tenant clusters.

Kubernetes provides two primary mechanisms to control pod-level privileges:

- **Security Context (Pod and Container level)**
- **Linux Capabilities (Container level)**

These together form the foundation for enforcing the principle of least privilege, reducing the attack surface, and achieving compliance and runtime security in production environments.

## 1. Security Context

A Security Context defines privilege and access control settings for a Pod or individual containers. It allows you to configure:

- The user and group IDs containers should run as (`runAsUser`, `runAsGroup`)
- Whether containers must run as non-root users (`runAsNonRoot`)
- Volume file ownership through filesystem group (`fsGroup`)
- Privilege escalation restrictions (`allowPrivilegeEscalation`)
- Filesystem mutability (`readOnlyRootFilesystem`)

Security context settings can be applied at:

- **Pod level** — affects all containers in the Pod
- **Container level** — overrides Pod-level settings for that specific container

## Security Context in Kubernetes: Field-by-Field Overview

Kubernetes Security Context allows fine-grained control over pod and container behavior with respect to user identity, filesystem access, and privileges. Below is a concise breakdown organized by where each setting applies.

| Field | Pod Level | Container Level | Notes |
|---|---|---|---|
| `runAsUser` | ✅ | ✅ | Container overrides Pod |
| `runAsGroup` | ✅ | ✅ | Container overrides Pod |
| `runAsNonRoot` | ✅ | ✅ | Container overrides Pod |
| `fsGroup` | ✅ | ❌ | Pod-level only; affects volume ownership |
| `allowPrivilegeEscalation` | ❌ | ✅ | Container-level only |
| `privileged` | ❌ | ✅ | Container-level only |
| `readOnlyRootFilesystem` | ❌ | ✅ | Container-level only |

## Demo: Security Context
In this demo, we will apply security context settings at both the Pod and Container levels and explore how they influence the behavior of the containerized application. This is a hands-on way to understand how various security-related configurations affect runtime identity, access, and capabilities.

# Verification

Once the pod is running, exec into it:

```bash
kubectl exec -it secure-pod -- sh
```

Run the `id` command:

```bash
~ $ id
uid=1000 gid=3000 groups=2000,3000
```

## Explanation:
- **uid=1000**: This comes from `runAsUser: 1000` in the pod’s security context. It means the main process is running as user ID 1000.
- **gid=3000**: This is from `runAsGroup: 3000`, the primary group ID assigned to the process.
- **groups=2000,3000**:
  - `2000` is from `fsGroup`, which grants access to mounted volumes.
  - `3000` is from `runAsGroup`, assigned as the primary GID.

> **Note:** The `fsGroup` setting changes the group ownership of mounted volumes, enabling read/write access as needed for shared volumes or storage mounts.

## Test Read-Only Root Filesystem
Try creating a directory:

```bash
~ $ mkdir test_dir
```
Expected output:
```
mkdir: can't create directory 'test_dir': Read-only file system
```
This error occurs because we set `readOnlyRootFilesystem: true`. It makes the root filesystem (`/`) immutable, including directories like `/etc`, `/var`, `/tmp`, etc.

This enhances security by preventing runtime changes to binaries or config files inside the container.

## Test Privilege Escalation
Try escalating privileges using common methods:

```bash
~ $ sudo su
tsh: sudo: not found
```
and:
```bash
y$ su -
su: must be suid to work properly
```
These attempts fail because:
- The image does not include `sudo` or `su` by default.
- Even if `su` was available, `allowPrivilegeEscalation: false` prevents privilege gain via setuid binaries.

# Linux Capabilities

In Linux, root is an all-powerful user — but in reality, root privileges are made up of many distinct low-level permissions called capabilities. Kubernetes allows you to selectively manage these capabilities at the container level, giving you fine-grained control over what a container can and cannot do.

Capabilities are configured via the `securityContext.capabilities` field and are only supported at the container level — not at the pod level.

By default, containers inherit a reduced set of Linux kernel capabilities compared to a full root user. However, they still retain some sensitive capabilities (like `NET_RAW` or `SYS_ADMIN`) unless explicitly dropped.

With Kubernetes, you can use the `capabilities` field inside the `securityContext` to:

- Drop unnecessary kernel privileges (`drop: ["ALL"]`)
- Add only what's needed (`add: ["NET_BIND_SERVICE"]`)

This gives fine-grained control over what your containerized process can do — beyond what basic UID/GID settings offer.

## What Are Linux Capabilities?

Instead of giving full root access to a container, Linux capabilities allow you to drop unnecessary privileges and optionally add only the ones required.

This aligns with the principle of least privilege, helping reduce the attack surface and prevent containers from performing dangerous operations.

## Best Practice with Linux Capabilities

By default, containers — even when not running as root — are granted a minimal set of Linux capabilities such as `CHOWN`, `SETUID`, `NET_BIND_SERVICE`, and `KILL` to enable essential operations like changing file ownership or binding to low-numbered ports.

However, if you explicitly drop all capabilities using `capabilities.drop: ["ALL"]`, even these default privileges are removed. As a result, many standard operations will fail — demonstrating that simply running as root (`UID 0`) doesn't equate to full privilege without the necessary capabilities.

Always tailor capabilities to your application's needs. Start with none and add only what is absolutely required.

## Common Linux Capabilities
to be added here for clarity:
| Capability | Description |
| --- | --- |
| NET_ADMIN | Modify network interfaces, routing tables, and firewall rules |
| SYS_ADMIN | Very broad; includes mounting filesystems, changing kernel params, system configuration changes |
| SYS_TIME | Set system clock, modify hardware clock, manage time sync services (e.g., NTP) |
| CHOWN | Change file ownership using chown |
| SETUID | Change user ID of a running process |
| DAC_OVERRIDE | Override standard file permission checks (Discretionary Access Control) |

Base images may include some of these by default. Explicitly dropping unneeded capabilities is key to reducing the container’s privilege footprint.
