# Demo Cluster Upgrade

> Topic: kubeadm ကိုသုံးပြီး Kubernetes cluster ကို `v1.28` မှ `v1.29` သို့ upgrade လုပ်ခြင်း

---

## 1. Overview

ဒီ lesson မှာ Kubernetes cluster ကို **kubeadm** အသုံးပြုပြီး upgrade လုပ်တဲ့ demo flow ကိုလေ့လာပါမယ်။

Example upgrade path:

```text
v1.28 → v1.29
```

Cluster upgrade မှာ အဓိကအားဖြင့် —

1. Package repository update လုပ်မယ်
2. Control plane node ပေါ်မှာ `kubeadm` upgrade လုပ်မယ်
3. `kubeadm upgrade plan` run မယ်
4. Control plane ကို upgrade apply လုပ်မယ်
5. Control plane node ပေါ်က `kubelet` and `kubectl` upgrade လုပ်မယ်
6. Worker nodes တွေကို one by one upgrade လုပ်မယ်
7. Drain / uncordon နဲ့ downtime လျှော့မယ်

---

## 2. Official Upgrade Documentation

Kubernetes upgrade ကို official documentation ထဲက **Upgrading kubeadm clusters** section အတိုင်းလုပ်သင့်ပါတယ်။

Upgrade path တစ်ခုချင်းစီအတွက် instruction မတူနိုင်ပါတယ်။

ဥပမာ —

```text
v1.27 → v1.28
v1.28 → v1.29
v1.29 → v1.30
```

အမြဲ correct upgrade path ကိုရွေးပြီး documentation အတိုင်းလုပ်ပါ။

---

## 3. Check Current Cluster Version

Cluster nodes တွေကိုကြည့်ရန်:

```bash
kubectl get nodes
```

Example output:

```text
NAME           STATUS   ROLES           AGE   VERSION
controlplane   Ready    control-plane   98m   v1.28.0
node01         Ready    <none>          98m   v1.28.0
```

ဒီ example မှာ —

- Control plane node = `v1.28.0`
- Worker node = `v1.28.0`

Target version = `v1.29.3`

---

## 4. Check Node OS Distribution

Package repository command တွေက OS distribution ပေါ်မူတည်ပြီးကွာနိုင်ပါတယ်။

OS check:

```bash
cat /etc/*release*
```

Ubuntu example:

```text
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=20.04
DISTRIB_CODENAME=focal
NAME="Ubuntu"
VERSION="20.04.6 LTS (Focal Fossa)"
ID=ubuntu
ID_LIKE=debian
```

ဒီ note မှာ Debian/Ubuntu-based system အတွက် command တွေကို focus လုပ်ထားပါတယ်။

---

## 5. Package Repository Update

Kubernetes legacy package repositories တွေဖြစ်တဲ့ —

```text
apt.kubernetes.io
yum.kubernetes.io
```

တွေက deprecated ဖြစ်ထားပါတယ်။

New repository:

```text
packages.k8s.io
```

Kubernetes packages တွေဖြစ်တဲ့ —

- kubeadm
- kubelet
- kubectl

တွေကို new repository ကနေ install/upgrade လုပ်ရပါတယ်။

---

## 6. Configure Kubernetes Package Repository for v1.29

Ubuntu/Debian system မှာ v1.29 repository ထည့်ရန်:

```bash
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

Public signing key download လုပ်ရန်:

```bash
curl -fsSL https://pkgs.k8s.io/core/stable/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

> Version number ကို upgrade target နဲ့ကိုက်အောင် ပြင်ပါ။ ဥပမာ `v1.29`.

Update apt cache:

```bash
sudo apt-get update
```

ဒီ repository update ကို control plane node နဲ့ worker nodes တွေမှာလိုအပ်နိုင်ပါတယ်။

---

## 7. Upgrade Flow Overview

Cluster upgrade flow:

```text
1. Update package repository
2. Upgrade kubeadm on control plane
3. Run kubeadm upgrade plan
4. Apply control plane upgrade
5. Drain control plane node
6. Upgrade kubelet and kubectl on control plane
7. Restart kubelet
8. Uncordon control plane node
9. Upgrade worker nodes one by one
10. Verify all nodes version
```

