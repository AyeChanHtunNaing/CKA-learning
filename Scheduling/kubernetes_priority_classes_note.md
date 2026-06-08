# Kubernetes Priority Classes Note

## 1. Priority Class ဆိုတာဘာလဲ

**Priority Class** ဆိုတာ Kubernetes မှာ Pod တွေရဲ့ အရေးကြီးမှုအဆင့်ကို numerical value နဲ့ သတ်မှတ်ပေးတဲ့ object ပါ။

Cluster ထဲမှာ workload တွေအမျိုးမျိုး run နိုင်ပါတယ်။

ဥပမာ -

- Control plane components
- Production database
- Critical application
- Normal application
- Background jobs

ဒီ workload တွေထဲမှာ အရေးကြီးမှု မတူပါဘူး။ အရေးကြီးတဲ့ workload တွေကို resource ရရှိအောင်၊ schedule အရင်လုပ်နိုင်အောင် Kubernetes က **Priority Class** ကို သုံးပါတယ်။

အလွယ်ပြောရရင် -

```text
Priority Class = Pod ရဲ့ အရေးကြီးမှု level သတ်မှတ်ပေးတာ
```

---

## 2. Priority Value အလုပ်လုပ်ပုံ

Priority Class မှာ `value` ဆိုတဲ့ number တစ်ခု သတ်မှတ်ပါတယ်။

```text
Higher value = Higher priority
Lower value  = Lower priority
```

ဥပမာ -

```text
Critical App Priority = 7
Job Priority          = 5
```

ဆိုရင် scheduler က critical app ကို job ထက် အရင် schedule လုပ်ဖို့ ကြိုးစားပါမယ်။

