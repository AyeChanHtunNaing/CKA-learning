# Kubernetes Node Selectors Note

## 1. Node Selector ဆိုတာဘာလဲ

**Node Selector** ဆိုတာ Kubernetes မှာ Pod တစ်ခုကို သတ်မှတ်ထားတဲ့ Node ပေါ်မှာပဲ run စေချင်တဲ့အခါ သုံးတဲ့ scheduling method ပါ။

ပုံမှန်အားဖြင့် Kubernetes Scheduler က Pod တွေကို cluster ထဲက available node တစ်ခုပေါ်မှာ အလိုအလျောက် schedule လုပ်ပါတယ်။

ဒါပေမယ့် တချို့ workload တွေမှာ specific hardware resource လိုနိုင်ပါတယ်။

ဥပမာ -

- Data processing pod
- Machine learning workload
- High CPU workload
- High memory workload

ဒီလို Pod တွေကို resource များတဲ့ Node ပေါ်မှာပဲ run စေချင်ရင် **Node Selector** ကို သုံးနိုင်ပါတယ်။

---

## 2. Node Selector ကို ဘာကြောင့်သုံးလဲ

Cluster ထဲမှာ Node ၃ ခုရှိတယ်ဆိုပါစို့။

```text
Node 1 = Small
Node 2 = Small
Node 3 = Large
```

Data processing workload တစ်ခုက CPU/RAM များများလိုတယ်ဆိုရင် `Large` node ပေါ်မှာပဲ run သင့်ပါတယ်။

Node Selector မသုံးရင် Scheduler က Pod ကို ဘယ် Node ပေါ်မဆို တင်နိုင်ပါတယ်။

အဲ့ဒီအခါ resource နည်းတဲ့ Node ပေါ် Pod တက်သွားရင် performance problem ဖြစ်နိုင်ပါတယ်။

Node Selector သုံးရင် -

```text
Pod needs size=Large
        ↓
Scheduler searches node with label size=Large
        ↓
Pod runs on Large node
```

---

## 3. Node Selector အလုပ်လုပ်ပုံ

Node Selector က **Pod specification** ထဲမှာ key-value pair ထည့်ပြီး Node label နဲ့ match လုပ်ပါတယ်။

အလုပ်လုပ်ပုံက -

1. Node ကို label တပ်ထားရမယ်။
2. Pod YAML ထဲမှာ `nodeSelector` ထည့်ရမယ်။
3. Scheduler က Pod ရဲ့ `nodeSelector` နဲ့ ကိုက်တဲ့ Node ကို ရှာမယ်။
4. Label ကိုက်တဲ့ Node ပေါ်မှာ Pod ကို run မယ်။

---

## 4. Node ကို Label တပ်နည်း

Node Selector သုံးဖို့အတွက် အရင်ဆုံး Node ကို label တပ်ရပါတယ်။

ဥပမာ `node-1` ကို `size=Large` လို့ label တပ်ချင်ရင် -

```bash
kubectl label nodes node-1 size=Large
```

ဒီ command ရဲ့ အဓိပ္ပါယ်က -

```text
node-1 ဆိုတဲ့ Node ကို size=Large ဆိုတဲ့ label တပ်မယ်
```

Label တပ်ပြီးသား Node တွေကို ကြည့်ချင်ရင် -

```bash
kubectl get nodes --show-labels
```

---

## 5. Pod မှာ Node Selector ထည့်နည်း

Pod ကို `size=Large` label ရှိတဲ့ Node ပေါ်မှာပဲ run စေချင်ရင် Pod manifest ထဲမှာ `nodeSelector` ထည့်ပေးရပါတယ်။

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
    - name: data-processor
      image: data-processor
  nodeSelector:
    size: Large
```

ဒီ YAML မှာ အရေးကြီးတာက -

```yaml
nodeSelector:
  size: Large
```

ဖြစ်ပါတယ်။

အဓိပ္ပါယ်က -

```text
ဒီ Pod ကို size=Large label ရှိတဲ့ Node ပေါ်မှာပဲ run ပါ
```

---

## 6. Pod Create လုပ်နည်း

Pod YAML file ကို `pod-definition.yaml` လို့ save ထားတယ်ဆိုရင် create လုပ်ရန် -

```bash
kubectl create -f pod-definition.yaml
```

သို့မဟုတ် -

```bash
kubectl apply -f pod-definition.yaml
```

Pod ဘယ် Node ပေါ်မှာ run နေလဲ စစ်ရန် -

```bash
kubectl get pods -o wide
```

Output မှာ `NODE` column ကို ကြည့်ရပါမယ်။

---

## 7. Node Selector Example Flow

### Step 1: Node ကို label တပ်မယ်

```bash
kubectl label nodes node-1 size=Large
```

### Step 2: Pod YAML ထဲမှာ nodeSelector ထည့်မယ်

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
    - name: data-processor
      image: data-processor
  nodeSelector:
    size: Large
```

### Step 3: Pod ကို create လုပ်မယ်

```bash
kubectl apply -f pod-definition.yaml
```

### Step 4: Pod ဘယ် Node ပေါ်တက်လဲ စစ်မယ်

```bash
kubectl get pods -o wide
```