---

# Part 1 — Upgrading the Control Plane

---

## 8. Step 1: Upgrade kubeadm on Control Plane

Available kubeadm versions ကြည့်ရန်:

```bash
sudo apt update
sudo apt-cache madison kubeadm
```

Example output:

```text
kubeadm | 1.29.3-1.1 | https://pkgs.k8s.io/core/stable/v1.29/deb Packages
kubeadm | 1.29.2-1.1 | https://pkgs.k8s.io/core/stable/v1.29/deb Packages
kubeadm | 1.29.1-1.1 | https://pkgs.k8s.io/core/stable/v1.29/deb Packages
kubeadm | 1.29.0-1.1 | https://pkgs.k8s.io/core/stable/v1.29/deb Packages
```

Latest patch version ကိုရွေးပါ။  
ဒီ example မှာ `1.29.3-1.1` ကိုသုံးပါတယ်။

Upgrade kubeadm:

```bash
sudo apt-mark unhold kubeadm && sudo apt-get update && sudo apt-get install -y kubeadm='1.29.3-1.1' && sudo apt-mark hold kubeadm
```

Verify:

```bash
kubeadm version
```

Expected output:

```text
kubeadm version: &version.Info{Major:"1", Minor:"29", GitVersion:"v1.29.3", ...}
```

---

## 9. Why apt-mark unhold / hold?

Kubernetes packages တွေကို accidental upgrade မဖြစ်စေဖို့ usually `hold` လုပ်ထားပါတယ်။

Upgrade လုပ်ချင်တဲ့အခါ —

```bash
sudo apt-mark unhold kubeadm
```

Upgrade ပြီးရင် ပြန် hold လုပ်ပါတယ်။

```bash
sudo apt-mark hold kubeadm
```

ဒီနည်းက version control ပိုကောင်းစေပါတယ်။

---

## 10. Step 2: Run kubeadm Upgrade Plan

Control plane upgrade apply မလုပ်ခင် dry-run / plan ကြည့်ပါ။

```bash
sudo kubeadm upgrade plan
```

ဒီ command ကပြပေးတာတွေ —

- Current cluster version
- kubeadm version
- Target Kubernetes version
- Upgradeable control plane components
- Manually upgrade လုပ်ရမယ့် components
- Recommended upgrade command

Example:

```text
[upgrade/versions] Cluster version: v1.28.0
[upgrade/versions] kubeadm version: v1.29.3
[upgrade/versions] Target version: v1.29.3
```

Important:

```text
kubeadm upgrades control plane components,
but kubelet must be upgraded manually.
```

---

## 11. Step 3: Apply Control Plane Upgrade

Control plane upgrade apply:

```bash
sudo kubeadm upgrade apply v1.29.3
```

ဒီ command က —

- Certificates renew လုပ်နိုင်တယ်
- Static Pod manifests update လုပ်တယ်
- Control plane components upgrade လုပ်တယ်
- Configuration changes apply လုပ်တယ်

Success output example:

```text
[upgrade/successful] SUCCESS! Your cluster was upgraded to "v1.29.3". Enjoy!
[upgrade/kubelet] Now that your control plane is upgraded, please proceed with upgrading your kubelets.
```

---

## 12. Why kubectl get nodes Still Shows Old Version?

Control plane upgrade ပြီးပြီးချင်း:

```bash
kubectl get nodes
```

run လိုက်ရင် node version က still `v1.28.0` လို့ပြနိုင်ပါတယ်။

အကြောင်းရင်း:

```text
kubectl get nodes shows kubelet version, not kube-apiserver version.
```

ဒါကြောင့် control plane upgrade ပြီးရင် kubelet ကိုလည်း manual upgrade လုပ်ရပါတယ်။

---

# Part 2 — Upgrading kubelet and kubectl on Control Plane

---

## 13. Step 1: Drain Control Plane Node

Control plane node ပေါ်မှာ kubelet/kubectl upgrade မလုပ်ခင် drain လုပ်ပါ။

```bash
kubectl drain controlplane --ignore-daemonsets
```