![Kubernetes Priority Classes Diagram](https://kodekloud.com/kk-media/image/upload/v1752869897/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-Priority-Classes/kubernetes-priorities-diagram.jpg)

---

## 3. Priority Value Range

User applications အတွက် priority value က အများအားဖြင့် -

```text
-2,000,000,000 to 1,000,000,000
```

အတွင်း သတ်မှတ်နိုင်ပါတယ်။

Kubernetes internal system-critical Pods တွေအတွက်တော့ reserved range ရှိပြီး value က 2 billion အထိ သုံးနိုင်ပါတယ်။

ဥပမာ system priority classes -

```text
system-cluster-critical
system-node-critical
```

ဒီလို system-critical PriorityClasses တွေက Kubernetes control plane နဲ့ node critical components အတွက် သုံးပါတယ်။

---

## 4. Existing Priority Classes ကြည့်နည်း

Cluster ထဲမှာ ရှိပြီးသား PriorityClasses တွေကို ကြည့်ချင်ရင် -

```bash
kubectl get priorityclass
```

Output example -

```bash
NAME                      VALUE          GLOBAL-DEFAULT   AGE     PREEMPTIONPOLICY
system-cluster-critical   2000000000     false            7m33s   PreemptLowerPriority
system-node-critical      2000010000     false            7m33s   PreemptLowerPriority
```

Short name အနေနဲ့လည်း သုံးနိုင်ပါတယ်။

```bash
kubectl get pc
```

---

## 5. PriorityClass YAML Create လုပ်နည်း

PriorityClass object create လုပ်ချင်ရင် `apiVersion` က -

```yaml
apiVersion: scheduling.k8s.io/v1
```

ဖြစ်ရပါမယ်။

`kind` က -

```yaml
kind: PriorityClass
```

ဖြစ်ပါတယ်။

Example -

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000000000
description: "Priority class for mission critical pods"
```

ဒီ YAML မှာ -

```yaml
name: high-priority
```

က PriorityClass name ဖြစ်ပါတယ်။

```yaml
value: 1000000000
```

က priority value ဖြစ်ပါတယ်။

```yaml
description:
```

က explanation အတွက် optional field ဖြစ်ပါတယ်။

---

## 6. PriorityClass ကို Create လုပ်နည်း

YAML file ကို `priority-class.yaml` လို့ save ထားတယ်ဆိုရင် -

```bash
kubectl apply -f priority-class.yaml
```

သို့မဟုတ် -

```bash
kubectl create -f priority-class.yaml
```

စစ်ရန် -

```bash
kubectl get priorityclass
```

---

## 7. Pod မှာ PriorityClass သုံးနည်း

Pod ကို PriorityClass assign လုပ်ချင်ရင် Pod spec ထဲမှာ `priorityClassName` ထည့်ရပါတယ်။

Example -

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  containers:
    - name: nginx
      image: nginx
      ports:
        - containerPort: 8080
  priorityClassName: high-priority
```

အရေးကြီးတဲ့ field က -

```yaml
priorityClassName: high-priority
```

ဖြစ်ပါတယ်။

ဒီ Pod က `high-priority` PriorityClass ရဲ့ value ကို သုံးပါမယ်။

---

## 8. PriorityClass နဲ့ Pod Full Example

```yaml
# priority-class.yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000000000
description: "Priority class for mission critical pods"
globalDefault: true

---
# pod-definition.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  containers:
    - name: nginx
      image: nginx
      ports:
        - containerPort: 8080
  priorityClassName: high-priority
```

ဒီ example မှာ -

- `high-priority` PriorityClass ကို create လုပ်ထားတယ်။
- `nginx` Pod မှာ `priorityClassName: high-priority` ထည့်ထားတယ်။
- `globalDefault: true` ကြောင့် default priority class အဖြစ် သတ်မှတ်ထားတယ်။

---

## 9. Default Priority

Pod မှာ `priorityClassName` မထည့်ထားရင် Pod ရဲ့ priority value က default အနေနဲ့ -

```text
0
```

ဖြစ်ပါတယ်။

PriorityClass တစ်ခုကို cluster default အဖြစ် သတ်မှတ်ချင်ရင် -

```yaml
globalDefault: true
```

ထည့်ပေးရပါတယ်။

သတိထားရန် -

```text
Cluster တစ်ခုမှာ globalDefault=true ဖြစ်တဲ့ PriorityClass တစ်ခုပဲ ရှိရပါမယ်။
```

---

## 10. `globalDefault` ဆိုတာဘာလဲ

`globalDefault` ဆိုတာ PriorityClass ကို default priority class အဖြစ် သတ်မှတ်ပေးတဲ့ field ပါ။

Example -

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: default-priority
value: 1000
globalDefault: true
description: "Default priority for pods"
```

ဒီလိုသတ်မှတ်ထားရင် `priorityClassName` မထည့်ထားတဲ့ Pod တွေက ဒီ PriorityClass ရဲ့ value ကို default အနေနဲ့ သုံးနိုင်ပါတယ်။

---

## 11. Pod Priority and Scheduling

Scheduler က Pod တွေကို schedule လုပ်တဲ့အခါ priority value ကို ထည့်စဉ်းစားပါတယ်။

ဥပမာ workload ၂ ခုရှိတယ်ဆိုပါစို့။

```text
Critical App = priority 7
Job          = priority 5
```

Resource ရှိရင် Critical App ကို အရင် schedule လုပ်ပါတယ်။ Resource ကျန်ရင် Job ကို schedule လုပ်ပါတယ်။

![Pod Priority Comparison](https://kodekloud.com/kk-media/image/upload/v1752869898/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-Priority-Classes/pod-priority-comparison-jobs-apps.jpg)

---

## 12. Preemption ဆိုတာဘာလဲ

**Preemption** ဆိုတာ higher priority Pod တစ်ခုကို schedule လုပ်ဖို့ resource မလုံလောက်တဲ့အခါ lower priority Pod တွေကို evict/remove လုပ်ပြီး resource ဖယ်ပေးတာပါ။

ဥပမာ -

```text
Existing Pod = priority 5
New Pod      = priority 6
No free resources
```

ဒီအချိန်မှာ new Pod ရဲ့ PriorityClass မှာ `PreemptLowerPriority` policy ရှိရင် scheduler က lower priority Pod ကို evict လုပ်ပြီး new Pod ကို schedule လုပ်နိုင်ပါတယ်။

အလွယ်မှတ်ရန် -

```text
Preemption = Higher priority Pod အတွက် lower priority Pod ကို ဖယ်ပေးတာ
```

---

## 13. Preemption Policy အမျိုးအစား

PriorityClass မှာ `preemptionPolicy` ကို သတ်မှတ်နိုင်ပါတယ်။

အဓိက ၂ မျိုးရှိပါတယ်။

| Policy | အဓိပ္ပါယ် |
|---|---|
| `PreemptLowerPriority` | Lower priority Pod တွေကို evict လုပ်နိုင်တယ် |
| `Never` | Lower priority Pod တွေကို မ evict လုပ်ဘဲ queue ထဲမှာ စောင့်နေမယ် |

Default policy က -

```text
PreemptLowerPriority
```

ဖြစ်ပါတယ်။

---

## 14. PreemptLowerPriority Example

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000000000
description: "Priority class for mission critical pods"
preemptionPolicy: PreemptLowerPriority
```

ဒီ YAML ရဲ့ အဓိပ္ပါယ်က -

```text
high-priority Pod ကို schedule လုပ်ဖို့ resource မလုံလောက်ရင် lower priority Pod တွေကို evict လုပ်နိုင်တယ်
```

---

## 15. Never Preemption Policy Example

တကယ်လို့ higher priority Pod ဖြစ်ပေမယ့် lower priority Pod တွေကို မဖယ်ချင်ဘူးဆိုရင် `Never` သုံးနိုင်ပါတယ်။

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000000000
description: "Priority class for mission critical pods"
preemptionPolicy: Never
```

ဒီ YAML ရဲ့ အဓိပ္ပါယ်က -

```text
high-priority Pod ဖြစ်ပေမယ့် lower priority Pod တွေကို evict မလုပ်ဘူး။
Resource ရတဲ့အထိ scheduling queue ထဲမှာ စောင့်မယ်။
```

---

## 16. Priority Class Fields Summary

| Field | Description |
|---|---|
| `apiVersion` | `scheduling.k8s.io/v1` |
| `kind` | `PriorityClass` |
| `metadata.name` | PriorityClass name |
| `value` | Priority numerical value |
| `description` | PriorityClass အကြောင်းရှင်းပြချက် |
| `globalDefault` | Default priority class အဖြစ်သတ်မှတ်ရန် |
| `preemptionPolicy` | Lower priority Pod တွေကို evict လုပ်မလား သတ်မှတ်ရန် |

---

## 17. အသုံးများတဲ့ Commands

### PriorityClasses ကြည့်ရန်

```bash
kubectl get priorityclass
```

Short form -

```bash
kubectl get pc
```

### PriorityClass create/apply လုပ်ရန်

```bash
kubectl apply -f priority-class.yaml
```

### PriorityClass detail ကြည့်ရန်

```bash
kubectl describe priorityclass high-priority
```

### Pod detail ထဲမှာ priority ကြည့်ရန်

```bash
kubectl describe pod nginx
```

### PriorityClass delete လုပ်ရန်

```bash
kubectl delete priorityclass high-priority
```

---

## 18. CKA Exam အတွက် မှတ်ရန်

CKA exam မှာ PriorityClass ကို scheduling concept အနေနဲ့ သိထားသင့်ပါတယ်။

အဓိက မှတ်ရန် -

- PriorityClass က Pod အရေးကြီးမှုကို numerical value နဲ့ သတ်မှတ်တယ်။
- Higher value = higher priority
- Pod မှာ `priorityClassName` ထည့်ပြီး သုံးတယ်။
- `priorityClassName` မရှိရင် default priority က 0 ဖြစ်တယ်။
- `globalDefault: true` သတ်မှတ်ထားတဲ့ PriorityClass က default ဖြစ်နိုင်တယ်။
- Cluster တစ်ခုမှာ global default PriorityClass တစ်ခုပဲ ရှိရမယ်။
- Default preemption policy က `PreemptLowerPriority` ဖြစ်တယ်။
- `preemptionPolicy: Never` ဆိုရင် lower priority Pod တွေကို မ evict လုပ်ဘူး။
- System critical priority classes တွေရှိနိုင်တယ်။
- `kubectl get priorityclass` command သိထားရမယ်။

---

## 19. Simple Summary

PriorityClass ဆိုတာ Kubernetes မှာ Pod တွေရဲ့ priority ကို number နဲ့ သတ်မှတ်ပေးတဲ့ object ပါ။

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000000000
description: "Priority class for mission critical pods"
```

Pod မှာ သုံးချင်ရင် -

```yaml
priorityClassName: high-priority
```

ထည့်ရပါတယ်။

Higher priority Pod တွေကို scheduler က အရေးကြီးအနေနဲ့ စဉ်းစားပါတယ်။

Resource မလုံလောက်တဲ့အခါ `PreemptLowerPriority` policy ကြောင့် lower priority Pod တွေ evict ဖြစ်နိုင်ပါတယ်။

Lower priority Pod တွေကို မဖယ်ချင်ရင် -

```yaml
preemptionPolicy: Never
```

သုံးနိုင်ပါတယ်။

အရေးကြီးဆုံး မှတ်ရန် -

```text
Higher value = Higher priority
Default Pod priority = 0
Default preemptionPolicy = PreemptLowerPriority
```
