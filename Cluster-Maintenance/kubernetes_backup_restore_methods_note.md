# Backup and Restore Methods

> Topic: Kubernetes cluster backup and restore strategies, declarative files, API resource backup, and etcd snapshot/restore

---

## 1. Overview

Kubernetes cluster မှာ backup and restore က အရေးကြီးတဲ့ operation ဖြစ်ပါတယ်။

Cluster failure ဖြစ်တာ၊ resource မတော်တဆ delete ဖြစ်တာ၊ etcd corruption ဖြစ်တာ၊ migration လုပ်တာတွေမှာ backup ရှိမှ cluster state ကိုပြန် restore လုပ်နိုင်ပါတယ်။

Backup strategy တွေမှာ အဓိက ၃ မျိုးရှိပါတယ်။

1. Declarative configuration files backup
2. Kubernetes API resources backup
3. etcd snapshot backup

---

## 2. What Should Be Backed Up?

Kubernetes environment မှာ backup လုပ်သင့်တာတွေက —

- YAML definition files
- Deployments
- Pods
- Services
- ConfigMaps
- Secrets
- Namespaces
- PersistentVolumeClaims
- RBAC resources
- etcd cluster state
- Imperatively created resources

---

## 3. Declarative Configuration Backup

Declarative approach ဆိုတာ Kubernetes resources တွေကို YAML file နဲ့ define လုပ်ပြီး `kubectl apply` နဲ့ create/update လုပ်တာပါ။

ဥပမာ Pod definition:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
    type: front-end
spec:
  containers:
    - name: nginx-container
      image: nginx
```

Apply:

```bash
kubectl apply -f pod-definition.yml
```

ဒီလို YAML files တွေကို GitHub / GitLab repository ထဲ version control နဲ့သိမ်းထားရင် restore လုပ်ရတာလွယ်ပါတယ်။

---

## 4. Why Store YAML Files in Git?

YAML definition files တွေကို Git repository ထဲသိမ်းထားခြင်းရဲ့ အကျိုးကျေးဇူးတွေ —

- Configuration history သိနိုင်တယ်
- Rollback လုပ်ရလွယ်တယ်
- Team members တွေ share လုပ်နိုင်တယ်
- Disaster recovery အတွက် redeploy လုပ်နိုင်တယ်
- GitOps workflow နဲ့ကိုက်တယ်

Example restore:

```bash
kubectl apply -f .
```

or

```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

---

## 5. Problem with Imperative Objects

တချို့ resources တွေကို imperative commands နဲ့ create လုပ်ထားနိုင်ပါတယ်။

ဥပမာ —

```bash
kubectl create namespace dev
kubectl create secret generic app-secret --from-literal=password=12345
kubectl create configmap app-config --from-literal=APP_COLOR=blue
```

ဒီ resources တွေက YAML file ထဲမရှိရင် Git repo ထဲမှာ backup မရှိပါဘူး။  
ဒါကြောင့် API server ကနေ current cluster resources တွေကို export လုပ်သင့်ပါတယ်။

---

## 6. API Server Resource Backup

Cluster ထဲက resources တွေကို API server ကနေ YAML အဖြစ် export လုပ်နိုင်ပါတယ်။

```bash
kubectl get all --all-namespaces -o yaml > all-deploy-services.yaml
```

ဒီ command က namespaces အားလုံးထဲက common workload resources တွေကို YAML file အဖြစ်သိမ်းပါတယ်။

သို့သော် `kubectl get all` က **resources အားလုံးကို 100% မပါစေပါဘူး**။

ဥပမာ ဒီ resource types တွေ မပါနိုင်ပါဘူး —

- Secrets
- ConfigMaps
- PersistentVolumeClaims
- PersistentVolumes
- RBAC
- CustomResourceDefinitions
- NetworkPolicies

ဒါကြောင့် production backup အတွက် Velero လို tool သုံးတာပိုကောင်းပါတယ်။

---

## 7. More Complete API Backup Examples

### Backup all common resources

```bash
kubectl get all --all-namespaces -o yaml > all-resources.yaml
```

### Backup Secrets

```bash
kubectl get secrets --all-namespaces -o yaml > secrets-backup.yaml
```

### Backup ConfigMaps

```bash
kubectl get configmaps --all-namespaces -o yaml > configmaps-backup.yaml
```

### Backup PVCs

```bash
kubectl get pvc --all-namespaces -o yaml > pvc-backup.yaml
```

### Backup PVs

```bash
kubectl get pv -o yaml > pv-backup.yaml
```

### Backup RBAC

