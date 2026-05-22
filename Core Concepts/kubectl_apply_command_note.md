# Kubernetes `kubectl apply` Command Note

> This note explains how the `kubectl apply` command works in Kubernetes.  
> Main topic: Declarative management, three-way merge, and last applied configuration.

---

## 1. `kubectl apply` ဆိုတာဘာလဲ?

`kubectl apply` ဆိုတာ Kubernetes objects တွေကို **declarative way** နဲ့ manage လုပ်ဖို့သုံးတဲ့ command ဖြစ်ပါတယ်။

YAML file ထဲမှာ ကိုယ်လိုချင်တဲ့ **desired state** ကိုရေးထားပြီး `kubectl apply` command နဲ့ cluster ထဲမှာ apply လုပ်ပါတယ်။

အလွယ်မှတ်ရန် -

```text
kubectl apply = YAML file ထဲက state အတိုင်း cluster ကိုဖြစ်အောင်လုပ်ပေးတဲ့ command
```

ဥပမာ -

```bash
kubectl apply -f nginx.yaml
```

ဒီ command က `nginx.yaml` ထဲမှာရေးထားတဲ့ Pod, Deployment, Service စတဲ့ Kubernetes resource ကို cluster ထဲမှာ create သို့မဟုတ် update လုပ်ပေးပါတယ်။

---

## 2. Basic Example

ဥပမာ `nginx.yaml` file ထဲမှာ Pod တစ်ခုရေးထားမယ်။

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
    type: front-end-service
spec:
  containers:
    - name: nginx-container
      image: nginx:1.18
```

Apply လုပ်ရန် -

```bash
kubectl apply -f nginx.yaml
```

ဒီ Pod မရှိသေးရင် Kubernetes က Pod အသစ် create လုပ်ပေးမယ်။

ရှိပြီးသားဆိုရင် YAML file ထဲက state နဲ့ cluster ထဲက live state ကို နှိုင်းယှဉ်ပြီး update လုပ်ပေးမယ်။

---

## 3. Directory တစ်ခုလုံး Apply လုပ်နည်း

File တစ်ခုချင်းမဟုတ်ဘဲ folder ထဲက YAML files အားလုံးကို apply လုပ်ချင်ရင် -

```bash
kubectl apply -f /path/to/config-files
```

ဥပမာ -

```bash
kubectl apply -f ./k8s/
```

ဒါက `./k8s/` folder ထဲက Kubernetes YAML files တွေကို apply လုပ်ပေးပါတယ်။

---

## 4. `kubectl apply` အတွင်းပိုင်းမှာ ဘာတွေဖြစ်လဲ?

`kubectl apply` run လိုက်တဲ့အခါ Kubernetes က configuration ၃ ခုကို နှိုင်းယှဉ်ပါတယ်။

```text
1. Local configuration file
2. Live object configuration
3. Last applied configuration
```

ဒီ ၃ ခုကိုနှိုင်းယှဉ်ပြီး ဘာ update လုပ်ရမလဲ၊ ဘာ remove လုပ်ရမလဲဆိုတာဆုံးဖြတ်ပါတယ်။

---

## 5. Local Configuration File

**Local configuration file** ဆိုတာ ကိုယ့် local machine ထဲမှာရှိတဲ့ YAML file ဖြစ်ပါတယ်။

ဥပမာ -

```text
nginx.yaml
```

ဒါက ကိုယ်လိုချင်တဲ့ desired state ဖြစ်ပါတယ်။

ဥပမာ local YAML ထဲမှာ -

```yaml
image: nginx:1.19
```

လို့ရေးထားရင် Kubernetes က live object ကိုလည်း `nginx:1.19` ဖြစ်အောင် update လုပ်ဖို့ကြိုးစားပါတယ်။

---

## 6. Live Object Configuration

**Live object configuration** ဆိုတာ Kubernetes cluster ထဲမှာ တကယ်ရှိနေတဲ့ object ရဲ့ current state ဖြစ်ပါတယ်။

Live object ကို ဒီလိုကြည့်နိုင်ပါတယ်။

```bash
kubectl get pod myapp-pod -o yaml
```

Live object ထဲမှာ local YAML file ထဲမှာမပါသေးတဲ့ system-generated fields တွေပါနိုင်ပါတယ်။

ဥပမာ -

```text
status
resourceVersion
uid
managedFields
creationTimestamp
```

ဒီ field တွေကို Kubernetes က object ကို manage/monitor လုပ်ဖို့ auto add လုပ်ပေးတာပါ။

---

## 7. Last Applied Configuration ဆိုတာဘာလဲ?

**Last applied configuration** ဆိုတာ `kubectl apply` အရင်တစ်ကြိမ် run လုပ်တုန်းက YAML configuration ကို Kubernetes က annotation အနေနဲ့ သိမ်းထားတာဖြစ်ပါတယ်။

Annotation key က -

```text
kubectl.kubernetes.io/last-applied-configuration
```

ဥပမာ live object ထဲမှာ ဒီလိုတွေ့နိုင်ပါတယ်။

```yaml
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: '{...}'
```

ဒီ annotation က နောက်တစ်ခါ `kubectl apply` ပြန် run လုပ်တဲ့အခါ ဘာတွေပြောင်းသွားလဲဆိုတာ သိဖို့ အရမ်းအရေးကြီးပါတယ်။

---

## 8. Three-way Merge ဆိုတာဘာလဲ?

`kubectl apply` က configuration ၃ ခုကို နှိုင်းယှဉ်ပြီး update လုပ်တာကို **three-way merge** လို့ခေါ်ပါတယ်။

```text
Local YAML
    +
