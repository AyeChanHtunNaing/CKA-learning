# Cluster Maintenance

This folder contains Kubernetes **Cluster Maintenance** notes for CKA preparation.  
The notes focus on node maintenance, Kubernetes cluster upgrades, kubeadm upgrade workflows, and backup/restore strategies.

---

## 📌 Topics Covered

| No. | Topic | Note File |
|---|---|---|
| 1 | OS Upgrades | [`kubernetes_os_upgrades_note.md`](./kubernetes_os_upgrades_note.md) |
| 2 | Cluster Upgrade Introduction | [`kubernetes_cluster_upgrade_introduction_note.md`](./kubernetes_cluster_upgrade_introduction_note.md) |
| 3 | Demo Cluster Upgrade | [`kubernetes_demo_cluster_upgrade_note.md`](./kubernetes_demo_cluster_upgrade_note.md) |
| 4 | Backup and Restore Methods | [`kubernetes_backup_restore_methods_note.md`](./kubernetes_backup_restore_methods_note.md) |

---

## 🎯 Learning Goals

After studying this section, you should understand how to:

- Safely prepare Kubernetes nodes for maintenance
- Use `cordon`, `drain`, and `uncordon`
- Upgrade worker nodes with minimal downtime
- Understand Kubernetes version skew rules
- Upgrade Kubernetes clusters one minor version at a time
- Use `kubeadm upgrade plan`
- Upgrade control plane components
- Upgrade `kubeadm`, `kubelet`, and `kubectl`
- Back up Kubernetes resources using YAML manifests
- Create and restore etcd snapshots
- Understand when to use Velero or API-based backup methods

---

## 🧭 Recommended Study Order

Study the notes in this order:

1. **OS Upgrades**
2. **Cluster Upgrade Introduction**
3. **Demo Cluster Upgrade**
4. **Backup and Restore Methods**

This order is recommended because node maintenance commands like `drain` and `uncordon` are used again during cluster upgrades.

---

## ⚡ CKA Quick Commands

### Check Nodes

```bash
kubectl get nodes
kubectl get pods -A -o wide
```

---

## Node Maintenance

### Cordon a Node

`cordon` marks a node as unschedulable. Existing Pods continue running, but no new Pods will be scheduled on that node.

```bash
kubectl cordon <node-name>
```

Example:

```bash
kubectl cordon node-1
```

---

### Drain a Node

`drain` safely evicts Pods from a node and marks the node as unschedulable.

```bash
kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data
```

Example:

```bash
kubectl drain node-1 --ignore-daemonsets --delete-emptydir-data
```

---

### Uncordon a Node

After maintenance or upgrade, make the node schedulable again.

```bash
kubectl uncordon <node-name>
```

Example:

```bash
kubectl uncordon node-1
```

---

## Cordon vs Drain

| Command | Existing Pods | New Pods | Common Use Case |
|---|---|---|---|
| `cordon` | Keep running | Not scheduled | Stop new workloads only |
| `drain` | Evicted | Not scheduled | Node maintenance / OS upgrade |
| `uncordon` | No direct effect | Scheduling allowed again | After maintenance |

---

## OS Upgrade Flow

A common node maintenance flow:

```bash
kubectl drain node-1 --ignore-daemonsets --delete-emptydir-data

# Perform OS upgrade / reboot / maintenance

kubectl uncordon node-1
kubectl get nodes
kubectl get pods -A -o wide
```

---

## Cluster Upgrade Concepts

### Version Skew Rule

The `kube-apiserver` is the main control plane component. Other Kubernetes components should not run at a version higher than the API server.

Easy memory:

```text
kube-apiserver = highest version
controller-manager / scheduler = up to 1 minor lower
kubelet / kube-proxy = up to 2 minor lower
kubectl = can be higher, lower, or same within supported skew
```

---

### Upgrade One Minor Version at a Time

Recommended:

```text
v1.28 → v1.29 → v1.30
```