```bash
kubectl get roles,rolebindings,clusterroles,clusterrolebindings --all-namespaces -o yaml > rbac-backup.yaml
```

---

## 8. Velero

**Velero** က Kubernetes backup and restore အတွက် popular tool ဖြစ်ပါတယ်။

Velero က backup လုပ်နိုင်တာတွေ —

- Kubernetes resources
- Namespaces
- Persistent volumes
- Cloud object storage
- Scheduled backups
- Disaster recovery
- Cluster migration

Production environment မှာ manual `kubectl get` backup ထက် Velero လို tool သုံးတာပိုသင့်တော်ပါတယ်။

---

## 9. etcd ဆိုတာဘာလဲ?

`etcd` က Kubernetes cluster ရဲ့ key-value store ဖြစ်ပါတယ်။

Cluster state အရေးကြီး data အားလုံးကို etcd ထဲမှာသိမ်းပါတယ်။

ဥပမာ —

- Nodes
- Pods
- Deployments
- Services
- ConfigMaps
- Secrets
- RBAC
- Cluster configuration

ဒါကြောင့် etcd backup က Kubernetes disaster recovery အတွက် အရေးကြီးဆုံး backup တစ်ခုဖြစ်ပါတယ်။

---

## 10. etcd Data Directory

etcd data directory ကို etcd configuration ထဲမှာ `--data-dir` flag နဲ့သတ်မှတ်ထားပါတယ်။

Example:

```bash
--data-dir=/var/lib/etcd
```

ဒီ directory ထဲမှာ etcd data files တွေရှိပါတယ်။

---

## 11. etcd Configuration Example

Example etcd service configuration:

```bash
ExecStart=/usr/local/bin/etcd \
   --name ${ETCD_NAME} \
   --cert-file=/etc/etcd/kubernetes.pem \
   --key-file=/etc/etcd/kubernetes-key.pem \
   --peer-cert-file=/etc/etcd/kubernetes.pem \
   --peer-key-file=/etc/etcd/kubernetes-key.pem \
   --trusted-ca-file=/etc/etcd/ca.pem \
   --peer-trusted-ca-file=/etc/etcd/ca.pem \
   --client-cert-auth \
   --initial-advertise-peer-urls https://${INTERNAL_IP}:2380 \
   --listen-peer-urls https://${INTERNAL_IP}:2380 \
   --advertise-client-urls https://${INTERNAL_IP}:2379 \
   --data-dir=/var/lib/etcd
```

Important flags:

| Flag                      | Meaning                    |
| ------------------------- | -------------------------- |
| `--data-dir`              | etcd data location         |
| `--cert-file`             | etcd server certificate    |
| `--key-file`              | etcd server key            |
| `--trusted-ca-file`       | CA certificate             |
| `--listen-client-urls`    | client access endpoint     |
| `--advertise-client-urls` | advertised client endpoint |

---

## 12. etcd Snapshot Backup

etcd မှာ built-in snapshot feature ရှိပါတယ်။

Snapshot save command:

```bash
ETCDCTL_API=3 etcdctl snapshot save snapshot.db
```

ဒီ command က current etcd state ကို `snapshot.db` file အဖြစ်သိမ်းပေးပါတယ်။

Check file:

```bash
ls
```

Snapshot status ကြည့်ရန်:

```bash
ETCDCTL_API=3 etcdctl snapshot status snapshot.db
```

---

## 13. Authenticated etcd Snapshot Backup

Production cluster မှာ etcd ကို certificate authentication နဲ့ access လုပ်ရတာများပါတယ်။

Authenticated backup command:

```bash
ETCDCTL_API=3 etcdctl snapshot save snapshot.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.crt \
  --cert=/etc/etcd/etcd-server.crt \
  --key=/etc/etcd/etcd-server.key
```

kubeadm cluster မှာ cert paths က များသောအားဖြင့် ဒီလိုဖြစ်နိုင်ပါတယ်။

```bash
--cacert=/etc/kubernetes/pki/etcd/ca.crt
--cert=/etc/kubernetes/pki/etcd/server.crt
--key=/etc/kubernetes/pki/etcd/server.key
```

---

## 14. etcd Snapshot Status

Snapshot status ကြည့်ရန်:

```bash
ETCDCTL_API=3 etcdctl snapshot status snapshot.db
```

Table format နဲ့ကြည့်ချင်ရင်:

```bash
ETCDCTL_API=3 etcdctl snapshot status snapshot.db -w table
```

Output example:

