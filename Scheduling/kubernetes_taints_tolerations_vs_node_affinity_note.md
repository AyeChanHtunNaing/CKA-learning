# Kubernetes Taints and Tolerations vs Node Affinity Note

## 1. Overview

ဒီ lesson မှာ Kubernetes cluster ထဲက Pod တွေကို မှန်ကန်တဲ့ Node ပေါ်မှာ schedule လုပ်နိုင်အောင် **Taints and Tolerations** နဲ့ **Node Affinity** ကို ဘယ်လိုပေါင်းသုံးရမလဲဆိုတာ ရှင်းပြထားပါတယ်။

ဥပမာအနေနဲ့ Node ၃ ခုနဲ့ Pod ၃ ခုရှိတယ်ဆိုပါစို့။

- Blue Node / Blue Pod
- Red Node / Red Pod
- Green Node / Green Pod

ရည်ရွယ်ချက်က -

```text
Blue Pod  -> Blue Node
Red Pod   -> Red Node
Green Pod -> Green Node
```

လိုမျိုး Pod တစ်ခုချင်းစီကို ကိုက်ညီတဲ့ color Node ပေါ်မှာပဲ run စေချင်တာပါ။

---

## 2. အဓိက Concept

Kubernetes မှာ Pod placement ကို control လုပ်ဖို့ အဓိက method ၂ ခုရှိပါတယ်။

| Method | အဓိကအလုပ် |
|---|---|
| Taints and Tolerations | မလိုချင်တဲ့ Pod တွေ Node ပေါ် မလာအောင် repel လုပ်တယ် |
| Node Affinity | Pod ကို label ကိုက်တဲ့ Node ပေါ် သွားအောင် attract လုပ်တယ် |

အလွယ်မှတ်ရန် -

```text
Taints/Tolerations = Node က Pod ကို လက်ခံ/မခံ ဆုံးဖြတ်တာ
Node Affinity      = Pod က ဘယ် Node ကို သွားချင်လဲ ဆုံးဖြတ်တာ
```

---

## 3. Taints and Tolerations ကို သုံးခြင်း

ပထမနည်းလမ်းက Node တစ်ခုချင်းစီကို color အလိုက် taint တပ်တာပါ။

ဥပမာ -

```bash
kubectl taint nodes node-blue color=blue:NoSchedule
kubectl taint nodes node-red color=red:NoSchedule
kubectl taint nodes node-green color=green:NoSchedule
```

ပြီးရင် Pod တစ်ခုချင်းစီမှာ matching toleration ထည့်ပေးရပါတယ်။

ဥပမာ Blue Pod မှာ -

```yaml
tolerations:
  - key: "color"
    operator: "Equal"
    value: "blue"
    effect: "NoSchedule"
```

ဒီလိုလုပ်ရင် Blue Pod က `color=blue:NoSchedule` taint ရှိတဲ့ Node ပေါ်မှာ run ခွင့်ရနိုင်ပါတယ်။