Drain လုပ်တာက —

- Node ကို unschedulable လုပ်တယ်
- Existing Pods တွေကို evict လုပ်တယ်
- DaemonSet Pods တွေကို ignore လုပ်တယ်

---

## 14. Step 2: Upgrade kubelet and kubectl

Control plane node ပေါ်မှာ kubelet နဲ့ kubectl ကို target version သို့ upgrade လုပ်ပါ။

```bash
sudo apt-mark unhold kubelet kubectl && sudo apt-get update && sudo apt-get install -y kubelet='1.29.3-1.1' kubectl='1.29.3-1.1' && sudo apt-mark hold kubelet kubectl
```

---

## 15. Step 3: Reload and Restart kubelet

Systemd reload:

```bash
sudo systemctl daemon-reload
```

Restart kubelet:

```bash
sudo systemctl restart kubelet
```

---

## 16. Step 4: Uncordon Control Plane Node

Upgrade ပြီးသွားရင် scheduling ပြန်ခွင့်ပြုပါ။

```bash
kubectl uncordon controlplane
```

Verify:

```bash
kubectl get nodes
```

Expected:

```text
controlplane   Ready   control-plane   ...   v1.29.3
```

---

# Part 3 — Upgrading Worker Nodes

---

## 17. Worker Node Upgrade Overview

Worker node upgrade ကို one by one လုပ်သင့်ပါတယ်။

Worker node တစ်လုံးချင်းစီအတွက် flow:

```text
1. Upgrade kubeadm
2. Run kubeadm upgrade node
3. Drain worker node
4. Upgrade kubelet and kubectl
5. Restart kubelet
6. Uncordon worker node
7. Verify node version
```

---

## 18. Step 1: Upgrade kubeadm on Worker Node

Worker node ပေါ်မှာ kubeadm upgrade လုပ်ပါ။

```bash
sudo apt-mark unhold kubeadm && sudo apt-get update && sudo apt-get install -y kubeadm='1.29.3-1.1' && sudo apt-mark hold kubeadm
```

Verify:

```bash
kubeadm version
```

---

## 19. Step 2: Upgrade Worker Node Configuration

Worker node ပေါ်မှာ node configuration update လုပ်ပါ။

```bash
sudo kubeadm upgrade node
```

ဒီ command က kubelet package ကို upgrade မလုပ်ပါဘူး။  
Node configuration ကို upgrade လုပ်တာပါ။

---

## 20. Step 3: Drain Worker Node

Control plane node ကနေ worker node ကို drain လုပ်ပါ။

```bash
kubectl drain node01 --ignore-daemonsets
```

CKA lab တွေမှာ emptyDir data warning ရှိရင် ဒီ command လိုနိုင်ပါတယ်။

```bash
kubectl drain node01 --ignore-daemonsets --delete-emptydir-data
```

---

## 21. Step 4: Upgrade kubelet and kubectl on Worker Node

Worker node ပေါ်မှာ kubelet နဲ့ kubectl ကို upgrade လုပ်ပါ။

```bash
sudo apt-mark unhold kubelet kubectl && sudo apt-get update && sudo apt-get install -y kubelet='1.29.3-1.1' kubectl='1.29.3-1.1' && sudo apt-mark hold kubelet kubectl
```

---

## 22. Step 5: Restart kubelet

```bash
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

---

## 23. Step 6: Uncordon Worker Node

Upgrade ပြီးသွားတဲ့ worker node ကို scheduling ပြန်ဖွင့်ပါ။

```bash
kubectl uncordon node01
```

---

## 24. Step 7: Verify Worker Node Version

```bash
kubectl get nodes
```

Expected:

```text
NAME           STATUS   ROLES           AGE   VERSION
controlplane   Ready    control-plane   ...   v1.29.3
node01         Ready    <none>          ...   v1.29.3
```

Worker nodes အားလုံးကို ဒီ flow အတိုင်း one by one upgrade လုပ်ပါ။

---

## 25. Control Plane Commands Summary

```bash
# Update repo first
sudo apt-get update

# Check available kubeadm versions
sudo apt-cache madison kubeadm

