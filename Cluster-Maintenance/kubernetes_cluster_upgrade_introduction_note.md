# Cluster Upgrade Introduction

> Topic: Kubernetes Cluster upgrade process, version skew rules, control plane upgrade, and worker node upgrade

---

## 1. Overview

Kubernetes cluster upgrade ဆိုတာ cluster ထဲက Kubernetes components တွေရဲ့ version ကို newer version သို့ update လုပ်တာဖြစ်ပါတယ်။

Cluster upgrade မှာ အဓိကအားဖြင့် —

- Control plane / master nodes upgrade
- Worker nodes upgrade
- kubeadm upgrade
- kubelet upgrade
- kube-proxy upgrade
- Node drain / uncordon

စတာတွေပါဝင်ပါတယ်။

ဒီ lesson က cluster upgrade concept ကို introduction အနေနဲ့ရှင်းပြထားပြီး CKA exam အတွက် အရေးကြီးတဲ့ `kubeadm upgrade`, `drain`, `uncordon` flow ကိုမှတ်ထားဖို့လိုပါတယ်။

---

## 2. Kubernetes Components Version

Kubernetes cluster ထဲမှာ component အများကြီးရှိပါတယ်။

အရေးကြီးတဲ့ components တွေက —

- kube-apiserver
- kube-controller-manager
- kube-scheduler
- kubelet
- kube-proxy
- kubectl
- etcd
- CoreDNS

ဒီ lesson မှာ mainly control plane components နဲ့ kubelet/kube-proxy upgrade ကို focus လုပ်ပါတယ်။  
External dependencies လိုမျိုး etcd, CoreDNS ကိုတော့ aside ထားပါတယ်။

---

## 3. kube-apiserver is the Main Component

Kubernetes control plane ထဲမှာ **kube-apiserver** က central component ဖြစ်ပါတယ်။

အခြား components အားလုံးက API server နဲ့ communicate လုပ်ပါတယ်။

ဒါကြောင့် rule အရေးကြီးပါတယ်။

```text
No component should run on a version higher than kube-apiserver.
```

ဥပမာ API server က `v1.10` ဖြစ်နေရင် component တစ်ခုက `v1.11` ဖြစ်နေတာ မသင့်တော်ပါဘူး။

---

## 4. Version Skew Rules

Kubernetes components တွေက version တူရမယ်လို့မလိုပါဘူး။  
သတ်မှတ်ထားတဲ့ version skew range အတွင်းမှာ version ကွာနိုင်ပါတယ်။

ဥပမာ kube-apiserver က `v1.10` ဖြစ်တယ်ဆိုပါစို့။

| Component               | Allowed Version                                     |
| ----------------------- | --------------------------------------------------- |
| kube-apiserver          | `v1.10`                                             |
| kube-controller-manager | `v1.10` or `v1.9`                                   |
| kube-scheduler          | `v1.10` or `v1.9`                                   |
| kubelet                 | `v1.10`, `v1.9`, or `v1.8`                          |
| kube-proxy              | `v1.10`, `v1.9`, or `v1.8`                          |
| kubectl                 | can be higher, same, or lower within supported skew |

---

## 5. Version Skew Easy Memory

```text
API Server = highest version
Controller Manager / Scheduler = up to 1 minor lower
Kubelet / Kube Proxy = up to 2 minor lower
kubectl = can be higher/lower/same
```

Example:

```text
kube-apiserver: v1.10
controller-manager: v1.10 or v1.9
scheduler: v1.10 or v1.9
kubelet: v1.10, v1.9, or v1.8
kube-proxy: v1.10, v1.9, or v1.8
```

---

## 6. kubectl Exception

`kubectl` က API server version ထက် higher/lower/same version ဖြစ်နိုင်ပါတယ်။

ဒါက live rolling upgrade တွေမှာ component တစ်ခုချင်းစီကို upgrade လုပ်နိုင်ဖို့ flexibility ပေးတာပါ။

---

## 7. When to Upgrade

