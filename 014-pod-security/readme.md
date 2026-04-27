Why Pod Security?
Kubernetes lets you run containers easily, but it does not secure them by default. If you spin up a basic pod, it often runs as the root user inside the container, which can be dangerous.

Let’s try it out:

kubectl run debugpod --image=busybox -it --restart=Never -- sh
Then run:

id
Output:

uid=0(root) gid=0(root) groups=0(root),10(wheel)
It means:

uid=0(root): The user ID is 0, which corresponds to the root user.

gid=0(root): The primary group ID is also 0, representing the root group.

groups=0(root),10(wheel):

The user belongs to both the root group (ID 0) and the wheel group (ID 10).
The wheel group traditionally grants users the ability to execute sudo or switch to root.
In summary: UID 0 is what makes a user root. It's not the username but the UID that determines privilege. The rest (username or group names) are just labels mapped from files like /etc/passwd or /etc/group.

But, what is the risk with using the root user?

By default, this pod runs as the root user inside the container — which poses several security risks. Running containers as root is not recommended because:

Privilege Escalation: If the container is compromised, an attacker could attempt to escalate privileges and break out of the container.
Access to Host Resources: In less strictly isolated environments (e.g., without user namespaces or Pod Security Standards), a root container may gain elevated access to the host system.
Accidental Misuse: Applications running as root can unintentionally modify critical files or perform actions that would otherwise be restricted.
Non-compliance: Many regulatory and security standards require workloads to run with the least privilege possible.
Important Note

By default, a pod may run as the root user inside the container — which can pose serious security risks. However, not all pods run as root, even if no explicit security settings are applied in the pod manifest. This is because:

Some container images are designed to run as non-root by default. Their Dockerfiles may include instructions like USER 1000, which ensures the container process does not start with UID 0.
Security-conscious base images (e.g., distroless, bitnami) often avoid running processes as root by design.
Still, relying solely on the image to enforce non-root execution is not sufficient for securing Kubernetes workloads.

As DevOps and Infrastructure engineers, we must:

Explicitly define security context fields (like runAsUser, runAsNonRoot, etc.) in the pod manifest to ensure pods do not run as root — regardless of how the container image was built.
Enforce policies using tools like Pod Security Standards, OPA Gatekeeper, or Kyverno to validate that workloads adhere to these constraints.
This ensures consistency, defense in depth, and aligns with the principle of least privilege, reducing the potential impact of a container compromise.
