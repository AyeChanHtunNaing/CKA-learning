# OS Upgrades

> Topic: Kubernetes node maintenance / OS upgrade အတွက် `drain`, `cordon`, `uncordon` အသုံးပြုခြင်း

---

## 1. Overview

Kubernetes cluster ထဲမှာ worker node တွေကို maintenance လုပ်ဖို့လိုတဲ့အချိန်တွေရှိပါတယ်။

ဥပမာ —

- OS upgrade
- Security patch update
- Kernel update
- Docker / container runtime update
- kubelet update
- Hardware maintenance
- Node reboot

ဒီလို maintenance လုပ်တဲ့အခါ node ပေါ်မှာ run နေတဲ့ Pods တွေကို ဘယ်လို handle လုပ်မလဲဆိုတာ အရေးကြီးပါတယ်။

---

## 2. Node Down ဖြစ်ရင် ဘာဖြစ်မလဲ?

Node တစ်ခု down ဖြစ်သွားရင် အဲဒီ node ပေါ်က Pods တွေ inaccessible ဖြစ်သွားနိုင်ပါတယ်။

Application impact က Pod replicas အရေအတွက်ပေါ်မူတည်ပါတယ်။

ဥပမာ —

| Workload                           | Impact                                        |
| ---------------------------------- | --------------------------------------------- |
| Multiple replicas ရှိတဲ့ app       | Other replicas တွေရှိလို့ service ဆက်ပေးနိုင် |
| Single Pod ပဲရှိတဲ့ app            | Downtime ဖြစ်နိုင်                            |
| ReplicaSet/Deployment managed Pods | Other nodes ပေါ်မှာ recreate ဖြစ်နိုင်        |
| Standalone Pod                     | Automatically recreate မဖြစ်နိုင်             |

---

## 3. Short Node Downtime

Node တစ်ခု down ဖြစ်ပြီး ခဏအတွင်းပြန်တက်လာရင် Kubernetes က Pods တွေကို node ပေါ်မှာ ပြန် restart လုပ်နိုင်ပါတယ်။

Lesson ထဲမှာဖော်ပြထားတဲ့အတိုင်း node က **5 minutes ထက်နည်းတဲ့အချိန်** down ဖြစ်ပြီး ပြန်တက်လာရင် quick reboot/upgrade အတွက် acceptable ဖြစ်နိုင်ပါတယ်။

သို့သော် production workload တွေမှာ safer approach အနေနဲ့ node ကို drain လုပ်တာပိုကောင်းပါတယ်။

---

## 4. Node Down for More Than 5 Minutes

Node တစ်ခုက 5 minutes ထက်ကြာကြာ down ဖြစ်နေမယ်ဆိုရင် Kubernetes က အဲဒီ node ပေါ်က Pods တွေကို dead လို့သတ်မှတ်နိုင်ပါတယ်။

### ReplicaSet / Deployment managed Pods

ReplicaSet သို့မဟုတ် Deployment က manage လုပ်နေတဲ့ Pods တွေဆိုရင် Kubernetes က other healthy nodes ပေါ်မှာ new Pods တွေ recreate လုပ်နိုင်ပါတယ်။

### Standalone Pods

Standalone Pod ဆိုရင် ReplicaSet/Deployment မရှိတဲ့အတွက် automatically recreate မဖြစ်နိုင်ပါဘူး။

ဒါကြောင့် production မှာ Pod တွေကို standalone မထားဘဲ Deployment/ReplicaSet/StatefulSet နဲ့ manage လုပ်တာပိုကောင်းပါတယ်။

---

## 5. Node Maintenance Options

Node maintenance လုပ်ဖို့ Kubernetes မှာ အဓိက command ၃ ခုရှိပါတယ်။

| Command            | Purpose                                                                         |
| ------------------ | ------------------------------------------------------------------------------- |
| `kubectl drain`    | Node ပေါ်က Pods တွေကို safely evict/remove လုပ်ပြီး node ကို unschedulable လုပ် |
| `kubectl cordon`   | Node ကို unschedulable လုပ်သော်လည်း current Pods မရွှေ့                         |
| `kubectl uncordon` | Node ကို scheduling ပြန်ခွင့်ပြု                                                |