Not recommended:

```text
v1.28 → v1.30
```

---

## kubeadm Upgrade Flow

### Control Plane Upgrade

```bash
sudo apt-cache madison kubeadm

sudo apt-mark unhold kubeadm && sudo apt-get update && sudo apt-get install -y kubeadm='<target-version>' && sudo apt-mark hold kubeadm

kubeadm version

sudo kubeadm upgrade plan

sudo kubeadm upgrade apply v<target-version>
```

Example:

```bash
sudo apt-mark unhold kubeadm && sudo apt-get update && sudo apt-get install -y kubeadm='1.29.3-1.1' && sudo apt-mark hold kubeadm

sudo kubeadm upgrade plan
sudo kubeadm upgrade apply v1.29.3
```

---

### Upgrade kubelet and kubectl on Control Plane

```bash
kubectl drain controlplane --ignore-daemonsets

sudo apt-mark unhold kubelet kubectl && sudo apt-get update && sudo apt-get install -y kubelet='<target-version>' kubectl='<target-version>' && sudo apt-mark hold kubelet kubectl

sudo systemctl daemon-reload
sudo systemctl restart kubelet

kubectl uncordon controlplane
kubectl get nodes
```

Example:

```bash
sudo apt-get install -y kubelet='1.29.3-1.1' kubectl='1.29.3-1.1'
```

---

## Worker Node Upgrade Flow

Upgrade worker nodes one at a time.

```bash
# On worker node
sudo apt-mark unhold kubeadm && sudo apt-get update && sudo apt-get install -y kubeadm='<target-version>' && sudo apt-mark hold kubeadm

sudo kubeadm upgrade node
```

Drain worker node from the control plane:

```bash
kubectl drain node01 --ignore-daemonsets --delete-emptydir-data
```

Upgrade kubelet and kubectl on the worker node:

```bash
sudo apt-mark unhold kubelet kubectl && sudo apt-get update && sudo apt-get install -y kubelet='<target-version>' kubectl='<target-version>' && sudo apt-mark hold kubelet kubectl

sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

Uncordon from the control plane:

```bash
kubectl uncordon node01
kubectl get nodes
```

---

## Package Repository Note

Kubernetes legacy package repositories are deprecated. Use the new package repository:

```text
pkgs.k8s.io
```

Example for Kubernetes `v1.29` on Ubuntu/Debian:

```bash
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

Add the signing key:

```bash
curl -fsSL https://pkgs.k8s.io/core/stable/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

Update package index:

```bash
sudo apt-get update
```

---

## Backup and Restore

### Declarative Backup

Keep Kubernetes YAML manifests in Git.

Example:

```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

Restore everything from a folder:

```bash
kubectl apply -f .
```

---

### API Resource Backup

Export common Kubernetes resources:

```bash
kubectl get all --all-namespaces -o yaml > all-deploy-services.yaml
```

Short form:

```bash
kubectl get all -A -o yaml > all-deploy-services.yaml
```

Important: `kubectl get all` does not include every resource type. Secrets, ConfigMaps, PVCs, PVs, RBAC, CRDs, and NetworkPolicies may need separate backups.

---

### Backup Secrets and ConfigMaps

```bash
kubectl get secrets -A -o yaml > secrets-backup.yaml
kubectl get configmaps -A -o yaml > configmaps-backup.yaml
```

---

### Backup Persistent Volumes and Claims

```bash
kubectl get pv -o yaml > pv-backup.yaml
kubectl get pvc -A -o yaml > pvc-backup.yaml
```

---

## etcd Backup

### Save etcd Snapshot

```bash
ETCDCTL_API=3 etcdctl snapshot save snapshot.db
```

With certificates:

```bash
ETCDCTL_API=3 etcdctl snapshot save snapshot.db   --endpoints=https://127.0.0.1:2379   --cacert=/etc/kubernetes/pki/etcd/ca.crt   --cert=/etc/kubernetes/pki/etcd/server.crt   --key=/etc/kubernetes/pki/etcd/server.key
```

