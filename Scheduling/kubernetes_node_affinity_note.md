# Kubernetes Node Affinity Note

## 1. Node Affinity ဆိုတာဘာလဲ

**Node Affinity** ဆိုတာ Kubernetes မှာ Pod တွေကို specific label ရှိတဲ့ Node တွေပေါ်မှာ schedule လုပ်နိုင်အောင် rule သတ်မှတ်ပေးတဲ့ advanced scheduling method ပါ။

Node Selector နဲ့ ဆင်တူပေမယ့် Node Affinity က ပိုပြီး flexible ဖြစ်ပါတယ်။

Node Selector မှာ simple key-value match ပဲ လုပ်နိုင်ပါတယ်။

```yaml
nodeSelector:
  size: Large
```

Node Affinity မှာတော့ advanced operators တွေ သုံးနိုင်ပါတယ်။

ဥပမာ -

- `In`
- `NotIn`
- `Exists`

ဒီ operators တွေကြောင့် Pod ကို ဘယ် Node ပေါ် run ချင်လဲဆိုတာ ပိုပြီး detail သတ်မှတ်နိုင်ပါတယ်။

---

## 2. Node Selector နဲ့ Node Affinity ကွာခြားချက်

Node Selector example -

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

ဒီ YAML က `size=Large` label ရှိတဲ့ Node ပေါ်မှာပဲ Pod ကို run စေပါတယ်။

ဒါပေမယ့် Node Selector က -

```text
size=Large
```

လို simple condition ပဲ support လုပ်ပါတယ်။

Node Affinity ကတော့ -

```text
size In Large, Medium
size NotIn Small
size Exists
```

လို advanced condition တွေ support လုပ်ပါတယ်။

---

## 3. Node Affinity Basic Example

Pod ကို `size=Large` label ရှိတဲ့ Node ပေါ်မှာ run စေချင်ရင် Node Affinity ကို ဒီလိုရေးနိုင်ပါတယ်။

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
    - name: data-processor
      image: data-processor
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: size
                operator: In
                values:
                  - Large
```

ဒီ YAML မှာ အရေးကြီးတဲ့ structure က -

```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
        - matchExpressions:
            - key: size
              operator: In
              values:
                - Large
```

---

## 4. Node Affinity YAML Structure ရှင်းပြချက်

### `affinity`

```yaml
affinity:
```

Pod spec ထဲမှာ scheduling rule တွေ သတ်မှတ်ဖို့ သုံးပါတယ်။

### `nodeAffinity`

```yaml
nodeAffinity:
```

Node label တွေအပေါ်မူတည်ပြီး Pod placement rule သတ်မှတ်တာပါ။

### `requiredDuringSchedulingIgnoredDuringExecution`

```yaml
requiredDuringSchedulingIgnoredDuringExecution:
```

ဒီ rule က scheduling လုပ်တဲ့အချိန်မှာ မဖြစ်မနေ ကိုက်ညီရမယ်လို့ ဆိုလိုပါတယ်။

အဓိပ္ပါယ်က -

- Pod ကို schedule လုပ်ချိန်မှာ rule ကို မဖြစ်မနေ စစ်မယ်။
- Rule မကိုက်ရင် Pod ကို schedule မလုပ်ဘူး။
- Pod running ဖြစ်ပြီးနောက် Node label ပြောင်းသွားရင်တော့ running Pod ကို မထိခိုက်ဘူး။

### `nodeSelectorTerms`

```yaml
nodeSelectorTerms:
```

Node ကို select လုပ်ဖို့ condition group တွေ ထည့်တဲ့နေရာပါ။

### `matchExpressions`

```yaml
matchExpressions:
```

Label key, operator, values တွေ သတ်မှတ်တဲ့နေရာပါ။

---

## 5. `In` Operator

`In` operator ဆိုတာ label value က သတ်မှတ်ထားတဲ့ values ထဲမှာ ပါရမယ်လို့ ဆိုလိုပါတယ်။

ဥပမာ -

```yaml
key: size
operator: In
values:
  - Large
```

အဓိပ္ပါယ်က -

```text
size label ရှိပြီး value က Large ဖြစ်တဲ့ Node ပေါ်မှာပဲ Pod run ပါ
```

Large သို့မဟုတ် Medium node ပေါ် run ချင်ရင် -

```yaml
key: size
operator: In
values:
  - Large
  - Medium