Kubernetes က normally latest **three minor versions** ကို support လုပ်ပါတယ်။

ဥပမာ latest version က `v1.12` ဆိုရင် supported versions တွေက —

```text
v1.12
v1.11
v1.10
```

`v1.13` release ဖြစ်လာရင် supported versions တွေက —

```text
v1.13
v1.12
v1.11
```

ဒါဆို `v1.10` က support မဖြစ်တော့ပါဘူး။

ဒါကြောင့် current version support မကျမီ upgrade လုပ်သင့်ပါတယ်။

---

## 8. Upgrade One Minor Version at a Time

Kubernetes cluster upgrade ကို one minor version at a time လုပ်တာ best practice ဖြစ်ပါတယ်။

မသင့်တော်:

```text
v1.10 → v1.13
```

သင့်တော်:

```text
v1.10 → v1.11 → v1.12 → v1.13
```

တစ်ဆင့်ချင်း upgrade လုပ်တာက safer ဖြစ်ပြီး compatibility issue လျော့စေပါတယ်။

---

## 9. Cluster Upgrade Methods

Cluster setup ပေါ်မူတည်ပြီး upgrade method ကွာနိုင်ပါတယ်။

| Cluster Type               | Upgrade Method                             |
| -------------------------- | ------------------------------------------ |
| Managed Kubernetes         | Cloud provider UI/API မှ upgrade လုပ်နိုင် |
| kubeadm cluster            | `kubeadm upgrade` command နဲ့ upgrade      |
| Manually installed cluster | Components တစ်ခုချင်း manually upgrade     |

Managed Kubernetes examples:

- Google Kubernetes Engine — GKE
- Amazon EKS
- Azure AKS

---

## 10. Upgrade Process Overview

Production cluster တစ်ခုမှာ master/control plane nodes နဲ့ worker nodes ရှိပါတယ်။

Cluster upgrade process က generally two major steps ပါ။

1. Upgrade master/control plane nodes
2. Upgrade worker nodes

---

## 11. Control Plane Upgrade

Control plane upgrade လုပ်တဲ့အခါ upgrade ဖြစ်နိုင်တဲ့ components —

- kube-apiserver
- kube-controller-manager
- kube-scheduler
- control plane static pods
- kubeadm-managed control plane components

Control plane upgrade အချိန်မှာ cluster management operations ခဏရပ်နိုင်ပါတယ်။

ဥပမာ —

- `kubectl` commands response မရနိုင်
- Deployment scale မလုပ်နိုင်
- Scheduler/controller operations ခဏ delay ဖြစ်နိုင်

ဒါပေမယ့် worker nodes ပေါ်က running Pods တွေက ဆက် run နေနိုင်ပါတယ်။

---

## 12. During Master Upgrade

Master/control plane upgrade ဖြစ်နေချိန် —

| Function                   | Status                           |
| -------------------------- | -------------------------------- |
| Running workloads          | Continue running on worker nodes |
| API server access          | Brief interruption ဖြစ်နိုင်     |
| New scheduling             | Pause/delay ဖြစ်နိုင်            |
| Pod restart by controllers | Delay ဖြစ်နိုင်                  |
| Scaling deployments        | Pause/delay ဖြစ်နိုင်            |

Important:

Control plane down ဖြစ်နေချိန် pod တစ်ခု fail ဖြစ်သွားရင် controller က ချက်ချင်း recreate မလုပ်နိုင်ပါဘူး။  
Control plane ပြန်တက်လာမှ normal operations resume ဖြစ်ပါတယ်။

---

## 13. Worker Node Upgrade Strategies

Control plane upgrade ပြီးရင် worker nodes upgrade လုပ်ရပါတယ်။

Worker node upgrade strategy ၃ မျိုးရှိပါတယ်။

1. Upgrade all worker nodes at the same time
2. Upgrade one worker node at a time
3. Add new upgraded nodes, move workloads, then remove old nodes

---

## 14. Strategy 1 — Upgrade All Worker Nodes Together