```text
+----------+----------+------------+------------+
|   HASH   | REVISION | TOTAL KEYS | TOTAL SIZE |
+----------+----------+------------+------------+
```

---

## 15. Restoring from etcd Backup

etcd snapshot restore process မှာ အဆင့်များစွာပါပါတယ်။

General steps:

1. Stop Kubernetes API server
2. Restore snapshot to new data directory
3. Update etcd configuration to use new data directory
4. Reload system daemon
5. Restart etcd
6. Restart kube-apiserver
7. Verify cluster state

---

## 16. Restore Snapshot

Snapshot ကို new data directory ထဲ restore လုပ်ပါ။

```bash
ETCDCTL_API=3 etcdctl snapshot restore snapshot.db \
  --data-dir /var/lib/etcd-from-backup
```

ဒီ command က `/var/lib/etcd-from-backup` directory ထဲမှာ restored etcd data ကို create လုပ်ပါတယ်။

---

## 17. Update etcd Data Directory

Restore ပြီးရင် etcd configuration ထဲက data directory ကို new path သို့ပြောင်းရပါတယ်။

Old:

```bash
--data-dir=/var/lib/etcd
```

New:

```bash
--data-dir=/var/lib/etcd-from-backup
```

kubeadm cluster မှာ etcd static pod manifest ကိုပြင်ရနိုင်ပါတယ်။

Common path:

```bash
/etc/kubernetes/manifests/etcd.yaml
```

---

## 18. Restart Services

Systemd-managed etcd ဆိုရင်:

```bash
systemctl daemon-reload
systemctl restart etcd
systemctl restart kube-apiserver
```

kubeadm static pod setup ဆိုရင် manifest file ပြင်ပြီး save လုပ်တာနဲ့ kubelet က static pod ကို auto restart လုပ်နိုင်ပါတယ်။

Check:

```bash
kubectl get pods -n kube-system
```

---

## 19. Important Certificate Files

etcd snapshot save/restore commands မှာ cert files တွေလိုနိုင်ပါတယ်။

Common kubeadm paths:

| File               | Path                                  |
| ------------------ | ------------------------------------- |
| CA certificate     | `/etc/kubernetes/pki/etcd/ca.crt`     |
| Server certificate | `/etc/kubernetes/pki/etcd/server.crt` |
| Server key         | `/etc/kubernetes/pki/etcd/server.key` |

Example:

```bash
ETCDCTL_API=3 etcdctl snapshot save snapshot.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key
```

---

## 20. Backup Approaches Comparison

| Backup Approach         | Use Case                               | Example                             |
| ----------------------- | -------------------------------------- | ----------------------------------- |
| Declarative file backup | YAML files are maintained as code      | Git repository                      |
| API server backup       | Capture resources created imperatively | `kubectl get all -A -o yaml`        |
| etcd snapshot           | Backup full cluster state              | `etcdctl snapshot save snapshot.db` |
| Velero                  | Production-grade Kubernetes backup     | Scheduled cluster backup            |

---

## 21. Managed Kubernetes Backup Strategy

Managed Kubernetes services မှာ direct etcd access မရနိုင်တာများပါတယ်။

Examples:

- GKE
- EKS
- AKS

ဒီလို environment တွေမှာ etcd snapshot ကို direct မလုပ်နိုင်နိုင်ပါဘူး။

Backup strategy:

- Use cloud provider backup feature
- Use Velero
- Export Kubernetes resources using API
- Keep YAML manifests in Git
- Backup persistent volumes separately

---

## 22. Restore from YAML Files

YAML manifests backup ရှိရင် restore လုပ်ရန်:

```bash
kubectl apply -f pod-definition.yml
```

Folder တစ်ခုလုံး apply:

```bash
kubectl apply -f .
```

Multiple resources restore:

```bash
kubectl apply -f all-deploy-services.yaml
```

---

## 23. Restore from API Backup YAML

API backup file ရှိရင်:

```bash
kubectl apply -f all-deploy-services.yaml
```

သတိထားရန် —

- Some generated fields may cause issues
- `status` fields may not be needed
- Secrets are only Base64 encoded
- Cluster-specific fields may need cleanup

---

## 24. CKA Exam Tips

### Tip 1 — etcd snapshot command မှတ်ထားပါ

```bash
ETCDCTL_API=3 etcdctl snapshot save snapshot.db
```

With certs:

```bash
ETCDCTL_API=3 etcdctl snapshot save snapshot.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key
```

---

### Tip 2 — Snapshot restore command မှတ်ထားပါ

```bash
ETCDCTL_API=3 etcdctl snapshot restore snapshot.db \
  --data-dir /var/lib/etcd-from-backup
```