```

အဓိပ္ပါယ်က -

```text
size=Large OR size=Medium ဖြစ်တဲ့ Node ပေါ် run ပါ
```

---

## 6. `NotIn` Operator

`NotIn` operator ဆိုတာ သတ်မှတ်ထားတဲ့ value မဟုတ်တဲ့ Node တွေပေါ်မှာ run စေချင်တဲ့အခါ သုံးပါတယ်။

ဥပမာ Small node တွေပေါ်မှာ မ run စေချင်ရင် -

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
    - name: data-processor
      image: data-processor
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: size
                operator: NotIn
                values:
                  - Small
```

အဓိပ္ပါယ်က -

```text
size=Small မဟုတ်တဲ့ Node ပေါ်မှာ Pod run ပါ
```

ဒါကြောင့် `size=Large` ဖြစ်ဖြစ် `size=Medium` ဖြစ်ဖြစ် run နိုင်ပါတယ်။

---

## 7. `Exists` Operator

`Exists` operator ဆိုတာ label key ရှိ/မရှိပဲ စစ်ချင်တဲ့အခါ သုံးပါတယ်။

ဒီ operator သုံးတဲ့အခါ `values` မထည့်ရပါဘူး။

ဥပမာ -

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
  - name: data-processor
    image: data-processor
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: size
            operator: Exists
