# Understanding DaemonSets in Kubernetes

A **DaemonSet** ensures that exactly one Pod runs on each node in the cluster.
- As new nodes are added to the cluster, Kubernetes automatically schedules a Pod onto the new node.
- Similarly, when nodes are removed, Kubernetes garbage-collects (deletes) the DaemonSet Pods on those nodes.
- If you delete a Pod created by a DaemonSet manually, the DaemonSet controller will immediately recreate the Pod to maintain the desired state.

Deleting a DaemonSet itself will clean up all the Pods it created.

## Use Cases for DaemonSets
DaemonSets are typically used to deploy cluster-wide system services, such as:

- Monitoring agents (e.g., Prometheus Node Exporter, Datadog Agent)
- Logging agents (e.g., Fluentd, Fluent Bit, Filebeat)
- Networking plugins (e.g., CNI plugins like Calico, Flannel)
- Storage plugins (e.g., CSI Node Drivers)
- Security agents (e.g., Falco runtime security monitors)
- Telemetry collectors (e.g., OpenTelemetry Collector DaemonSet)

DaemonSets ensure that these critical agents are automatically rolled out across all existing and new nodes, without any manual intervention.

## Demo: DaemonSet - Deploying a Dummy Logging Agent
In this demo, we will deploy a dummy logging agent across all nodes in our Kubernetes cluster using a DaemonSet.
The agent will simulate log collection by printing a message every 30 seconds.

> As a best practice, system-level DaemonSets are typically deployed into their own dedicated namespace for better organization and access control.

### Step 1: Create a New Namespace for Logging
We will create a new namespace called `logging-ns` to isolate our DaemonSet:
```bash
kubectl create namespace logging-ns
```

### Step 2: Switch the Context to the New Namespace
To avoid typing `-n logging-ns` with every command, we will temporarily set the default namespace in our current context:
```bash
kubectl config set-context --current --namespace=logging-ns
```
This ensures that all our upcoming commands automatically target the `logging-ns` namespace.

### Step 3: Apply the DaemonSet Manifest


# Deployment vs DaemonSet vs Job vs CronJob

| Aspect | Deployment | DaemonSet | Job | CronJob |
| --- | --- | --- | --- | --- |
| **Purpose** | Run long-running stateless applications with scalability and rolling updates. | Ensure one Pod runs on every (or selected) node. | Run a task once to successful completion. | Schedule Jobs to run at specific times (like a cron job). |
| **Common Use Cases** | Web servers, APIs, frontend apps. | Node-level monitoring, logging agents, networking/storage plugins. | Database migration, batch jobs, backups, one-time tasks. | Scheduled backups, nightly report generation, log rotation. |
| **Examples** | Deploying a Node.js API server, Nginx deployment, React app backend. | kube-proxy, Fluentd, Calico CNI, AWS EBS CSI Node plugin. | Backup database, clean temporary files, send a one-time notification. | Backup database every night, rotate logs weekly, send monthly invoices. |