Live Object
    +
Last Applied Configuration
    ↓
kubectl apply decides what to create, update, or remove
```

ဒီ process ကြောင့် Kubernetes က precise update လုပ်နိုင်ပါတယ်။

---

## 9. Image Version Update Example

အရင် local YAML file ထဲမှာ image version က ဒီလိုရှိတယ်။

```yaml
image: nginx:1.18
```

နောက်ပိုင်း local YAML ထဲမှာ ဒီလိုပြောင်းလိုက်တယ်။

```yaml
image: nginx:1.19
```

ပြီးရင် -

```bash
kubectl apply -f nginx.yaml
```

လုပ်လိုက်ရင် Kubernetes က -

```text
Last applied config ထဲမှာ nginx:1.18 ရှိတယ်။
Local YAML အသစ်ထဲမှာ nginx:1.19 ရှိတယ်။
Live object ကို nginx:1.19 ဖြစ်အောင် update လုပ်မယ်။
```

လို့ဆုံးဖြတ်ပါတယ်။

---

## 10. Field ဖြုတ်လိုက်ရင် ဘာဖြစ်မလဲ?

`kubectl apply` ရဲ့အရေးကြီးဆုံး concept တစ်ခုက **field removal** ပါ။

အရင် YAML ထဲမှာ label ၂ ခုရှိတယ်ဆိုပါစို့။

```yaml
metadata:
  labels:
    app: myapp
    type: front-end-service
```

နောက်ပိုင်း local YAML ထဲက `type` label ကိုဖြုတ်လိုက်တယ်။

```yaml
metadata:
  labels:
    app: myapp