ဒီနည်းက worker nodes အားလုံးကို တစ်ပြိုင်နက် upgrade လုပ်တာပါ။

အားနည်းချက် —

- Application downtime ဖြစ်နိုင်
- Production မှာ risky ဖြစ်တယ်
- Workloads အားလုံး impact ဖြစ်နိုင်

ဒါကြောင့် production မှာ generally မသင့်တော်ပါဘူး။

---

## 15. Strategy 2 — Upgrade One Worker Node at a Time

ဒီနည်းက recommended approach ဖြစ်ပါတယ်။

Flow:

```text
1. Drain worker node
2. Upgrade kubeadm/kubelet/kubectl packages
3. Upgrade node config
4. Restart kubelet
5. Uncordon node
6. Repeat for next worker node
```

ဒီနည်းက applications availability ကိုထိန်းနိုင်ပါတယ်။

---

## 16. Strategy 3 — Add New Nodes and Remove Old Nodes

ဒီနည်းက cloud environment မှာအသုံးများပါတယ်။

Flow:

```text
1. Create new worker nodes with newer Kubernetes version
2. Join them to the cluster
3. Move workloads to new nodes
4. Drain old nodes
5. Remove old nodes from cluster
```

အားသာချက် —

- Downtime လျှော့နိုင်
- Old nodes ကို cleanly decommission လုပ်နိုင်
- Cloud/VM environment နဲ့ကိုက်တယ်

---

## 17. Upgrade with kubeadm

kubeadm cluster မှာ upgrade process ကို kubeadm command တွေနဲ့လုပ်နိုင်ပါတယ်။

Upgrade plan ကြည့်ရန်:

```bash
kubeadm upgrade plan
```

ဒီ command က ပြပေးတာတွေ —

- Current cluster version
- kubeadm tool version
- Latest stable Kubernetes version
- Current control plane component versions
- Target upgrade versions
- Upgrade command suggestion

---

## 18. Important kubeadm Note

kubeadm က control plane components ကို upgrade လုပ်နိုင်ပေမယ့် kubelet ကို automatically upgrade မလုပ်ပါဘူး။

```text
kubeadm does not upgrade kubelet automatically.
```

ဒါကြောင့် kubelet ကို node တစ်ခုချင်းစီမှာ package manager နဲ့ manual upgrade လုပ်ပြီး restart လုပ်ရပါတယ်။

---

## 19. Upgrade Order with kubeadm

ဥပမာ `v1.11` ကနေ `v1.13` သို့ upgrade လုပ်ချင်တယ်ဆိုပါစို့။

Direct jump မလုပ်ပါနဲ့။

Correct order:

```text
v1.11 → v1.12 → v1.13
```

For each minor version:

```text
1. Upgrade kubeadm
2. Run kubeadm upgrade plan
3. Run kubeadm upgrade apply <version>
4. Upgrade kubelet and kubectl
5. Restart kubelet
6. Upgrade worker nodes one by one
```

---

## 20. Control Plane Upgrade Example

ဥပမာ cluster ကို `v1.11` ကနေ `v1.12.0` သို့ upgrade လုပ်မယ်ဆိုပါစို့။

### Step 1 — Upgrade kubeadm

```bash
apt-get upgrade -y kubeadm=1.12.0-00
```

### Step 2 — Check upgrade plan

```bash
kubeadm upgrade plan
```

### Step 3 — Apply upgrade

```bash
kubeadm upgrade apply v1.12.0
```

Success output example:

```text
[upgrade/successful] SUCCESS! Your cluster was upgraded to "v1.12.0".
[upgrade/kubelet] Now that your control plane is upgraded, please proceed with upgrading your kubelets.
```

---

## 21. Why `kubectl get nodes` Still Shows Old Version

Control plane upgrade ပြီးနောက် `kubectl get nodes` run လုပ်တဲ့အခါ master node version က old version အတိုင်းပြနိုင်ပါတယ်။

```bash
kubectl get nodes
```

