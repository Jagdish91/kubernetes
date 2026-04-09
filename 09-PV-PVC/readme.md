# Kubernetes Storage Overview

Kubernetes supports CSI (Container Storage Interface) plugins for both file (NFS) and block storage options. These plugins are provided by storage vendors such as NetApp, Dell EMC, and cloud providers like AWS, Azure, and Google Cloud Platform (GCP). CSI plugins enable Kubernetes Pods to access file and block storage seamlessly, offering advanced features like dynamic provisioning, snapshots, and volume resizing.

## Object Storage in Kubernetes

For object storage solutions like Amazon S3, Azure Blob Storage, or Google Cloud Storage (GCS), Kubernetes does not natively support mounting object storage as volumes because object storage operates differently from file systems. Instead, it is recommended to use application-specific SDKs or APIs to interact with object storage. These SDKs allow applications to efficiently access and manage object storage for tasks such as retrieving files, uploading data, and performing operations like versioning or replication.

## Use Cases: File/Block vs Object Storage
- **File and Block Storage:** Designed for direct mounting and integration with Kubernetes workloads using CSI plugins.
- **Object Storage:** Better suited for application-level access via APIs or SDKs, as it was not designed to be mounted like file systems.

By leveraging CSI plugins for file and block storage, and SDKs for object storage, you can make the most of modern, scalable storage options tailored to your Kubernetes workloads.

## Transition from In-tree Drivers to CSI Drivers
Earlier versions of Kubernetes supported a lot of built-in drivers called in-tree plugins. Today, the ecosystem has fully transitioned to CSI drivers — the future-proof way to provision and consume persistent storage. Whether dealing with AWS EBS, NFS, iSCSI — if you're doing it the modern way — you're using a CSI driver.

# Persistent Storage in Kubernetes

## hostPath: A Basic Persistent Storage Option
While Kubernetes does not generally recommend using `hostPath` due to its limitations and security concerns, it can still be useful in specific scenarios.

### What is hostPath?
A `hostPath` volume mounts files or directories from the host node’s filesystem directly into the Pod. These volumes are node-specific; all Pods running on the same node can access shared data stored in a `hostPath` volume. However,
- Pods on different nodes cannot share the same `hostPath` volume because each node has its own independent storage.
- It is unsuitable for distributed workloads requiring cross-node data sharing.

### Why is hostPath Used?
It is used for scenarios such as:
- Debugging
- Accessing host-level files (logs, configuration files)
- Sharing specific host resources with containers.

### Why Kubernetes Recommends Avoiding hostPath:
- **Security Risks:** Pods can access sensitive host files if misconfigured.
- **Limited Portability:** Tied to a specific node; reduces flexibility in scheduling.
- **Better Alternatives:** Use PersistentVolumes (PV) with PersistentVolumeClaims (PVC) or local PersistentVolumes for improved control and security.
