# Node Info Collector - Kubernetes CronJob

## Overview

This project defines a Kubernetes `CronJob` that periodically collects system information (such as uptime, users, CPU/memory/disk usage) from cluster nodes. The output is saved to a shared NFS volume, making it easy to analyze logs for diagnostics or auditing.

---

## Architecture

- **CronJob Name**: `node-info`
- **Namespace**: `demo`
- **Schedule**: Every hour (`* */1 * * *`)
- **Execution**: One pod runs per job; can be scaled by setting `parallelism` and `completions`.

---

## Functionality

1. Uses `hostPID: true` to access host-level process information.
2. Runs standard Linux commands like `uptime`, `who`, `top`, and `df` to gather system status.
3. Writes output to a log file named:
`node_<node-name>sysinfo<timestamp>.log`
and stores it in `/mnt/logs` on a shared NFS mount.
4. Sleeps for 300 seconds after execution for debugging or observation purposes.

---

## Resource Configuration

### CronJob

- **Container Image**: `spiqyq.nat.fia.gov.tw/redhat/ubi:latest`
- **Service Account**: `node-command-executor-sa`
- **Tolerations**: Allows scheduling on `master` or `infra` nodes.
- **Anti-Affinity**: Ensures only one pod per node by avoiding co-scheduling.

### Persistent Volume / Claim

- **NFS Server**: `10.190.33.171`
- **Path**: `/os_downtime_logs`
- **PV Name**: `nfs-log-pv`
- **PVC Name**: `nfs-log-pvc`
- **Storage Capacity**: `10Gi`

---

## RBAC & Permissions

- A `ServiceAccount` named `node-command-executor-sa` is created.
- It is bound to the `ClusterRole` `system:openshift:scc:privileged` to allow privileged operations (required in OpenShift environments).

---

## Environment Variables

These are injected using the Kubernetes Downward API:

- `MY_NODE_NAME`: Name of the node where the pod is running
- `NODE_IP`: IP address of the node
- `POD_NAME`: Name of the pod
- `POD_NAMESPACE`: Namespace of the pod
- `POD_IP`: Pod's IP address

---

## Notes

- To run on **all nodes**, set `parallelism` and `completions` equal to the number of nodes, and use `podAntiAffinity` to spread the pods.
- Ensure NFS permissions and network access are properly configured for all nodes.
- OpenShift users must confirm the ServiceAccount has access to `privileged` SCC.

---

## Use Cases

- Automated node health checks
- Periodic system logging for diagnostics and compliance
- Reference example for privileged pods collecting host-level data