---

## 6. Draining a Node

`drain` command က node ပေါ်မှာ run နေတဲ့ Pods တွေကို gracefully terminate/evict လုပ်ပြီး other nodes ပေါ်မှာ recreate ဖြစ်စေပါတယ်။

Drain လုပ်တဲ့အခါ Kubernetes က node ကို automatically **cordon** လုပ်ပါတယ်။  
ဆိုလိုတာက new Pods တွေကို အဲဒီ node ပေါ် schedule မလုပ်တော့ပါဘူး။

Command:

```bash
kubectl drain node-1
```

---

## 7. Drain Flow

```text
1. kubectl drain node-1
2. Node becomes unschedulable
3. Pods on node-1 are evicted
4. ReplicaSet/Deployment creates replacement Pods on other nodes
5. Node is ready for maintenance/reboot/upgrade
6. Maintenance ပြီးရင် kubectl uncordon node-1
```

---

## 8. Why Drain is Safer

Drain က safer ဖြစ်တဲ့အကြောင်း —

- Workloads တွေကို maintenance မလုပ်ခင် other nodes ပေါ်ရွှေ့ပေးနိုင်တယ်
- New Pods တွေ node ပေါ် schedule မဖြစ်အောင်တားပေးတယ်
- Application downtime ကိုလျှော့နိုင်တယ်
- Node reboot/upgrade ကို safer လုပ်နိုင်တယ်

---

## 9. Uncordon a Node

Maintenance ပြီးသွားရင် node ကို scheduling ပြန်ခွင့်ပြုဖို့ `uncordon` လုပ်ရပါတယ်။

```bash
kubectl uncordon node-1
```

Uncordon လုပ်ပြီးနောက် Kubernetes scheduler က new Pods တွေကို အဲဒီ node ပေါ်မှာ schedule လုပ်နိုင်ပါပြီ။

---

## 10. Important Note About Uncordon

Uncordon လုပ်ပြီးနောက် drain လုပ်တုန်းက other nodes ပေါ်ရွှေ့သွားတဲ့ Pods တွေဟာ original node ပေါ်ကို automatic ပြန်မလာပါဘူး။

ဆိုလိုတာက —

```text
Pods moved from node-1 to node-2
uncordon node-1
Pods do not automatically move back to node-1
```

Kubernetes က current distribution ကိုဆက်ထိန်းထားပါတယ်။ New Pods တွေ create ဖြစ်တဲ့အချိန်မှ scheduler က node-1 ကိုပြန်ရွေးနိုင်ပါတယ်။

---

## 11. Cordon a Node

`cordon` command က node ကို unschedulable လုပ်ပေးပါတယ်။

ဒါပေမယ့် current running Pods တွေကို မဖျက်ပါဘူး၊ မရွှေ့ပါဘူး။

```bash
kubectl cordon node-1
```

Cordon လုပ်ပြီးနောက် —

- Existing Pods remain running
- New Pods will not be scheduled on that node

---

## 12. Cordon Flow

```text
1. kubectl cordon node-1
2. node-1 becomes unschedulable
3. Existing Pods keep running
4. New Pods are scheduled on other nodes
```

---

## 13. Drain vs Cordon

| Feature                         | `cordon`         | `drain` |
| ------------------------------- | ---------------- | ------- |
| Makes node unschedulable        | Yes              | Yes     |
| Evicts existing Pods            | No               | Yes     |
| Good for future scheduling stop | Yes              | Yes     |
| Good before maintenance/reboot  | Not enough alone | Yes     |
| Existing Pods continue running  | Yes              | No      |
| New Pods scheduled to node      | No               | No      |

---

## 14. When to Use Cordon

`cordon` ကိုသုံးသင့်တဲ့ case တွေ —