```

အဓိပ္ပါယ်က -

```text
size ဆိုတဲ့ label key ရှိတဲ့ Node ပေါ်မှာ Pod run ပါ
```

ဒီမှာ `size=Large` ဖြစ်ရင်လည်း ရတယ်၊ `size=Medium` ဖြစ်ရင်လည်း ရတယ်။ အဓိကက `size` label ရှိဖို့ပဲ လိုပါတယ်။

---

## 8. Node Affinity Scheduling Behavior ၂ မျိုး

Node Affinity မှာ scheduling behavior ၂ မျိုးရှိပါတယ်။

### 1. Required During Scheduling, Ignored During Execution

```yaml
requiredDuringSchedulingIgnoredDuringExecution
```

ဒီ behavior က hard rule ပါ။

အဓိပ္ပါယ် -

- Scheduling လုပ်တဲ့အချိန်မှာ rule ကို မဖြစ်မနေ ကိုက်ရမယ်။
- Rule မကိုက်ရင် Pod ကို schedule မလုပ်ဘူး။
- Pod running ဖြစ်ပြီးနောက် Node label ပြောင်းသွားရင် Pod ကို မဖယ်ရှားဘူး။

ဥပမာ -

```text
Pod requires size=Large
```

Cluster ထဲမှာ `size=Large` Node မရှိရင် Pod က `Pending` ဖြစ်နေနိုင်ပါတယ်။

### 2. Preferred During Scheduling, Ignored During Execution

```yaml
preferredDuringSchedulingIgnoredDuringExecution
```

ဒီ behavior က soft rule ပါ။

အဓိပ္ပါယ် -

- Scheduler က rule ကိုက်တဲ့ Node ကို ပိုကြိုက်တယ်။
- ဒါပေမယ့် rule ကိုက်တဲ့ Node မရှိရင် အခြား Node ပေါ်မှာလည်း schedule လုပ်နိုင်တယ်။
- Pod running ဖြစ်ပြီးနောက် Node label ပြောင်းသွားရင် Pod ကို မထိခိုက်ဘူး။

![Node Affinity Types](https://kodekloud.com/kk-media/image/upload/v1752869895/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-Node-Affinity/frame_340.jpg)

---

## 9. Ignored During Execution ဆိုတာဘာလဲ

`IgnoredDuringExecution` ဆိုတာ Pod running ဖြစ်ပြီးနောက်မှာ Node label ပြောင်းသွားရင် Pod ကို မထိခိုက်ဘူးလို့ ဆိုလိုပါတယ်။

ဥပမာ -

1. Pod ကို `size=Large` Node ပေါ် schedule လုပ်ထားတယ်။
2. နောက်မှ Node ရဲ့ label ကို `size=Small` လို့ ပြောင်းလိုက်တယ်။
3. Pod က running ဖြစ်နေပြီးသားဆိုရင် ဆက် run နေပါမယ်။
4. Kubernetes က အဲ့ဒီ Pod ကို auto evict မလုပ်ပါဘူး။

ဒါကြောင့် Node Affinity rule က scheduling လုပ်တဲ့အချိန်မှာပဲ အဓိက သက်ရောက်ပါတယ်။

---

## 10. Future Affinity Types

Source ထဲမှာ future enhancement အဖြစ် `Required During Execution` ဆိုတဲ့ concept ကို ဖော်ပြထားပါတယ်။

ဒီ model မှာဆိုရင် Pod running ဖြစ်ပြီးနောက် Node label ပြောင်းသွားပြီး rule မကိုက်တော့ဘူးဆိုရင် Pod ကို evict လုပ်နိုင်ပါတယ်။

ဥပမာ -

```text
Pod requires size=Large
Node label changes from size=Large to size=Small
Pod no longer matches
Pod gets evicted
```

လက်ရှိ lesson မှာတော့ အဓိကသုံးတာက -

- `requiredDuringSchedulingIgnoredDuringExecution`
- `preferredDuringSchedulingIgnoredDuringExecution`

ဖြစ်ပါတယ်။

![Future Node Affinity Types](https://kodekloud.com/kk-media/image/upload/v1752869896/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-Node-Affinity/frame_410.jpg)

---

## 11. Node Affinity vs Node Selector

| Feature | Node Selector | Node Affinity |
|---|---|---|
| Usage | Simple scheduling | Advanced scheduling |
| Operators | မရှိ | `In`, `NotIn`, `Exists` |
| OR condition | မလွယ် | Support လုပ်နိုင် |
| Soft preference | မရှိ | `preferredDuringSchedulingIgnoredDuringExecution` နဲ့ လုပ်နိုင် |
| Complexity | လွယ် | ပိုရှုပ် |

---

## 12. Node Affinity vs Taints and Tolerations

| Feature | အသုံးပြုမှု |
|---|---|
| Node Affinity | Pod ကို specific Node label ပေါ် schedule လုပ်စေချင်တဲ့အခါ |
| Taints and Tolerations | Node ပေါ် မလိုချင်တဲ့ Pod တွေ မလာအောင် တားချင်တဲ့အခါ |

အလွယ်မှတ်ရန် -

```text
Node Affinity = Pod က Node ကို ရွေးချင်တာ
Taints/Tolerations = Node က Pod ကို လက်ခံ/မခံ ဆုံးဖြတ်တာ
```

---

## 13. Node Affinity မကိုက်ရင် ဘာဖြစ်မလဲ

`requiredDuringSchedulingIgnoredDuringExecution` သုံးထားပြီး rule ကိုက်တဲ့ Node မရှိရင် Pod က schedule မလုပ်နိုင်ပါဘူး။

Pod status က `Pending` ဖြစ်နေနိုင်ပါတယ်။

စစ်ကြည့်ရန် -

```bash
kubectl get pods
```

ပို detail ကြည့်ရန် -

```bash
kubectl describe pod myapp-pod
```

Events section ထဲမှာ node affinity/selector မကိုက်တဲ့ message တွေ့နိုင်ပါတယ်။

---

## 14. အသုံးများတဲ့ Commands

### Node ကို label တပ်ရန်

```bash
kubectl label nodes node-1 size=Large
```

### Node labels ကြည့်ရန်

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

---

## 15. CKA Exam အတွက် မှတ်ရန်

CKA exam မှာ Node Affinity က Scheduling topic ထဲမှာ အရေးကြီးပါတယ်။

အဓိက မှတ်ထားရမယ့်အချက်တွေက -

- Node Selector က simple, Node Affinity က advanced
- `affinity.nodeAffinity` YAML structure
- `requiredDuringSchedulingIgnoredDuringExecution`
- `preferredDuringSchedulingIgnoredDuringExecution`
- `In`, `NotIn`, `Exists` operators
- `Exists` သုံးရင် `values` မထည့်ရ
- Required rule မကိုက်ရင် Pod `Pending` ဖြစ်နိုင်တယ်
- Running ဖြစ်ပြီးနောက် label ပြောင်းသွားရင် `IgnoredDuringExecution` ကြောင့် Pod ကို မဖယ်ရှားဘူး

---

## 16. Simple Summary

Node Affinity ဆိုတာ Pod ကို Node labels အပေါ်မူတည်ပြီး advanced scheduling rule နဲ့ run စေချင်တဲ့အခါ သုံးတာပါ။

Node Selector ထက် ပို flexible ဖြစ်ပြီး operators တွေ သုံးနိုင်ပါတယ်။

```yaml
operator: In
```

ဆိုရင် သတ်မှတ်ထားတဲ့ values ထဲက value ကိုက်တဲ့ Node ပေါ် run ပါ။

```yaml
operator: NotIn
```

ဆိုရင် သတ်မှတ်ထားတဲ့ values မဟုတ်တဲ့ Node ပေါ် run ပါ။

```yaml
operator: Exists
```

ဆိုရင် label key ရှိရုံနဲ့ run ပါ။

အရေးကြီးဆုံး YAML structure -

```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
        - matchExpressions:
            - key: size
              operator: In
              values:
                - Large
```

Node Affinity က Pod placement ကို ပိုပြီး detail control လုပ်ချင်တဲ့အခါ အသုံးပြုရတဲ့ Kubernetes scheduling feature ဖြစ်ပါတယ်။