---

## 8. Node Selector Diagram

```text
Pod
nodeSelector:
  size: Large
        |
        v
Scheduler checks node labels
        |
        v
+---------------+     +---------------+     +---------------+
| Node 1        |     | Node 2        |     | Node 3        |
| size=Small    |     | size=Small    |     | size=Large    |
+---------------+     +---------------+     +---------------+
                                                |
                                                v
                                           Pod runs here
```

---

## 9. Node Selector မကိုက်ရင် ဘာဖြစ်မလဲ

Pod မှာ ဒီလိုရေးထားတယ်ဆိုပါစို့ -

```yaml
nodeSelector:
  size: Large
```

ဒါပေမယ့် cluster ထဲမှာ `size=Large` label ရှိတဲ့ Node မရှိဘူးဆိုရင် Pod က schedule မလုပ်နိုင်ပါဘူး။

Pod status က `Pending` ဖြစ်နေနိုင်ပါတယ်။

စစ်ကြည့်ရန် -

```bash
kubectl get pods
```

ပိုပြီး detail ကြည့်ရန် -

```bash
kubectl describe pod myapp-pod
```

Event ထဲမှာ Node selector ကိုက်တဲ့ Node မရှိကြောင်း error တွေ့နိုင်ပါတယ်။

---

## 10. Node Selector ရဲ့ Limitations

Node Selector က simple scheduling အတွက် အဆင်ပြေပါတယ်။ ဒါပေမယ့် complex condition တွေအတွက်တော့ limitation ရှိပါတယ်။

ဥပမာ ဒီလို condition တွေ Node Selector နဲ့ရေးရခက်ပါတယ်။

### Case 1: Large or Medium node ပေါ် run ချင်တယ်

```text
size=Large OR size=Medium
```

### Case 2: Small မဟုတ်တဲ့ node ပေါ် run ချင်တယ်

```text
size != Small
```

### Case 3: Specific condition အများကြီးနဲ့ schedule လုပ်ချင်တယ်

```text
region=asia AND disk=ssd AND size in (Large, Medium)
```

ဒီလို advanced scheduling rule တွေလိုရင် **Node Affinity** ကို သုံးသင့်ပါတယ်။

---

## 11. Node Selector vs Node Affinity

| Feature | အသုံးပြုမှု |
|---|---|
| Node Selector | Simple key-value match အတွက် သုံးတယ် |
| Node Affinity | Advanced scheduling rules အတွက် သုံးတယ် |

Node Selector က simple ဖြစ်ပြီး exam မှာလည်း အသုံးများပါတယ်။

Node Affinity က ပိုပြီး flexible ဖြစ်ပါတယ်။

---

## 12. အသုံးများတဲ့ Commands

### Node ကို label တပ်ရန်

```bash
kubectl label nodes node-1 size=Large
```

### Node labels တွေကြည့်ရန်

```bash
kubectl get nodes --show-labels
```

### Specific label ရှိတဲ့ Node တွေကြည့်ရန်

```bash
kubectl get nodes -l size=Large
```

### Pod create/apply လုပ်ရန်

```bash
kubectl apply -f pod-definition.yaml
```

### Pod ဘယ် Node ပေါ် run နေလဲ ကြည့်ရန်

```bash
kubectl get pods -o wide
```

### Pod scheduling issue ကြည့်ရန်

```bash
kubectl describe pod myapp-pod
```

### Node label ဖယ်ရန်

```bash
kubectl label nodes node-1 size-
```

---

## 13. CKA Exam အတွက် မှတ်ရန်

CKA exam မှာ Node Selector နဲ့ပတ်သက်ပြီး အောက်ပါအချက်တွေကို သိထားသင့်ပါတယ်။

- Node ကို label တပ်နည်း
- Pod YAML ထဲမှာ `nodeSelector` ထည့်နည်း
- Pod ဘယ် Node ပေါ် run နေလဲ `kubectl get pods -o wide` နဲ့ စစ်နည်း
- Node label မကိုက်ရင် Pod `Pending` ဖြစ်နိုင်တာ
- Simple rule အတွက် Node Selector, complex rule အတွက် Node Affinity သုံးတာ

---

## 14. Simple Summary

Node Selector ဆိုတာ Pod ကို specific label ရှိတဲ့ Node ပေါ်မှာပဲ run စေချင်တဲ့အခါ သုံးတဲ့ method ပါ။

အရင်ဆုံး Node ကို label တပ်ရပါတယ်။

```bash
kubectl label nodes node-1 size=Large
```

ပြီးရင် Pod YAML ထဲမှာ `nodeSelector` ထည့်ပါတယ်။

```yaml
nodeSelector:
  size: Large
```

ဒီလိုရေးထားရင် Pod က `size=Large` label ရှိတဲ့ Node ပေါ်မှာပဲ schedule လုပ်နိုင်ပါတယ်။

အရေးကြီးဆုံး မှတ်ရန် -

```bash
kubectl get nodes --show-labels
kubectl get pods -o wide
```

Node Selector က simple key-value match အတွက်ကောင်းပါတယ်။ Advanced rule တွေလိုရင် Node Affinity ကို သုံးရပါတယ်။