# Upgrade kubeadm
sudo apt-mark unhold kubeadm && sudo apt-get update && sudo apt-get install -y kubeadm='1.29.3-1.1' && sudo apt-mark hold kubeadm

# Verify kubeadm
kubeadm version

# Check upgrade plan
sudo kubeadm upgrade plan

# Apply upgrade
sudo kubeadm upgrade apply v1.29.3

# Drain control plane
kubectl drain controlplane --ignore-daemonsets

# Upgrade kubelet and kubectl
sudo apt-mark unhold kubelet kubectl && sudo apt-get update && sudo apt-get install -y kubelet='1.29.3-1.1' kubectl='1.29.3-1.1' && sudo apt-mark hold kubelet kubectl

# Restart kubelet
sudo systemctl daemon-reload
sudo systemctl restart kubelet

# Uncordon
kubectl uncordon controlplane

# Verify
kubectl get nodes
```

---

## 26. Worker Node Commands Summary

```bash
# On worker node: upgrade kubeadm
sudo apt-mark unhold kubeadm && sudo apt-get update && sudo apt-get install -y kubeadm='1.29.3-1.1' && sudo apt-mark hold kubeadm

# On worker node: upgrade node config
sudo kubeadm upgrade node

# On control plane: drain worker node
kubectl drain node01 --ignore-daemonsets

# On worker node: upgrade kubelet and kubectl
sudo apt-mark unhold kubelet kubectl && sudo apt-get update && sudo apt-get install -y kubelet='1.29.3-1.1' kubectl='1.29.3-1.1' && sudo apt-mark hold kubelet kubectl

# On worker node: restart kubelet
sudo systemctl daemon-reload
sudo systemctl restart kubelet

# On control plane: uncordon worker node
kubectl uncordon node01