- Node ပေါ်မှာ new Pods မတက်စေချင်ဘူး
- Existing Pods ကိုတော့ run ထားချင်တယ်
- Maintenance မစခင် scheduling ကိုအရင်ပိတ်ထားချင်တယ်
- Node troubleshoot လုပ်နေချိန် new workload မဝင်စေချင်ဘူး

Example:

```bash
kubectl cordon node-1
```

---

## 15. When to Use Drain

`drain` ကိုသုံးသင့်တဲ့ case တွေ —

- Node reboot လုပ်မယ်
- OS upgrade လုပ်မယ်
- Security patch apply လုပ်မယ်
- Node ကို offline ယူမယ်
- Existing Pods တွေကို other nodes ပေါ်ရွှေ့ချင်တယ်

Example:

```bash
kubectl drain node-1
```

---

## 16. Check Node Status

Nodes ကြည့်ရန်:

```bash
kubectl get nodes
```

Cordon/drain လုပ်ထားတဲ့ node တွေမှာ status ထဲမှာ `SchedulingDisabled` တွေ့နိုင်ပါတယ်။

Example:

```text
NAME     STATUS                     ROLES    AGE   VERSION
node-1   Ready,SchedulingDisabled   worker   10d   v1.30.0
node-2   Ready                      worker   10d   v1.30.0
```

---

## 17. Drain with DaemonSets

Node ပေါ်မှာ DaemonSet Pods ရှိရင် normal drain command က error ဖြစ်နိုင်ပါတယ်။

DaemonSet-managed Pods တွေကို ignore လုပ်ဖို့ —

```bash
kubectl drain node-1 --ignore-daemonsets
```

CKA exam မှာ ဒီ flag အရေးကြီးပါတယ်။

---

## 18. Drain with Local Data

Pod တွေမှာ local storage / emptyDir data ရှိရင် drain command က warning/error ပေးနိုင်ပါတယ်။

Force delete local data warning ကို accept လုပ်ချင်ရင် —

```bash
kubectl drain node-1 --ignore-daemonsets --delete-emptydir-data
```

> `--delete-emptydir-data` သုံးရင် `emptyDir` volume data ပျက်နိုင်ပါတယ်။

---

## 19. Drain Force Option

Standalone Pod / unmanaged Pod ရှိရင် drain command က error ဖြစ်နိုင်ပါတယ်။

Force drain:

```bash
kubectl drain node-1 --ignore-daemonsets --force
```

သတိထားရန် — unmanaged Pods တွေဟာ recreate မဖြစ်နိုင်ပါဘူး။

---

## 20. Common Drain Command for CKA

CKA exam မှာ node drain လုပ်တဲ့အခါ ဒီ command ကိုအများဆုံးသုံးရနိုင်ပါတယ်။

```bash
kubectl drain node-1 --ignore-daemonsets --delete-emptydir-data
```

Maintenance ပြီးရင် —

```bash
kubectl uncordon node-1
```

---

## 21. OS Upgrade / Maintenance Flow

Node OS upgrade လုပ်တဲ့ typical flow:

```bash
kubectl drain node-1 --ignore-daemonsets --delete-emptydir-data
# perform OS upgrade / reboot / maintenance
kubectl uncordon node-1
```

Check:

```bash
kubectl get nodes
kubectl get pods -A -o wide
```

---

## 22. Verify Pods Moved to Other Nodes

Pods တွေ ဘယ် node ပေါ်မှာ run နေလဲကြည့်ရန်:

```bash
kubectl get pods -A -o wide
```

Example:

```text
NAMESPACE   NAME        READY   STATUS    NODE
default     app-abc     1/1     Running   node-2
default     app-def     1/1     Running   node-3
```

---

## 23. Important Commands

| Task                              | Command                                                           |
| --------------------------------- | ----------------------------------------------------------------- |
| Get nodes                         | `kubectl get nodes`                                               |
| Cordon node                       | `kubectl cordon node-1`                                           |
| Drain node                        | `kubectl drain node-1`                                            |
| Drain ignoring DaemonSets         | `kubectl drain node-1 --ignore-daemonsets`                        |
| Drain with emptyDir data deletion | `kubectl drain node-1 --ignore-daemonsets --delete-emptydir-data` |
| Force drain unmanaged Pods        | `kubectl drain node-1 --ignore-daemonsets --force`                |
| Uncordon node                     | `kubectl uncordon node-1`                                         |
| Check Pods and nodes              | `kubectl get pods -A -o wide`                                     |