```

ပြီးရင် -

```bash
kubectl apply -f nginx.yaml
```

လုပ်လိုက်ရင် `kubectl apply` က -

```text
Last applied configuration ထဲမှာ type label ရှိခဲ့တယ်။
Local YAML အသစ်ထဲမှာ type label မရှိတော့ဘူး။
ဒါကြောင့် live object ထဲက type label ကို remove လုပ်မယ်။
```

လို့ဆုံးဖြတ်နိုင်ပါတယ်။

မှတ်ရန် -

```text
Local YAML ထဲက field ကိုဖြုတ်ပြီး apply လုပ်ရင်
kubectl apply က live object ထဲက field ကိုလည်း remove လုပ်နိုင်တယ်။
```

---

## 11. Why Last Applied Configuration is Important

Last applied configuration annotation မရှိရင် Kubernetes က -

```text
အရင်တုန်းက user apply လုပ်ခဲ့တဲ့ config ကဘာလဲ?
အခု local file ထဲကဘာတွေပြောင်းသွားလဲ?
field တစ်ခုကို remove လုပ်သင့်လား?
```

ဆိုတာကိုတိတိကျကျသိဖို့ခက်နိုင်ပါတယ်။

ဒါကြောင့် `kubectl apply` workflow မှာ annotation က အရေးကြီးပါတယ်။

---

## 12. `kubectl apply` နဲ့ `kubectl create/replace` ကို မရောသင့်တဲ့အကြောင်း

Declarative workflow မှာ `kubectl apply` ကို consistently သုံးတာကောင်းပါတယ်။

ဘာလို့လဲဆိုတော့ -

```bash
kubectl create -f nginx.yaml
```

သို့မဟုတ် -

```bash
kubectl replace -f nginx.yaml
```

တွေက `kubectl apply` လို last applied configuration annotation ကို properly store/update မလုပ်နိုင်ပါဘူး။

ဒါကြောင့် create/replace နဲ့ apply ကိုရောသုံးရင် future apply operation တွေမှာ inconsistency ဖြစ်နိုင်ပါတယ်။

အလွယ်မှတ်ရန် -

```text
Declarative workflow သုံးမယ်ဆိုရင် kubectl apply ကို consistently သုံးပါ။
```

---

## 13. Recommended Workflow

အကောင်းဆုံး workflow က ဒီလိုပါ။

```text
1. YAML file ရေး
2. kubectl apply -f file.yaml
3. Change လုပ်ချင်ရင် YAML file ကိုပြင်
4. kubectl apply -f file.yaml ပြန် run
5. YAML file ကို Git ထဲမှာ version control ထား
```

ဥပမာ -

```bash
kubectl apply -f nginx.yaml
```

Image version ပြောင်းချင်ရင် YAML file ထဲမှာပြင် -

```yaml
image: nginx:1.19
```

ပြီးရင် -

```bash
kubectl apply -f nginx.yaml
```

---

## 14. Actionable Summary

| Step | Description | Command |
|---|---|---|
| Initial Creation | Object မရှိသေးရင် create လုပ်ပြီး config ကို annotation ထဲသိမ်းသည် | `kubectl apply -f nginx.yaml` |
| Update Configuration | Local, live, last applied config ၃ ခုကိုနှိုင်းယှဉ်ပြီး update လုပ်သည် | `kubectl apply -f nginx.yaml` |
| Remove a Field | Local YAML ထဲက field ဖြုတ်ထားရင် live object ထဲက field ကိုပါ remove လုပ်နိုင်သည် | `kubectl apply -f nginx.yaml` |

---

## 15. CKA Exam မှာ မှတ်ရမယ့်အချက်များ

```text
kubectl apply က declarative command ဖြစ်တယ်။
YAML file ထဲက desired state ကို cluster ထဲမှာ apply လုပ်တယ်။
kubectl apply က local file, live object, last applied configuration ၃ ခုကို compare လုပ်တယ်။
last applied configuration ကို annotation ထဲမှာသိမ်းတယ်။
field ဖြုတ်ပြီး apply လုပ်ရင် live object ထဲက field ကိုလည်း remove လုပ်နိုင်တယ်။
kubectl create/replace နဲ့ kubectl apply ကို မရောသင့်ဘူး။
Team/production environment မှာ kubectl apply + YAML + Git workflow ကောင်းတယ်။
```

---

## 16. Quick Memory

```text
kubectl apply = desired state ကို cluster ထဲမှာ match ဖြစ်အောင်လုပ်ပေးတာ
```

```text
Three-way merge:
Local YAML + Live Object + Last Applied Configuration
```

```text
Last Applied Configuration =
kubectl apply အရင်တုန်းက apply လုပ်ထားတဲ့ config ကို annotation ထဲမှာသိမ်းထားတာ
```

---

## 17. Final Summary

`kubectl apply` ဆိုတာ Kubernetes objects တွေကို declarative way နဲ့ manage လုပ်တဲ့ command ဖြစ်ပါတယ်။ YAML file ထဲမှာ desired state ကိုရေးပြီး `kubectl apply -f file.yaml` လုပ်လိုက်ရင် Kubernetes က local YAML, cluster ထဲက live object, annotation ထဲမှာသိမ်းထားတဲ့ last applied configuration ၃ ခုကိုနှိုင်းယှဉ်ပြီး လိုအပ်တဲ့ update ကိုလုပ်ပေးပါတယ်။

ဒါကြောင့် long-term management, team collaboration, GitOps-style workflow တွေအတွက် `kubectl apply` က အရေးကြီးပါတယ်။