# Verify
kubectl get nodes
```

---

## 27. Important Commands Table

| Task                        | Command                                                             |
| --------------------------- | ------------------------------------------------------------------- |
| Check nodes                 | `kubectl get nodes`                                                 |
| Check OS release            | `cat /etc/*release*`                                                |
| Update apt cache            | `sudo apt-get update`                                               |
| List kubeadm versions       | `sudo apt-cache madison kubeadm`                                    |
| Upgrade kubeadm             | `sudo apt-get install -y kubeadm='1.29.3-1.1'`                      |
| Hold kubeadm                | `sudo apt-mark hold kubeadm`                                        |
| Check kubeadm version       | `kubeadm version`                                                   |
| Check upgrade plan          | `sudo kubeadm upgrade plan`                                         |
| Apply control plane upgrade | `sudo kubeadm upgrade apply v1.29.3`                                |
| Drain node                  | `kubectl drain <node> --ignore-daemonsets`                          |
| Upgrade node config         | `sudo kubeadm upgrade node`                                         |
| Upgrade kubelet/kubectl     | `sudo apt-get install -y kubelet='1.29.3-1.1' kubectl='1.29.3-1.1'` |
| Restart kubelet             | `sudo systemctl restart kubelet`                                    |
| Uncordon node               | `kubectl uncordon <node>`                                           |

---

## 28. CKA Exam Tips

### Tip 1 — Correct order မှတ်ပါ

```text
Control plane first → worker nodes later
```

---

### Tip 2 — kubeadm before cluster upgrade

```bash
sudo apt-get install -y kubeadm='<target-version>'
sudo kubeadm upgrade plan
sudo kubeadm upgrade apply v<target-version>
```

---

### Tip 3 — kubelet is manual

```text
kubeadm does not upgrade kubelet automatically.
```

Control plane upgrade ပြီးနောက် kubelet/kubectl ကို manually upgrade လုပ်ပါ။

---

### Tip 4 — kubectl get nodes shows kubelet version

`kubectl get nodes` ထဲက VERSION column က kubelet version ဖြစ်ပါတယ်။

---

### Tip 5 — Drain and uncordon

Before node maintenance/upgrade:

```bash
kubectl drain <node> --ignore-daemonsets
```

After upgrade:

```bash
kubectl uncordon <node>
```

---

### Tip 6 — New package repo

Legacy repository မသုံးပါနဲ့။ New repository:

```text
pkgs.k8s.io
```

---

## 29. Common Mistakes

### Mistake 1 — kubelet upgrade မလုပ်မိခြင်း

Control plane upgrade ပြီးလည်း `kubectl get nodes` မှာ old version ပြနေရင် kubelet မ upgrade ရသေးတာဖြစ်နိုင်ပါတယ်။

---

### Mistake 2 — Worker node drain မလုပ်မိခြင်း

Worker node upgrade/restart လုပ်မယ်ဆိုရင် drain လုပ်ပါ။

```bash
kubectl drain node01 --ignore-daemonsets
```

---

### Mistake 3 — Uncordon မလုပ်မိခြင်း

Upgrade ပြီးနောက် node က SchedulingDisabled ဖြစ်နေပါမယ်။

Fix:

```bash
kubectl uncordon node01
```

---

### Mistake 4 — Wrong repository version

Target version `v1.29` ဖြစ်ရင် repository URL ထဲမှာလည်း `v1.29` ဖြစ်ရပါမယ်။

---

### Mistake 5 — kubeadm upgrade node ကို control plane apply နဲ့ရောထွေးခြင်း

Control plane:

```bash
sudo kubeadm upgrade apply v1.29.3
```

Worker node:

```bash
sudo kubeadm upgrade node
```

---

## 30. Practice Example

### Task

Cluster ကို `v1.28.0` မှ `v1.29.3` သို့ kubeadm နဲ့ upgrade လုပ်ပါ။  
Control plane name က `controlplane`, worker node name က `node01` ဖြစ်သည်။

---

### Control Plane Solution

```bash
sudo apt-cache madison kubeadm

sudo apt-mark unhold kubeadm && sudo apt-get update && sudo apt-get install -y kubeadm='1.29.3-1.1' && sudo apt-mark hold kubeadm

kubeadm version

sudo kubeadm upgrade plan

sudo kubeadm upgrade apply v1.29.3

kubectl drain controlplane --ignore-daemonsets

sudo apt-mark unhold kubelet kubectl && sudo apt-get update && sudo apt-get install -y kubelet='1.29.3-1.1' kubectl='1.29.3-1.1' && sudo apt-mark hold kubelet kubectl

sudo systemctl daemon-reload
sudo systemctl restart kubelet

kubectl uncordon controlplane

kubectl get nodes
```

---

### Worker Node Solution

```bash
sudo apt-mark unhold kubeadm && sudo apt-get update && sudo apt-get install -y kubeadm='1.29.3-1.1' && sudo apt-mark hold kubeadm

sudo kubeadm upgrade node
```

From control plane:

```bash
kubectl drain node01 --ignore-daemonsets
```

On worker node:

```bash
sudo apt-mark unhold kubelet kubectl && sudo apt-get update && sudo apt-get install -y kubelet='1.29.3-1.1' kubectl='1.29.3-1.1' && sudo apt-mark hold kubelet kubectl

sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

From control plane:

```bash
kubectl uncordon node01
kubectl get nodes
```

---

## 31. Key Takeaways

- kubeadm cluster upgrade ကို official Kubernetes documentation အတိုင်းလုပ်သင့်ပါတယ်။
- Legacy repositories deprecated ဖြစ်ပြီး new packages are hosted at `pkgs.k8s.io`.
- Upgrade မစခင် node OS နဲ့ package repository ကိုစစ်ပါ။
- Control plane ကိုအရင် upgrade လုပ်ပါ။
- `kubeadm upgrade plan` နဲ့ compatibility စစ်ပါ။
- `kubeadm upgrade apply v1.29.3` နဲ့ control plane upgrade လုပ်ပါ။
- kubelet and kubectl ကို manual upgrade လုပ်ရပါတယ်။
- Worker nodes တွေကို one by one upgrade လုပ်ပါ။
- Node upgrade မလုပ်ခင် `drain`, upgrade ပြီးနောက် `uncordon` လုပ်ပါ။
- `kubectl get nodes` ထဲက version က kubelet version ဖြစ်ပါတယ်။