---

### Tip 3 — etcd static pod manifest path

```bash
/etc/kubernetes/manifests/etcd.yaml
```

---

### Tip 4 — API backup command

```bash
kubectl get all --all-namespaces -o yaml > all-deploy-services.yaml
```

Short form:

```bash
kubectl get all -A -o yaml > all-deploy-services.yaml
```

---

### Tip 5 — Declarative config in Git is best practice

YAML files ကို Git ထဲသိမ်းထားတာ restore/redeploy အတွက် အလွန်အသုံးဝင်ပါတယ်။

---

## 25. Common Mistakes

### Mistake 1 — `kubectl get all` က everything ပါတယ်လို့ထင်ခြင်း

`kubectl get all` က Secrets, ConfigMaps, PVCs, RBAC resources တွေကို မပါစေနိုင်ပါဘူး။

---

### Mistake 2 — etcd cert flags မထည့်ခြင်း

etcd authentication enabled ဖြစ်ရင် `--cacert`, `--cert`, `--key` flags မထည့်ရင် snapshot command fail ဖြစ်နိုင်ပါတယ်။

---

### Mistake 3 — Restore ပြီး data-dir မပြောင်းခြင်း

Snapshot restore လုပ်ပြီး etcd configuration ထဲမှာ new `--data-dir` ကို point မလုပ်ရင် restored data ကိုအသုံးမပြုနိုင်ပါဘူး။

---

### Mistake 4 — Managed cluster မှာ etcd access ရှိမယ်လို့ထင်ခြင်း

EKS/GKE/AKS လို managed Kubernetes မှာ etcd ကို direct access မရနိုင်ပါဘူး။

---

## 26. Practice Example

### Task 1

Cluster resources backup ကို YAML file အဖြစ်သိမ်းပါ။

Solution:

```bash
kubectl get all --all-namespaces -o yaml > all-deploy-services.yaml
```

---

### Task 2

etcd snapshot backup ကို `snapshot.db` အဖြစ် save လုပ်ပါ။

Solution:

```bash
ETCDCTL_API=3 etcdctl snapshot save snapshot.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key
```

Verify:

```bash
ETCDCTL_API=3 etcdctl snapshot status snapshot.db -w table
```

---

### Task 3

`snapshot.db` ကို `/var/lib/etcd-from-backup` ထဲ restore လုပ်ပါ။

Solution:

```bash
ETCDCTL_API=3 etcdctl snapshot restore snapshot.db \
  --data-dir /var/lib/etcd-from-backup
```

Then update etcd manifest:

```bash
vi /etc/kubernetes/manifests/etcd.yaml
```

Change:

```bash
--data-dir=/var/lib/etcd-from-backup
```

---

## 27. Quick Reference

```bash
# Backup all common resources
kubectl get all -A -o yaml > all-deploy-services.yaml

# Backup etcd snapshot
ETCDCTL_API=3 etcdctl snapshot save snapshot.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

# Check snapshot status
ETCDCTL_API=3 etcdctl snapshot status snapshot.db -w table

# Restore snapshot
ETCDCTL_API=3 etcdctl snapshot restore snapshot.db \
  --data-dir /var/lib/etcd-from-backup

# Edit etcd static pod manifest
vi /etc/kubernetes/manifests/etcd.yaml
```

---

## 28. Key Takeaways

- Kubernetes backup strategy မှာ YAML manifests, API resources, and etcd snapshot backup ပါဝင်သင့်ပါတယ်။
- Declarative YAML files တွေကို Git ထဲသိမ်းထားတာ best practice ဖြစ်ပါတယ်။
- Imperative resources တွေကို API server ကနေ export လုပ်ပြီး backup လုပ်နိုင်ပါတယ်။
- `kubectl get all -A -o yaml` က common resources ကို backup လုပ်ပေးပေမယ့် everything မပါပါ။
- etcd က Kubernetes cluster state ရဲ့ main data store ဖြစ်ပါတယ်။
- etcd snapshot ကို `etcdctl snapshot save` နဲ့ backup လုပ်နိုင်ပါတယ်။
- etcd restore ကို `etcdctl snapshot restore` နဲ့ new data directory ထဲ restore လုပ်ရပါတယ်။
- Restore ပြီးရင် etcd configuration ကို new data directory သို့ point လုပ်ရပါတယ်။
- Managed Kubernetes မှာ direct etcd access မရနိုင်တာကြောင့် Velero/API backup/cloud provider tools သုံးရနိုင်ပါတယ်။