---

## 24. CKA Exam Tips

### Tip 1 — Drain command မှတ်ထားပါ

```bash
kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data
```

---

### Tip 2 — Maintenance ပြီးရင် uncordon မမေ့ပါနဲ့

```bash
kubectl uncordon <node-name>
```

---

### Tip 3 — Cordon vs Drain ကိုခွဲမှတ်ပါ

```text
cordon = new Pods မတက်စေ
drain  = existing Pods တွေရွှေ့ + new Pods မတက်စေ
```

---

### Tip 4 — Pods original node ကို automatic မပြန်လာ

Uncordon လုပ်ပြီးနောက် Pod တွေ original node ကို automatic ပြန်မရွှေ့ပါဘူး။

---

### Tip 5 — Standalone Pod risk

ReplicaSet/Deployment မရှိတဲ့ Pod တွေဟာ drain/delete ပြီးနောက် automatically recreate မဖြစ်နိုင်ပါဘူး။

---

## 25. Common Mistakes

### Mistake 1 — Cordon လုပ်ရုံနဲ့ maintenance လုပ်ခြင်း

`cordon` က existing Pods တွေကိုမရွှေ့ပါဘူး။  
Node reboot/upgrade လုပ်မယ်ဆိုရင် `drain` လုပ်တာပိုမှန်ပါတယ်။

---

### Mistake 2 — Uncordon မလုပ်မိခြင်း

Maintenance ပြီးသွားပြီး `uncordon` မလုပ်ရင် node က SchedulingDisabled ဖြစ်နေပါမယ်။

Fix:

```bash
kubectl uncordon node-1
```

---

### Mistake 3 — DaemonSet error ကိုမဖြေရှင်းနိုင်ခြင်း

Drain command error ဖြစ်ရင် `--ignore-daemonsets` ထည့်ပါ။

```bash
kubectl drain node-1 --ignore-daemonsets
```

---

### Mistake 4 — emptyDir data ပျက်နိုင်တာကိုမသိခြင်း

`--delete-emptydir-data` သုံးရင် `emptyDir` data ပျက်နိုင်ပါတယ်။

---

## 26. Practice Example

### Task

Node `node-1` ကို OS upgrade အတွက် safely drain လုပ်ပါ။  
Upgrade ပြီးသွားရင် node ကို scheduling ပြန်ခွင့်ပြုပါ။

### Solution

Drain node:

```bash
kubectl drain node-1 --ignore-daemonsets --delete-emptydir-data
```

Check node:

```bash
kubectl get nodes
```

Check Pods moved:

```bash
kubectl get pods -A -o wide
```

After upgrade, uncordon:

```bash
kubectl uncordon node-1
```

Verify:

```bash
kubectl get nodes
```

---

## 27. Key Takeaways

- Node maintenance / OS upgrade မလုပ်ခင် node ကို drain လုပ်တာ safer ဖြစ်ပါတယ်။
- `kubectl drain` က existing Pods တွေကို evict လုပ်ပြီး node ကို unschedulable လုပ်ပါတယ်။
- `kubectl cordon` က node ကို unschedulable လုပ်ပေမယ့် existing Pods တွေကို မရွှေ့ပါဘူး။
- `kubectl uncordon` က node ကို scheduling ပြန်ခွင့်ပြုပါတယ်။
- ReplicaSet/Deployment-managed Pods တွေက other nodes ပေါ် recreate ဖြစ်နိုင်ပါတယ်။
- Standalone Pods တွေက automatically recreate မဖြစ်နိုင်ပါဘူး။
- Drain လုပ်တဲ့အခါ `--ignore-daemonsets` နဲ့ `--delete-emptydir-data` flags တွေ CKA မှာအသုံးများပါတယ်။
