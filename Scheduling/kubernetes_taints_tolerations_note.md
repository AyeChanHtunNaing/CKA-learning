# Kubernetes Taints and Tolerations

## 1. Taints and Tolerations ဆိုတာဘာလဲ

**Taints** နဲ့ **Tolerations** ဆိုတာ Kubernetes မှာ Pod တွေကို Node တစ်ခုပေါ်မှာ schedule လုပ်ခွင့်ရှိ/မရှိ ထိန်းချုပ်ဖို့ သုံးတဲ့ mechanism ပါ။

ရိုးရိုးဥပမာနဲ့ပြောရရင် -

- **Taint** = Node ပေါ်မှာ တပ်ထားတဲ့ repellent
- **Toleration** = Pod မှာ ရှိတဲ့ အဲ့ဒီ repellent ကို ခံနိုင်တဲ့ permission

Node တစ်ခုမှာ taint တပ်ထားရင် အဲ့ဒီ taint ကို tolerate မလုပ်နိုင်တဲ့ Pod တွေက အဲ့ဒီ Node ပေါ်မှာ run လို့မရပါဘူး။

![Taints and Tolerations Concept](https://kodekloud.com/kk-media/image/upload/v1752869914/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-Taints-and-Tolerations/frame_60.jpg)

---

## 2. Taints and Tolerations ကို ဘာကြောင့်သုံးလဲ

Kubernetes cluster ထဲမှာ Node တွေ အများကြီးရှိနိုင်ပါတယ်။ တချို့ Node တွေကို specific workload အတွက်ပဲ သီးသန့်ထားချင်တဲ့အခါ Taints and Tolerations ကို သုံးပါတယ်။

ဥပမာ -

- Node 1 ကို `blue` application အတွက်ပဲ သုံးချင်တယ်။
- အခြား Pod တွေ Node 1 ပေါ် မလာစေချင်ဘူး။
- ဒါကြောင့် Node 1 ကို taint တပ်မယ်။
- Blue application Pod မှာတော့ matching toleration ထည့်မယ်။

ဒီလိုလုပ်ရင် Blue app Pod ပဲ Node 1 ပေါ်မှာ run လို့ရနိုင်ပါတယ်။

---

## 3. Taints and Tolerations အလုပ်လုပ်ပုံ

Pod တစ်ခု Node တစ်ခုပေါ်မှာ run လို့ရမရကို အဓိကအချက် ၂ ခုပေါ်မူတည်ပါတယ်။

1. Node ပေါ်မှာ တပ်ထားတဲ့ **Taint**
2. Pod မှာ ရှိတဲ့ **Toleration**

Node မှာ taint ရှိပြီး Pod မှာ matching toleration မရှိရင် Scheduler က အဲ့ဒီ Pod ကို အဲ့ဒီ Node ပေါ် မတင်ပါဘူး။

![Taints and Tolerations Scheduling Example](https://kodekloud.com/kk-media/image/upload/v1752869915/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-Taints-and-Tolerations/frame_240.jpg)

---

## 4. Taint ကို Node မှာ တပ်နည်း

Node တစ်ခုကို taint တပ်ချင်ရင် အောက်က command ကို သုံးပါတယ်။

```bash
kubectl taint nodes node-name key=value:taint-effect
```

Format က -

```bash
kubectl taint nodes <node-name> <key>=<value>:<effect>
```

ဥပမာ `node1` ကို `app=blue` ဆိုတဲ့ taint နဲ့ `NoSchedule` effect တပ်ချင်ရင် -

```bash
kubectl taint nodes node1 app=blue:NoSchedule
```

ဒီ command ရဲ့ အဓိပ္ပါယ်က -

`node1` ပေါ်မှာ `app=blue` taint တပ်ထားမယ်။ ဒီ taint ကို tolerate မလုပ်နိုင်တဲ့ Pod တွေကို `node1` ပေါ် schedule မလုပ်ပါနဲ့။

---

## 5. Taint Effects အမျိုးအစား ၃ မျိုး

Kubernetes taint မှာ effect ၃ မျိုးရှိပါတယ်။

| Effect             | အဓိပ္ပါယ်                                                                                                         |
| ------------------ | ----------------------------------------------------------------------------------------------------------------- |
| `NoSchedule`       | Matching toleration မရှိတဲ့ Pod အသစ်တွေကို ဒီ Node ပေါ် schedule မလုပ်ဘူး                                         |
| `PreferNoSchedule` | ဖြစ်နိုင်ရင် ဒီ Node ပေါ် မတင်ဘူး၊ ဒါပေမယ့် strictly မတားဘူး                                                      |
| `NoExecute`        | Matching toleration မရှိတဲ့ Pod အသစ်တွေကို schedule မလုပ်ဘူး၊ ရှိပြီးသား Pod တွေကိုလည်း evict/remove လုပ်နိုင်တယ် |

---

## 6. NoSchedule Effect

`NoSchedule` ဆိုတာ taint ကို tolerate မလုပ်နိုင်တဲ့ Pod တွေကို Node ပေါ်မှာ အသစ် schedule မလုပ်စေတဲ့ effect ပါ။

ဥပမာ -

```bash
kubectl taint nodes node1 app=blue:NoSchedule
```

ဒီ command ပြီးရင် `node1` ပေါ်မှာ `app=blue` ကို tolerate မလုပ်နိုင်တဲ့ Pod တွေ run မလာနိုင်ပါဘူး။

ဒါပေမယ့် အရင်ကတည်းက `node1` ပေါ်မှာ run နေတဲ့ Pod တွေကိုတော့ remove မလုပ်ပါဘူး။

---

## 7. PreferNoSchedule Effect

`PreferNoSchedule` ကတော့ soft rule ပါ။

Scheduler က taint ကို tolerate မလုပ်နိုင်တဲ့ Pod တွေကို အဲ့ဒီ Node ပေါ် မတင်ဖို့ ကြိုးစားပါမယ်။ ဒါပေမယ့် cluster condition အရ လိုအပ်ရင် တင်နိုင်ပါတယ်။

ဥပမာ -

```bash
kubectl taint nodes node1 app=blue:PreferNoSchedule
```

အဓိပ္ပါယ်က -

ဖြစ်နိုင်ရင် `node1` ပေါ် Pod မတင်ပါနဲ့။ ဒါပေမယ့် မဖြစ်မနေလိုရင် တင်နိုင်ပါတယ်။

---

## 8. NoExecute Effect

`NoExecute` က effect အပြင်းဆုံးပါ။

ဒီ effect ကို Node မှာ taint တပ်လိုက်ရင် -

- Matching toleration မရှိတဲ့ Pod အသစ်တွေကို schedule မလုပ်ပါဘူး။
- အဲ့ဒီ Node ပေါ်မှာ ရှိပြီးသား matching toleration မရှိတဲ့ Pod တွေကိုလည်း evict လုပ်နိုင်ပါတယ်။

ဥပမာ -

```bash
kubectl taint nodes node1 app=blue:NoExecute
```

ဒီ command ပြီးရင် `app=blue` toleration မရှိတဲ့ Pod တွေက `node1` ပေါ်မှာ ဆက် run မလုပ်နိုင်တော့ပါဘူး။

![NoExecute Taint Example](https://kodekloud.com/kk-media/image/upload/v1752869916/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-Taints-and-Tolerations/frame_420.jpg)

---

## 9. Pod မှာ Toleration ထည့်နည်း

Tainted node ပေါ်မှာ Pod တစ်ခု run စေချင်ရင် Pod manifest ထဲမှာ `tolerations` ထည့်ပေးရပါတယ်။

ဥပမာ `node1` မှာ ဒီ taint ရှိတယ်ဆိုပါစို့ -

```bash
app=blue:NoSchedule
```

အဲ့ဒီ taint ကို tolerate လုပ်နိုင်တဲ့ Pod manifest က ဒီလိုရေးရပါတယ်။

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
    - name: nginx-container
      image: nginx
  tolerations:
    - key: "app"
      operator: "Equal"
      value: "blue"
      effect: "NoSchedule"
```

ဒီ Pod မှာ -

```yaml
key: "app"
value: "blue"
effect: "NoSchedule"
```

ဆိုပြီး Node ရဲ့ taint နဲ့ ကိုက်ညီတဲ့ toleration ထည့်ထားပါတယ်။

ဒါကြောင့် ဒီ Pod က `app=blue:NoSchedule` taint ရှိတဲ့ Node ပေါ်မှာ run လို့ရပါတယ်။

---

## 10. Taint နဲ့ Toleration ကိုက်ညီရမယ့်အချက်များ

Node taint က -

```bash
app=blue:NoSchedule
```

ဖြစ်ရင် Pod toleration မှာလည်း -

```yaml
key: "app"
operator: "Equal"
value: "blue"
effect: "NoSchedule"
```

ဖြစ်ရပါမယ်။

ကိုက်ညီရမယ့်အချက်တွေက -

- Key ကိုက်ရမယ်
- Value ကိုက်ရမယ်
- Effect ကိုက်ရမယ်
- Operator မှန်ရမယ်

---

## 11. Taints and Tolerations သုံးတာ Security မဟုတ်ပါ

Taints and Tolerations က **Security feature မဟုတ်ပါဘူး**။

ဒါက scheduling control အတွက်ပဲ သုံးတာပါ။

ဥပမာ Node တစ်ခုကို specific Pod တွေ run ဖို့ သီးသန့်ထားချင်ရင် taint/toleration သုံးနိုင်ပါတယ်။ ဒါပေမယ့် Pod တစ်ခုကို Node တစ်ခုပေါ်မှာ မဖြစ်မနေ run စေချင်ရင်တော့ **Node Affinity** ကို သုံးသင့်ပါတယ်။

---

## 12. Taints and Tolerations vs Node Affinity

Taints and Tolerations က Pod ကို Node တစ်ခုပေါ် မလာအောင် တားတဲ့အပိုင်းမှာ အသုံးများပါတယ်။

Node Affinity ကတော့ Pod ကို သတ်မှတ်ထားတဲ့ Node ပေါ်မှာ run စေချင်တဲ့အခါ အသုံးများပါတယ်။

| Feature                | အဓိကအသုံးပြုမှု                                   |
| ---------------------- | ------------------------------------------------- |
| Taints and Tolerations | Node ပေါ် Pod မလာအောင် repel/tolerate လုပ်ဖို့    |
| Node Affinity          | Pod ကို specific Node ပေါ်မှာ schedule လုပ်စေဖို့ |

အရေးကြီးတာက -

Toleration ရှိတယ်ဆိုတာ Pod ကို အဲ့ဒီ Node ပေါ်မှာ မဖြစ်မနေ run စေမယ်လို့ မဆိုလိုပါဘူး။ အဲ့ဒီ Node ပေါ် run လို့ရခွင့်ပေးတာပဲ ဖြစ်ပါတယ်။

---

## 13. Master Node Taints

Kubernetes မှာ master/control plane node တွေက workload Pod တွေကို ပုံမှန်အားဖြင့် run မစေချင်ပါဘူး။

အကြောင်းက master node တွေက cluster ကို control လုပ်တဲ့ important system components တွေ run နေတဲ့နေရာဖြစ်လို့ပါ။

ဒါကြောင့် Kubernetes က master node ပေါ်မှာ default taint တပ်ထားပါတယ်။

စစ်ကြည့်ချင်ရင် -

```bash
kubectl describe node kubemaster | grep Taint
```

Output example -

```text
Taints:             node-role.kubernetes.io/master:NoSchedule
```

ဒါက workload Pod တွေ master node ပေါ် မတက်အောင် တားထားတာပါ။

အသစ် Kubernetes version တွေမှာ control-plane node taint က ဒီလိုလည်း ဖြစ်နိုင်ပါတယ်။

```text
node-role.kubernetes.io/control-plane:NoSchedule
```

---

## 14. Taint ဖယ်ရှားနည်း

Node မှာ တပ်ထားတဲ့ taint ကို ဖယ်ချင်ရင် taint command ရဲ့နောက်မှာ `-` ထည့်ပေးရပါတယ်။

ဥပမာ -

```bash
kubectl taint nodes node1 app=blue:NoSchedule-
```

ဒီ command က `node1` ပေါ်က `app=blue:NoSchedule` taint ကို ဖယ်ရှားပေးပါတယ်။

---

## 15. အသုံးများတဲ့ Commands

### Node ကို taint တပ်ရန်

```bash
kubectl taint nodes node1 app=blue:NoSchedule
```

### Node ရဲ့ taints ကို ကြည့်ရန်

```bash
kubectl describe node node1 | grep Taint
```

### All nodes ကို ကြည့်ရန်

```bash
kubectl get nodes
```

### Pod ဘယ် Node ပေါ် run နေလဲ ကြည့်ရန်

```bash
kubectl get pods -o wide
```

### Taint ဖယ်ရန်

```bash
kubectl taint nodes node1 app=blue:NoSchedule-
```

---

## 16. CKA Exam အတွက် မှတ်ရန်

CKA exam မှာ Taints and Tolerations က အရေးကြီးတဲ့ scheduling topic တစ်ခုပါ။

အဓိက မှတ်ထားရမယ့်အချက်တွေက -

- Node မှာ taint တပ်နည်း
- Pod မှာ toleration ထည့်နည်း
- `NoSchedule`, `PreferNoSchedule`, `NoExecute` effect တွေကွာခြားချက်
- Master/control-plane node တွေမှာ default taint ရှိတတ်တာ
- Taint ဖယ်ရှားနည်း
- `kubectl describe node <node-name> | grep Taint`
- `kubectl get pods -o wide`

---

## 17. Simple Summary

Taint ဆိုတာ Node ပေါ်မှာ တပ်ထားတဲ့ rule ဖြစ်ပြီး Pod တွေကို repel လုပ်ဖို့ သုံးပါတယ်။

```bash
kubectl taint nodes node1 app=blue:NoSchedule
```

Toleration ဆိုတာ Pod မှာ ထည့်ထားတဲ့ permission ဖြစ်ပြီး tainted node ပေါ်မှာ run ခွင့်ပေးပါတယ်။

```yaml
tolerations:
  - key: "app"
    operator: "Equal"
    value: "blue"
    effect: "NoSchedule"
```

`NoSchedule` က matching toleration မရှိတဲ့ Pod အသစ်တွေကို schedule မလုပ်ပါဘူး။

`PreferNoSchedule` က ဖြစ်နိုင်ရင် မတင်ပါဘူး၊ ဒါပေမယ့် strict မဟုတ်ပါဘူး။

`NoExecute` က matching toleration မရှိတဲ့ Pod တွေကို schedule မလုပ်သလို ရှိပြီးသား Pod တွေကိုလည်း evict လုပ်နိုင်ပါတယ်။

Taints and Tolerations က Pod placement ကို influence လုပ်တာပဲ ဖြစ်ပြီး specific Node ပေါ် မဖြစ်မနေ run စေချင်ရင် Node Affinity ကို သုံးရပါမယ်။
