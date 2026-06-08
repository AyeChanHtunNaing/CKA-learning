# Kubernetes Labels, Selectors and Annotations

## 1. Labels and Selectors ဆိုတာဘာလဲ

**Labels** နဲ့ **Selectors** ဆိုတာ Kubernetes object တွေကို စနစ်တကျ ခွဲခြား၊ စုစည်း၊ ရှာဖွေဖို့ သုံးတဲ့ concept တွေပါ။

ဥပမာ တိရစ္ဆာန်တွေကို ခွဲကြည့်မယ်ဆိုရင် -

- `class: bird`
- `kind: parrot`
- `color: green`

လို label တွေတပ်ထားနိုင်ပါတယ်။

နောက်မှ **green animals** တွေကိုပဲ ရှာချင်ရင် selector နဲ့ filter လုပ်နိုင်ပါတယ်။

```bash
color=green
```

Green bird တွေကိုပဲ ရှာချင်ရင် -

```bash
color=green,kind=bird
```

လိုမျိုး filter လုပ်နိုင်ပါတယ်။

![Labels and Selectors Animal Example](https://kodekloud.com/kk-media/image/upload/v1752869891/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-Labels-and-Selectors/frame_70.jpg)

---

## 2. Kubernetes မှာ Labels and Selectors ကို ဘာကြောင့်သုံးလဲ

Kubernetes cluster ထဲမှာ object တွေ အများကြီးရှိနိုင်ပါတယ်။

ဥပမာ -

- Pods
- Services
- ReplicaSets
- Deployments

အဲ့ဒီ object တွေကို application, function, environment, version စတာတွေအလိုက် ခွဲခြားစီမံဖို့ Labels နဲ့ Selectors ကို သုံးပါတယ်။

ဥပမာ -

```yaml
app: App1
function: Front-end
```

ဒီလို label တပ်ထားရင် နောက်ပိုင်းမှာ `app=App1` ဆိုပြီး App1 နဲ့သက်ဆိုင်တဲ့ object တွေကို filter လုပ်နိုင်ပါတယ်။

![Kubernetes Labels and Selectors Example](https://kodekloud.com/kk-media/image/upload/v1752869893/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-Labels-and-Selectors/frame_140.jpg)

![Application and Function Labels](https://kodekloud.com/kk-media/image/upload/v1752869894/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-Labels-and-Selectors/frame_160.jpg)

---

## 3. Pod မှာ Label ထည့်နည်း

Pod YAML file ထဲမှာ label ထည့်ချင်ရင် `metadata.labels` အောက်မှာ ထည့်ရပါတယ်။

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
  labels:
    app: App1
    function: Front-end
spec:
  containers:
    - name: simple-webapp
      image: simple-webapp
      ports:
        - containerPort: 8080
```

ဒီ Pod မှာ label ၂ ခုရှိပါတယ်။

```yaml
app: App1
function: Front-end
```

---

## 4. Selector နဲ့ Pod ရှာနည်း

Label တပ်ထားတဲ့ Pod တွေကို selector နဲ့ ရှာနိုင်ပါတယ်။

```bash
kubectl get pods --selector app=App1
```

သို့မဟုတ် short form အနေနဲ့ -

```bash
kubectl get pods -l app=App1
```

Output example -

```bash
NAME            READY   STATUS      RESTARTS   AGE
simple-webapp   0/1     Completed   0          1d
```

Label တစ်ခုထက်ပိုပြီး filter လုပ်ချင်ရင် -

```bash
kubectl get pods -l app=App1,function=Front-end
```

---

## 5. ReplicaSet မှာ Labels and Selectors သုံးနည်း

ReplicaSet က Pod တွေကို manage လုပ်ဖို့ label နဲ့ selector ကို အသုံးပြုပါတယ်။

ReplicaSet မှာ label ထည့်တဲ့နေရာ ၂ ခုရှိပါတယ်။

### 1. ReplicaSet ကိုယ်တိုင်အတွက် label

```yaml
metadata:
  labels:
    app: App1
    function: Front-end
```

ဒါက ReplicaSet object ကို label တပ်တာပါ။

### 2. Pod template အတွက် label

```yaml
template:
  metadata:
    labels:
      app: App1
      function: Front-end
```

ဒါက ReplicaSet က create လုပ်မယ့် Pod တွေမှာ label တပ်တာပါ။

---

## 6. ReplicaSet Example

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: simple-webapp
  labels:
    app: App1
    function: Front-end
spec:
  replicas: 3
  selector:
    matchLabels:
      app: App1
  template:
    metadata:
      labels:
        app: App1
        function: Front-end
    spec:
      containers:
        - name: simple-webapp
          image: simple-webapp
```

ဒီ YAML မှာ အရေးကြီးဆုံးက -

```yaml
selector:
  matchLabels:
    app: App1
```

ဖြစ်ပါတယ်။

ReplicaSet က `app: App1` label ပါတဲ့ Pod တွေကို ရှာပြီး manage လုပ်ပါတယ်။

Pod template မှာလည်း -

```yaml
labels:
  app: App1
```

ရှိရပါမယ်။

အဲ့ဒီ label မကိုက်ရင် ReplicaSet က Pod တွေကို မှန်မှန်ကန်ကန် manage မလုပ်နိုင်ပါဘူး။

---

## 7. ReplicaSet မှာ Selector နဲ့ Pod Label ကိုက်ရမယ်

အောက်က selector နဲ့ pod label ကိုက်ရပါမယ်။

```yaml
selector:
  matchLabels:
    app: App1
```

```yaml
template:
  metadata:
    labels:
      app: App1
```

ဒါကြောင့် ReplicaSet ရေးတဲ့အခါ -

- `spec.selector.matchLabels`
- `spec.template.metadata.labels`

နှစ်ခု ကိုက်ညီဖို့ အရေးကြီးပါတယ်။

---

## 8. Service မှာ Labels and Selectors သုံးနည်း

Service ကလည်း Pod တွေကို select လုပ်ဖို့ selector ကို သုံးပါတယ်။

ဥပမာ -

```yaml
selector:
  app: App1
```

ဆိုရင် Service က `app: App1` label ပါတဲ့ Pod တွေဆီ traffic ပို့ပေးပါတယ်။

ဒါကြောင့် Service နဲ့ Pod ချိတ်ဖို့ label/selector ကိုက်ညီရပါမယ်။

---

## 9. Annotations ဆိုတာဘာလဲ

**Annotations** ဆိုတာ Labels နဲ့ မတူပါဘူး။

Labels က object တွေကို select/filter လုပ်ဖို့ သုံးပါတယ်။

Annotations ကတော့ extra information သိမ်းဖို့ သုံးပါတယ်။

ဥပမာ -

- Build version
- Tool version
- Contact information
- Description
- Release note
- Git commit ID

စတာတွေကို သိမ်းထားနိုင်ပါတယ်။

Annotations ကို selector အနေနဲ့ မသုံးပါဘူး။

---

## 10. Annotation Example

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: simple-webapp
  labels:
    app: App1
    function: Front-end
  annotations:
    buildversion: "1.34"
spec:
  replicas: 3
  selector:
    matchLabels:
      app: App1
  template:
    metadata:
      labels:
        app: App1
        function: Front-end
    spec:
      containers:
        - name: simple-webapp
          image: simple-webapp
```

ဒီမှာ annotation က -

```yaml
annotations:
  buildversion: "1.34"
```

ဖြစ်ပါတယ်။

ဒါက ReplicaSet ရဲ့ build version ကို သိမ်းထားတာပါ။ Pod selection အတွက် မသုံးပါဘူး။

---

## 11. Labels vs Selectors vs Annotations

| Concept    | အသုံးပြုမှု                         | Example                |
| ---------- | ----------------------------------- | ---------------------- |
| Label      | Object ကို category ခွဲဖို့         | `app: App1`            |
| Selector   | Label အပေါ်မူတည်ပြီး object ရှာဖို့ | `app=App1`             |
| Annotation | Extra metadata သိမ်းဖို့            | `buildversion: "1.34"` |

---

## 12. အသုံးများတဲ့ Commands

### Label ပါတဲ့ Pod တွေကို ကြည့်ရန်

```bash
kubectl get pods --show-labels
```

### Selector နဲ့ Pod ရှာရန်

```bash
kubectl get pods --selector app=App1
```

Short form -

```bash
kubectl get pods -l app=App1
```

### Label တစ်ခုထက်ပိုပြီး ရှာရန်

```bash
kubectl get pods -l app=App1,function=Front-end
```

### All objects ကို label နဲ့ ရှာရန်

```bash
kubectl get all -l app=App1
```

---

## 13. CKA Exam အတွက် မှတ်ရန်

CKA မှာ Labels and Selectors က အရေးကြီးပါတယ်။

အထူးသဖြင့် -

- Pod မှာ label ထည့်နည်း
- Selector နဲ့ Pod ရှာနည်း
- Service selector နဲ့ Pod label ကိုက်အောင်လုပ်နည်း
- ReplicaSet selector နဲ့ template label ကိုက်အောင်ရေးနည်း
- `kubectl get pods --show-labels`
- `kubectl get pods -l key=value`

တွေကို သေချာလေ့ကျင့်ထားသင့်ပါတယ်။

---

## 14. Simple Summary

Labels ဆိုတာ Kubernetes object တွေကို category ခွဲဖို့ သုံးတဲ့ key-value pair တွေပါ။

```yaml
labels:
  app: App1
  function: Front-end
```

Selectors ဆိုတာ label တွေကို အခြေခံပြီး object တွေကို ရှာဖို့ သုံးတာပါ။

```bash
kubectl get pods -l app=App1
```

ReplicaSet နဲ့ Service တွေက Pod တွေကို ချိတ်ဆက်ဖို့ selector ကို သုံးပါတယ်။

Annotations ကတော့ selection အတွက် မဟုတ်ဘဲ extra metadata သိမ်းဖို့ သုံးပါတယ်။

```yaml
annotations:
  buildversion: "1.34"
```

အရေးကြီးဆုံး မှတ်ရန် -

```yaml
selector:
  matchLabels:
    app: App1
```

နဲ့

```yaml
template:
  metadata:
    labels:
      app: App1
```

ကိုက်ညီရပါမယ်။
