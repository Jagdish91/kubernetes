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

# Persistent Volumes (PVs) & Persistent Volume Claims (PVCs)

## Persistent Volumes (PVs)
### What is a PV?
A PV is a piece of storage in your cluster that has been provisioned either manually by an administrator or dynamically using Storage Classes.

### Key Characteristics:
- PVs exist independently of Pod lifecycles and can be reused or retained even after the Pod is deleted.
- They have properties such as capacity, access modes, and reclaim policies.

## Persistent Volume Claims (PVCs)
### What is a PVC?
A PVC is a request for storage by a user. It functions similarly to how a Pod requests compute resources. When a PVC is created, Kubernetes searches for a PV that meets the claim's requirements.

### Binding Process:
1. **Administrator:** Provisions PVs (or sets up Storage Classes for dynamic provisioning).
2. **Developer:** Creates a PVC in the Pod specification requesting specific storage attributes.
3. **Kubernetes:** Binds the PVC to a suitable PV, thereby making the storage available to the Pod.

Pods rely on Node resources—such as CPU, memory, and network—to run containers. When a Pod requires persistent storage, it uses a PersistentVolumeClaim (PVC) to request storage from a PersistentVolume (PV), which serves as the actual storage backend. This separation of compute and storage allows Kubernetes to manage them independently, improving flexibility and scalability.

## Understanding Scope & Relationships of PV and PVC in Kubernetes
In Kubernetes, PersistentVolumes (PVs) and PersistentVolumeClaims (PVCs) play a central role in persistent storage — but they differ in how they're scoped and used.

### PVs are Cluster-Scoped Resources
- A PersistentVolume (PV) is a cluster-wide resource, just like Nodes or StorageClasses.
- This means it is not tied to any specific namespace, and it can be viewed or managed from anywhere within the cluster.
- You can verify this using:
  ```bash
  kubectl api-resources | grep persistentvolume
  ```
- This shows that the resource `persistentvolumes` has no namespace, indicating it's cluster-scoped.

### PVCs are Namespace-Scoped
- A PersistentVolumeClaim (PVC), on the other hand, is a namespaced resource, just like Pods or Deployments.
- This means it exists within a specific namespace and is only accessible by workloads (Pods) within that namespace.
- You can verify this using:
  ```bash
  kubectl api-resources | grep persistentvolumeclaim
  ```
- This shows that `persistentvolumeclaims` are scoped within a namespace.

### Why Is This Important?
Let’s say you have a namespace called `app1-ns`. If a PVC is created in `app1-ns` and binds to a PV, only Pods in `app1-ns` can use that PVC.

If a Pod in `app2-ns` tries to reference the same PVC, it will fail — because the PVC is invisible and inaccessible outside its namespace.

## 1-to-1 Binding Relationship Between PVC and PV
- A PVC can bind to only one PV.
- Similarly, a PV can be bound to only one PVC.
- This is a strict one-to-one relationship, ensuring data integrity and predictable access control.
tOnce bound, its `claimRef` field is populated; it cannot be claimed by any other PVC unless explicitly released.
details about `claimRef`:
details like the PVC’s name and namespace are recorded here,
enforcing that each PV corresponds to only one claiming PVC at any time.

## Additional Key Points
- PVCs request storage; PVs fulfill that request if they match capacity, access mode, and storage class.
- Once bound:
  - The PVC remains bound until it is deleted,
or 
dthe PV is manually reclaimed or deleted (depending on reclaim policy).
the reclaim policy options include: **Retain**, **Delete**, or deprecated **Recycle**,
and determine what happens after deletion of the associated PVC.

# Example Scenario

Let’s say:

- You create a PVC named `data-pvc` in the namespace `app1-ns`.
- It binds to a cluster-scoped PV.
- Only Pods in `app1-ns` can now reference this PVC.
- If a Pod in `app2-ns` tries to mount this PVC, it will result in an error like:

```
persistentvolumeclaims "data-pvc" not found
```

Because from `app2-ns`'s perspective, that PVC does not exist.

# Kubernetes Persistent Storage Flow (Manual Provisioning)

| Step | Role | Action | Details / Notes |
|-------|--------------|------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------|
| 1 | Developer | Requests 5Gi persistent storage for a Pod. | May request via a PVC or through communication with the Kubernetes Admin. |
| 2 | Kubernetes Admin | Coordinates with Storage Admin for backend volume. | Backend storage could be SAN/NAS exposed via iSCSI, NFS, etc. |
| 3 | Storage Admin | Allocates a physical volume from a 500Ti storage pool. | May involve LUN creation, NFS export, etc., based on the infrastructure. |
| 4 | Kubernetes Admin | Creates a PersistentVolume (PV) representing the physical volume in Kubernetes. | Specifies capacity, accessModes, volumeMode, storageClassName, etc. |
| 5 | Developer | Creates a PersistentVolumeClaim (PVC) requesting 5Gi with specific access and volume modes. | PVC must match criteria defined in the PV. |
| 6 | Kubernetes | Binds PVC to a suitable PV if all parameters match. | Matching criteria include: storage class, access mode, volume mode, size, etc. |
| 7 | Pod | References the PVC in its volume definition and mounts it in a container. | PVC acts as an abstraction; Pod doesn’t interact with the PV directly.

# Important Notes

- **PV** is a cluster-scoped resource.
- **PVC** is a namespaced resource.
- One **PV** can be claimed by only one **PVC** (1:1 relationship).
- The Pod must be in the same namespace as the PVC it is using.
- Communication with physical storage is handled by either:
  - In-tree drivers (legacy; e.g., `awsElasticBlockStore`, `azureDisk`)
  - CSI drivers (modern; e.g., `ebs.csi.aws.com`, `azurefile.csi.azure.com`)

In many cases, developers are well-versed with Kubernetes and can handle the creation of PersistentVolumeClaims (**PVCs**) themselves. With the introduction of StorageClasses, the process of provisioning PersistentVolumes (**PVs**) has been automated—eliminating the need for Kubernetes administrators to manually coordinate with storage admins and pre-create PVs.

When a PVC is created with a StorageClass, Kubernetes dynamically provisions the corresponding PV. We’ll explore StorageClasses in detail shortly.

# Access Modes in Kubernetes Persistent Volumes

Persistent storage in Kubernetes supports various access modes that dictate how a volume can be mounted. Access modes essentially govern how the volume is mounted across nodes, which is critical in clustered environments like Kubernetes.

| Access Mode | Description | Example Use Case | Type of Storage & Examples |
|--------------|--------------|------------------|---------------------------|
| **ReadWriteOnce (RWO)** | The volume can be mounted as read-write by a single node. Multiple Pods can access it only if they are on the same node. | Databases that require exclusive access but may run multiple replicas per node. | Block Storage (e.g., Amazon EBS, GCP Persistent Disk, Azure Managed Disks) |
| **ReadOnlyMany (ROX)** | The volume can be mounted as read-only by multiple nodes simultaneously. | Sharing static data like configuration files or read-only datasets across multiple nodes. | File Storage (e.g., NFS, Azure File Storage) |
| **ReadWriteMany (RWX)** | The volume can be mounted as read-write by multiple nodes simultaneously. | Content management systems, shared data applications, or log aggregation. | File Storage (e.g., Amazon EFS, Azure File Storage, On-Prem NFS) |
| **ReadWriteOncePod (RWOP)** *(Introduced in v1.29)* | The volume can be mounted as read-write by only one Pod across the entire cluster. | Ensuring exclusive access to a volume for a single Pod, such as in tightly controlled workloads. | Block Storage (e.g., Amazon EBS with ReadWriteOncePod enforcement) |