---

### Check Snapshot Status

```bash
ETCDCTL_API=3 etcdctl snapshot status snapshot.db -w table
```

---

### Restore etcd Snapshot

```bash
ETCDCTL_API=3 etcdctl snapshot restore snapshot.db   --data-dir /var/lib/etcd-from-backup
```

After restore, update the etcd data directory in the etcd manifest.

Common kubeadm static pod manifest path:

```bash
/etc/kubernetes/manifests/etcd.yaml
```

Update:

```bash
--data-dir=/var/lib/etcd-from-backup
```

---

## Important File Paths

| Item | Path |
|---|---|
| kubeadm etcd manifest | `/etc/kubernetes/manifests/etcd.yaml` |
| etcd CA certificate | `/etc/kubernetes/pki/etcd/ca.crt` |
| etcd server certificate | `/etc/kubernetes/pki/etcd/server.crt` |
| etcd server key | `/etc/kubernetes/pki/etcd/server.key` |
| Default etcd data directory | `/var/lib/etcd` |
| Restored etcd data directory example | `/var/lib/etcd-from-backup` |

---

## Maintenance Command Summary

| Task | Command |
|---|---|
| Get nodes | `kubectl get nodes` |
| Check Pod placement | `kubectl get pods -A -o wide` |
| Cordon node | `kubectl cordon <node>` |
| Drain node | `kubectl drain <node> --ignore-daemonsets --delete-emptydir-data` |
| Uncordon node | `kubectl uncordon <node>` |
| Upgrade plan | `sudo kubeadm upgrade plan` |
| Apply control plane upgrade | `sudo kubeadm upgrade apply v<version>` |
| Upgrade worker node config | `sudo kubeadm upgrade node` |
| Restart kubelet | `sudo systemctl restart kubelet` |
| Backup API resources | `kubectl get all -A -o yaml > all-deploy-services.yaml` |
| Save etcd snapshot | `ETCDCTL_API=3 etcdctl snapshot save snapshot.db` |
| Restore etcd snapshot | `ETCDCTL_API=3 etcdctl snapshot restore snapshot.db --data-dir <path>` |

---

## Key Concepts

### `kubectl get nodes` Version

The `VERSION` column from:

```bash
kubectl get nodes
```

shows the **kubelet version**, not necessarily the API server version.

---

### kubeadm Does Not Upgrade kubelet Automatically

After upgrading the control plane with `kubeadm upgrade apply`, you still need to manually upgrade and restart `kubelet`.

---

### Worker Nodes Should Be Upgraded One by One

Recommended flow:

```text
drain → upgrade → restart kubelet → uncordon
```

---

### Managed Kubernetes Backup

For managed Kubernetes services like EKS, GKE, or AKS, you may not have direct etcd access. In that case, use:

- Cloud provider backup options
- Velero
- API resource export
- Git-based manifests
- Persistent volume backup

---

## CKA Exam Focus

Focus especially on:

1. `kubectl drain`
2. `kubectl cordon`
3. `kubectl uncordon`
4. `kubeadm upgrade plan`
5. `kubeadm upgrade apply`
6. `kubeadm upgrade node`
7. Manual `kubelet` upgrade
8. etcd snapshot backup
9. etcd snapshot restore
10. `kubectl get all -A -o yaml`

---

## Folder Structure

```text
Cluster-Maintenance/
├── kubernetes_backup_restore_methods_note.md
├── kubernetes_cluster_upgrade_introduction_note.md
├── kubernetes_demo_cluster_upgrade_note.md
└── kubernetes_os_upgrades_note.md
```

---

## References

- Kubernetes Documentation
- KodeKloud CKA Course Notes
- Kubernetes kubeadm Upgrade Documentation
- Kubernetes etcd Backup and Restore Concepts