![Taints and Tolerations Example](https://kodekloud.com/kk-media/image/upload/v1752869912/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-Taints-and-Tolerations-vs-Node-Affinity/frame_50.jpg)

---

## 4. Taints and Tolerations ရဲ့ Limitation

Taints and Tolerations က Node ပေါ် မလိုချင်တဲ့ Pod တွေ မလာအောင် တားရာမှာကောင်းပါတယ်။

ဒါပေမယ့် **Pod ကို သတ်မှတ်ထားတဲ့ Node ပေါ်မှာ မဖြစ်မနေ run စေမယ်** လို့ guarantee မပေးနိုင်ပါဘူး။

ဥပမာ -

- Red Pod မှာ red toleration ရှိတယ်။
- Red Node က red taint ရှိတယ်။
- ဒါပေမယ့် Red Pod က untainted node ပေါ်မှာလည်း schedule ဖြစ်သွားနိုင်ပါတယ်။

အကြောင်းက toleration ဆိုတာ taint ရှိတဲ့ Node ပေါ်မှာ run ခွင့်ပေးတာပဲ ဖြစ်ပြီး အဲ့ဒီ Node ပေါ်ကို မဖြစ်မနေသွားပါလို့ မဆိုလိုပါဘူး။

အလွယ်ပြောရရင် -

```text
Toleration ရှိတယ် = အဲ့ဒီ Node ပေါ် run လို့ရတယ်
Toleration ရှိတယ် ≠ အဲ့ဒီ Node ပေါ်မှာပဲ run မယ်
```

---

## 5. Node Affinity ကို သုံးခြင်း

Taints and Tolerations ရဲ့ limitation ကို ဖြေရှင်းဖို့ Node Affinity ကို သုံးနိုင်ပါတယ်။

Node Affinity သုံးဖို့ အရင်ဆုံး Node တွေကို label တပ်ရပါတယ်။

ဥပမာ -

```bash
kubectl label nodes node-blue color=blue
kubectl label nodes node-red color=red
kubectl label nodes node-green color=green
```

ပြီးရင် Pod တွေမှာ Node Affinity rule ထည့်ပေးရပါတယ်။

ဥပမာ Blue Pod ကို Blue Node ပေါ်မှာပဲ run စေချင်ရင် -

```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
        - matchExpressions:
            - key: color
              operator: In
              values:
                - blue
```

ဒီ rule ရဲ့ အဓိပ္ပါယ်က -

```text
color=blue label ရှိတဲ့ Node ပေါ်မှာပဲ ဒီ Pod ကို schedule လုပ်ပါ
```

---

## 6. Node Affinity ရဲ့ Limitation

Node Affinity က Pod ကို မှန်ကန်တဲ့ Node ပေါ်သွားအောင် attract လုပ်နိုင်ပါတယ်။

ဒါပေမယ့် Node Affinity တစ်ခုတည်းသုံးရင် **အခြား Pod တွေ အဲ့ဒီ Node ပေါ် မလာအောင် မတားနိုင်ပါဘူး**။

ဥပမာ -

- Blue Pod ကို Blue Node ပေါ်မှာ run စေဖို့ Node Affinity ထည့်ထားတယ်။
- Blue Pod က Blue Node ပေါ်မှာ run ဖြစ်မယ်။
- ဒါပေမယ့် အခြား unrelated Pod တွေလည်း Blue Node ပေါ် schedule ဖြစ်နိုင်ပါတယ်။

အလွယ်ပြောရရင် -

```text
Node Affinity = Pod ကို မှန်တဲ့ Node ပေါ် သွားစေတယ်
Node Affinity ≠ အခြား Pod တွေ မလာအောင် တားတယ်
```

---

## 7. နှစ်ခုလုံးပေါင်းသုံးခြင်း

Dedicated node usage လိုချင်ရင် **Taints/Tolerations** နဲ့ **Node Affinity** ကို ပေါင်းသုံးရပါတယ်။

ပေါင်းသုံးတဲ့အခါ -

1. Node ပေါ်မှာ taint တပ်မယ်။
2. Intended Pod မှာ matching toleration ထည့်မယ်။
3. Node ပေါ်မှာ label တပ်မယ်။
4. Pod မှာ matching node affinity rule ထည့်မယ်။

ဒီလိုလုပ်ရင် -

- Node Affinity က Pod ကို မှန်တဲ့ Node ပေါ်သွားစေမယ်။
- Taints/Tolerations က အခြား Pod တွေ အဲ့ဒီ Node ပေါ် မလာအောင် တားမယ်။

![Taints/Tolerations and Node Affinity Combined](https://kodekloud.com/kk-media/image/upload/v1752869913/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-Taints-and-Tolerations-vs-Node-Affinity/frame_130.jpg)

---

## 8. Combined Example: Blue Pod and Blue Node

### Step 1: Blue Node ကို label တပ်မယ်

```bash
kubectl label nodes node-blue color=blue
```

### Step 2: Blue Node ကို taint တပ်မယ်

```bash
kubectl taint nodes node-blue color=blue:NoSchedule
```

### Step 3: Blue Pod YAML မှာ toleration နဲ့ node affinity ထည့်မယ်

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: blue-pod
spec:
  containers:
    - name: nginx
      image: nginx

  tolerations:
    - key: "color"
      operator: "Equal"
      value: "blue"
      effect: "NoSchedule"

  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: color
                operator: In
                values:
                  - blue
```

ဒီ YAML မှာ -

```yaml
tolerations:
```

က Blue Pod ကို Blue taint ရှိတဲ့ Node ပေါ် run ခွင့်ပေးပါတယ်။

```yaml
affinity:
```

က Blue Pod ကို `color=blue` label ရှိတဲ့ Node ပေါ်မှာပဲ schedule လုပ်စေပါတယ်။

---

## 9. Taints/Tolerations vs Node Affinity Comparison

| Feature | Taints and Tolerations | Node Affinity |
|---|---|---|
| အလုပ်လုပ်ပုံ | Node က Pod ကို repel/accept လုပ်တယ် | Pod က label ကိုက်တဲ့ Node ကို ရွေးတယ် |
| Main Purpose | မလိုချင်တဲ့ Pod မလာအောင် တားရန် | Pod ကို မှန်တဲ့ Node ပေါ် schedule လုပ်ရန် |
| Guarantee correct node? | မပေးနိုင် | ပေးနိုင် |
| Prevent other pods? | လုပ်နိုင် | မလုပ်နိုင် |
| Best Use | Dedicated nodes ကို protect လုပ်ရန် | Pod placement ကို control လုပ်ရန် |

---

## 10. ဘယ်အချိန်မှာ ဘာသုံးမလဲ

### Taints and Tolerations ပဲ သုံးမယ်

Node တစ်ခုကို general workloads မလာအောင် တားချင်ရင် သုံးနိုင်ပါတယ်။

ဥပမာ -

```text
GPU node ပေါ်မှာ GPU workload တွေပဲ run စေချင်တယ်
```

### Node Affinity ပဲ သုံးမယ်

Pod ကို label ကိုက်တဲ့ Node ပေါ်မှာ run စေချင်ပေမယ့် အခြား Pod တွေလည်း run လို့ရတယ်ဆိုရင် သုံးနိုင်ပါတယ်။

ဥပမာ -

```text
High memory Pod ကို memory=high label ရှိတဲ့ Node ပေါ် run စေချင်တယ်
```

### နှစ်ခုလုံး ပေါင်းသုံးမယ်

Node ကို specific workloads အတွက် exclusive/dedicated ထားချင်ရင် ပေါင်းသုံးရပါတယ်။

ဥပမာ -

```text
Blue app Pod တွေပဲ Blue Node ပေါ် run စေချင်တယ်
အခြား Pod တွေ Blue Node ပေါ် မလာစေချင်ဘူး
```

---

## 11. CKA Exam အတွက် မှတ်ရန်

CKA exam မှာ ဒီ topic က scheduling concept တွေကို နားလည်ထားဖို့ အရေးကြီးပါတယ်။

အဓိက မှတ်ရန် -

- Taints/Tolerations က Pod တွေကို repel/allow လုပ်တယ်။
- Node Affinity က Pod ကို matching label ရှိတဲ့ Node ပေါ် schedule လုပ်စေတယ်။
- Toleration ရှိတာက node ပေါ် run ခွင့်ရတာပဲ၊ node ပေါ်မှာ မဖြစ်မနေ run မယ်လို့ မဆိုလိုဘူး။
- Node Affinity တစ်ခုတည်းက အခြား Pod တွေ node ပေါ် မလာအောင် မတားနိုင်ဘူး။
- Exclusive node usage လိုချင်ရင် နှစ်ခုလုံးပေါင်းသုံးရမယ်။

---

## 12. Simple Summary

**Taints and Tolerations** က Node ကို protect လုပ်ဖို့ သုံးပါတယ်။

```text
မကိုက်တဲ့ Pod တွေ Node ပေါ် မလာအောင် တားတယ်
```

**Node Affinity** က Pod ကို မှန်တဲ့ Node ပေါ် သွားစေဖို့ သုံးပါတယ်။

```text
Pod ကို label ကိုက်တဲ့ Node ပေါ် schedule လုပ်စေတယ်
```

နှစ်ခုလုံးပေါင်းသုံးရင် -

```text
Node Affinity      -> Pod ကို မှန်တဲ့ Node ပေါ် ပို့တယ်
Taints/Toleration  -> အခြား Pod တွေ မလာအောင် တားတယ်
```

အရေးကြီးဆုံး မှတ်ရန် -

```text
Taints/Tolerations only = Pod placement ကို guarantee မပေးနိုင်
Node Affinity only      = အခြား Pod တွေကို မတားနိုင်
Both together           = Correct placement + Exclusive usage
```

ဒါကြောင့် dedicated node တွေကို correct pod တွေအတွက်ပဲ သီးသန့်သုံးချင်ရင် **Taints/Tolerations + Node Affinity** ကို ပေါင်းသုံးရပါတယ်။