ဘာကြောင့်လဲဆိုတော့ `kubectl get nodes` မှာပြတဲ့ version က **node ရဲ့ kubelet version** ဖြစ်ပါတယ်။  
API server version မဟုတ်ပါဘူး။

ဒါကြောင့် control plane upgrade ပြီးနောက် kubelet ကိုလည်း upgrade လုပ်ရပါတယ်။

---

## 22. Upgrade kubelet on Control Plane Node

kubeadm cluster မှာ master/control plane node ပေါ်မှာလည်း kubelet run နေပါတယ်။  
Static pods ဖြစ်တဲ့ control plane components တွေကို kubelet က manage လုပ်ပါတယ်။

Upgrade kubelet:

```bash
apt-get upgrade -y kubelet=1.12.0-00
```

Restart kubelet:

```bash
systemctl restart kubelet
```

Verify:

```bash
kubectl get nodes
```

---

## 23. Worker Node Upgrade Flow

Worker node upgrade ကို one node at a time လုပ်သင့်ပါတယ်။

### Step 1 — Drain worker node

```bash
kubectl drain node-1
```

CKA lab တွေမှာ flags လိုနိုင်ပါတယ်။

```bash
kubectl drain node-1 --ignore-daemonsets --delete-emptydir-data
```

### Step 2 — Upgrade kubeadm on worker node

```bash
apt-get upgrade -y kubeadm=1.12.0-00
```

### Step 3 — Upgrade node configuration

```bash
kubeadm upgrade node
```

### Step 4 — Upgrade kubelet

```bash
apt-get upgrade -y kubelet=1.12.0-00
```

### Step 5 — Restart kubelet

```bash
systemctl restart kubelet
```

### Step 6 — Uncordon worker node

```bash
kubectl uncordon node-1
```

### Step 7 — Verify

```bash
kubectl get nodes
```

Repeat for each worker node.

---

## 24. Drain and Uncordon in Worker Upgrade

Before upgrade:

```bash
kubectl drain node-1 --ignore-daemonsets --delete-emptydir-data
```

After upgrade:

```bash
kubectl uncordon node-1
```

Why?

- `drain` က workloads တွေကို other nodes ပေါ်ရွှေ့ပေးတယ်
- node ကို unschedulable လုပ်တယ်
- upgrade/reboot ကို safe ဖြစ်စေတယ်
- `uncordon` က upgrade ပြီးနောက် scheduling ပြန်ခွင့်ပြုတယ်

---

## 25. Important Commands Summary

| Task                        | Command                                                           |
| --------------------------- | ----------------------------------------------------------------- |
| Check nodes                 | `kubectl get nodes`                                               |
| Check upgrade plan          | `kubeadm upgrade plan`                                            |
| Upgrade kubeadm             | `apt-get upgrade -y kubeadm=<version>-00`                         |
| Apply control plane upgrade | `kubeadm upgrade apply v<version>`                                |
| Upgrade worker node config  | `kubeadm upgrade node`                                            |
| Upgrade kubelet             | `apt-get upgrade -y kubelet=<version>-00`                         |
| Restart kubelet             | `systemctl restart kubelet`                                       |
| Drain node                  | `kubectl drain <node> --ignore-daemonsets --delete-emptydir-data` |
| Uncordon node               | `kubectl uncordon <node>`                                         |
| Check pods node placement   | `kubectl get pods -A -o wide`                                     |

---

## 26. Upgrade Flow Summary

### Control Plane Node

```bash
apt-get upgrade -y kubeadm=1.12.0-00
kubeadm upgrade plan
kubeadm upgrade apply v1.12.0
apt-get upgrade -y kubelet=1.12.0-00
systemctl restart kubelet
kubectl get nodes
```

### Worker Node

```bash
kubectl drain node-1 --ignore-daemonsets --delete-emptydir-data
apt-get upgrade -y kubeadm=1.12.0-00
kubeadm upgrade node
apt-get upgrade -y kubelet=1.12.0-00
systemctl restart kubelet
kubectl uncordon node-1
kubectl get nodes
```

---

## 27. CKA Exam Tips

### Tip 1 — Upgrade one minor version at a time

```text
v1.11 → v1.12 → v1.13
```

Direct jump မလုပ်ပါနဲ့။

---

### Tip 2 — API server must be highest or equal

```text
No component should be higher than kube-apiserver.
```

---

### Tip 3 — kubeadm does not upgrade kubelet automatically

Control plane upgrade ပြီးနောက် kubelet ကို package manager နဲ့ upgrade လုပ်ပြီး restart လုပ်ပါ။

---

### Tip 4 — `kubectl get nodes` shows kubelet version

Node version column က API server version မဟုတ်ပါဘူး။  
Node ပေါ်က kubelet version ဖြစ်ပါတယ်။

---

### Tip 5 — Worker nodes upgrade one by one

```bash
kubectl drain node-1 --ignore-daemonsets --delete-emptydir-data
# upgrade packages
kubectl uncordon node-1
```

---

## 28. Common Mistakes

### Mistake 1 — kubelet upgrade မလုပ်မိခြင်း

`kubeadm upgrade apply` လုပ်ပြီးနောက် `kubectl get nodes` မှာ old version ပြနေတယ်ဆိုရင် kubelet မ upgrade ရသေးတာဖြစ်နိုင်ပါတယ်။

---

### Mistake 2 — Direct minor version jump

မမှန်:

```text
v1.11 → v1.13
```

မှန်:

```text
v1.11 → v1.12 → v1.13
```

---

### Mistake 3 — Worker node drain မလုပ်ဘဲ upgrade/reboot လုပ်ခြင်း

Production မှာ downtime ဖြစ်နိုင်ပါတယ်။  
Worker node upgrade မလုပ်ခင် drain လုပ်ပါ။

---

### Mistake 4 — Uncordon မလုပ်မိခြင်း

Upgrade ပြီးနောက် uncordon မလုပ်ရင် node က SchedulingDisabled ဖြစ်နေပါမယ်။

Fix:

```bash
kubectl uncordon node-1
```

---

## 29. Practice Example

### Task

Cluster ကို `v1.11` ကနေ `v1.12.0` သို့ kubeadm နဲ့ upgrade လုပ်ပါ။  
Worker node `node-1` ကို safely upgrade လုပ်ပါ။

### Control Plane Solution

```bash
apt-get upgrade -y kubeadm=1.12.0-00
kubeadm upgrade plan
kubeadm upgrade apply v1.12.0
apt-get upgrade -y kubelet=1.12.0-00
systemctl restart kubelet
kubectl get nodes
```

### Worker Node Solution

```bash
kubectl drain node-1 --ignore-daemonsets --delete-emptydir-data
apt-get upgrade -y kubeadm=1.12.0-00
kubeadm upgrade node
apt-get upgrade -y kubelet=1.12.0-00
systemctl restart kubelet
kubectl uncordon node-1
kubectl get nodes
```

---

## 30. Key Takeaways

- Kubernetes components တွေက version တူနေစရာမလိုပေမယ့် version skew rules ကိုလိုက်နာရပါတယ်။
- kube-apiserver က primary control plane component ဖြစ်ပြီး အခြား components တွေက API server ထက် higher version မဖြစ်သင့်ပါ။
- Kubernetes officially supports latest three minor versions ကို support လုပ်ပါတယ်။
- Cluster upgrade ကို one minor version at a time လုပ်သင့်ပါတယ်။
- Upgrade process က control plane upgrade ပြီးမှ worker node upgrade လုပ်တာဖြစ်ပါတယ်။
- kubeadm က control plane upgrade ကိုကူညီပေးပေမယ့် kubelet ကို automatically upgrade မလုပ်ပါဘူး။
- `kubectl get nodes` ထဲက version က kubelet version ဖြစ်ပါတယ်။
- Worker nodes တွေကို one by one drain, upgrade, uncordon လုပ်တာ recommended ဖြစ်ပါတယ်။
