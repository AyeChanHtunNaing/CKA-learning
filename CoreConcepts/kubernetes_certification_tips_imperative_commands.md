# Kubernetes Certification Tips & Tricks Note

> Topic: Imperative Commands with `kubectl` for CKA practice  
> Main idea: Use imperative commands to create resources quickly or generate YAML templates during the exam.

---

## 1. ဒီ Lesson ရဲ့ အဓိက Idea

CKA exam မှာ YAML file တွေကို terminal ထဲမှာ အစကနေရေးရတာ အချိန်ကုန်နိုင်ပါတယ်။ အထူးသဖြင့် browser ကနေ terminal ထဲ copy/paste လုပ်ရတာလည်း အဆင်မပြေတတ်ပါဘူး။

ဒါကြောင့် `kubectl run`, `kubectl create deployment`, `kubectl expose` စတဲ့ imperative commands တွေကို သုံးပြီး resource ကို အမြန် create လုပ်နိုင်သလို YAML template ကိုလည်း generate လုပ်နိုင်ပါတယ်။

အလွယ်မှတ်ရန် -

```text
Imperative command = အမြန် create လုပ်ဖို့
--dry-run=client -o yaml = YAML template generate လုပ်ဖို့
```

---

## 2. Exam မှာဘာကြောင့် Imperative Commands သုံးသင့်လဲ?

CKA exam က hands-on exam ဖြစ်တဲ့အတွက် အချိန်အရမ်းအရေးကြီးပါတယ်။

Imperative command တွေသုံးရင် -

```text
YAML အစကနေမရေးရဘူး
Command တစ်ကြောင်းနဲ့ resource create လုပ်နိုင်တယ်
YAML template ကို generate လုပ်နိုင်တယ်
အချိန်ချွေတာနိုင်တယ်
Mistake လျော့နိုင်တယ်
```

ဥပမာ Pod တစ်ခု create လုပ်ဖို့ YAML အရှည်ကြီးရေးမယ့်အစား -

```bash
kubectl run nginx --image=nginx
```

လို့ရေးလိုက်ရုံနဲ့ရပါတယ်။

---

## 3. အရေးကြီး Option ၂ ခု

### `--dry-run=client`

ဒီ option က command ကို test လုပ်ပေးတာပါ။ Resource ကို တကယ် create မလုပ်ပါဘူး။

```bash
kubectl run nginx --image=nginx --dry-run=client
```

အသုံးဝင်တဲ့အချက် -

```text
Command မှန်လားစစ်နိုင်တယ်
Resource တကယ်မ create လုပ်ဘူး
YAML generate လုပ်ဖို့အသုံးပြုနိုင်တယ်
```

### `-o yaml`

ဒီ option က output ကို YAML format နဲ့ပြပေးပါတယ်။

```bash
kubectl run nginx --image=nginx --dry-run=client -o yaml
```

ဒီ command က Pod ကို မ create လုပ်ဘဲ YAML manifest ကို screen ပေါ်မှာပြပေးပါတယ်။

---

## 4. `--dry-run=client` နဲ့ `-o yaml` ကိုပေါင်းသုံးခြင်း

ဒီနှစ်ခုကိုပေါင်းသုံးရင် YAML file ကိုအမြန် generate လုပ်နိုင်ပါတယ်။

```bash
kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml
```

ဆိုလိုတာက -

```text
nginx Pod YAML ကို generate လုပ်
တကယ် create မလုပ်
output ကို pod.yaml file ထဲ save လုပ်
```

ပြီးရင် file ကိုလိုသလိုပြင်ပြီး create/apply လုပ်နိုင်ပါတယ်။

```bash
kubectl create -f pod.yaml
kubectl apply -f pod.yaml
```

---

# Part 1: Pod Commands

## 5. NGINX Pod Create လုပ်နည်း

```bash
kubectl run nginx --image=nginx
```

ဒီ command က `nginx` image သုံးပြီး `nginx` ဆိုတဲ့ Pod ကို create လုပ်ပေးပါတယ်။

## 6. Pod YAML Generate လုပ်နည်း

Pod ကို မ create လုပ်ဘဲ YAML template ထုတ်ချင်ရင် -

```bash
kubectl run nginx --image=nginx --dry-run=client -o yaml
```

File ထဲ save ချင်ရင် -

```bash
kubectl run nginx --image=nginx --dry-run=client -o yaml > nginx-pod.yaml
```

ပြီးရင် -

```bash
kubectl create -f nginx-pod.yaml
kubectl apply -f nginx-pod.yaml
```

---

# Part 2: Deployment Commands

## 7. Deployment Create လုပ်နည်း

```bash
kubectl create deployment nginx --image=nginx
```

ဒီ command က `nginx` image နဲ့ Deployment တစ်ခု create လုပ်ပေးပါတယ်။

## 8. Deployment YAML Generate လုပ်နည်း

Deployment ကို မ create လုပ်ဘဲ YAML template ထုတ်ချင်ရင် -

```bash
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml
```

File ထဲ save ချင်ရင် -

```bash
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > nginx-deployment.yaml
```

ပြီးရင် YAML file ကိုလိုသလိုပြင်နိုင်ပါတယ်။

## 9. Replicas ပါတဲ့ Deployment Generate လုပ်နည်း

Kubernetes version 1.19+ မှာ `--replicas` option ကိုသုံးနိုင်ပါတယ်။

```bash
kubectl create deployment nginx --image=nginx --replicas=4
```

YAML generate + file save လုပ်ချင်ရင် -

```bash
kubectl create deployment nginx --image=nginx --replicas=4 --dry-run=client -o yaml > nginx-deployment.yaml
```

ပြီးရင် create/apply လုပ်နိုင်ပါတယ်။

```bash
kubectl create -f nginx-deployment.yaml
kubectl apply -f nginx-deployment.yaml
```

## 10. Deployment Scale လုပ်နည်း

ရှိပြီးသား Deployment ကို replicas 4 လုံးဖြစ်အောင် scale လုပ်ချင်ရင် -

```bash
kubectl scale deployment nginx --replicas=4
```

ဒီ command က live Deployment ကိုတန်းပြီး scale လုပ်ပေးပါတယ်။

---

# Part 3: Service Commands

## 11. ClusterIP Service Create/Generate လုပ်နည်း

ဥပမာ `redis` Pod ကို port `6379` နဲ့ expose လုပ်မယ်ဆိုရင် -

```bash
kubectl expose pod redis --port=6379 --name=redis-service --dry-run=client -o yaml
```

ဒီ command က `redis-service` ဆိုတဲ့ ClusterIP Service YAML ကို generate လုပ်ပေးပါတယ်။

အရေးကြီးတဲ့အချက် -

```text
kubectl expose pod redis က Pod ရဲ့ labels တွေကို selector အဖြစ် automatically သုံးနိုင်တယ်။
```

## 12. `kubectl create service clusterip` သုံးနည်း

နောက်တစ်နည်း -

```bash
kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml
```

ဒီ command က Pod ရဲ့ real labels ကို auto မယူဘဲ `app=redis` selector လို့ assume လုပ်တတ်ပါတယ်။ Pod မှာ label မတူရင် Service က Pod ကိုမချိတ်နိုင်ပါဘူး။

မှတ်ရန် -

```text
kubectl expose pod ... = selector ပိုမှန်နိုင်တယ်
kubectl create service clusterip ... = selector ကိုပြန်စစ်ဖို့လိုတယ်
```

## 13. NodePort Service Generate လုပ်နည်း

ဥပမာ `nginx` Pod ရဲ့ port 80 ကို NodePort အနေနဲ့ expose လုပ်ချင်ရင် -

```bash
kubectl expose pod nginx --type=NodePort --port=80 --name=nginx-service --dry-run=client -o yaml
```

ဒီ command က Pod labels ကို selector အဖြစ် auto သုံးနိုင်ပါတယ်။

ဒါပေမယ့် limitation တစ်ခုရှိပါတယ် -

```text
kubectl expose command နဲ့ nodePort number ကိုတိုက်ရိုက် specify လုပ်လို့မရနိုင်ပါ။
```

ဒါကြောင့် nodePort ကို သတ်မှတ်ချင်ရင် YAML file generate လုပ်ပြီး manual ထည့်ရပါမယ်။

```bash
kubectl expose pod nginx --type=NodePort --port=80 --name=nginx-service --dry-run=client -o yaml > nginx-service.yaml
```

ပြီးရင် YAML file ထဲမှာ -

```yaml
nodePort: 30080
```

ကို manual ထည့်ပါ။

## 14. `kubectl create service nodeport` သုံးနည်း

နောက်တစ်နည်း -

```bash
kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml
```

ဒီ command က nodePort ကို specify လုပ်နိုင်ပါတယ်။ ဒါပေမယ့် Pod labels ကို selector အနေနဲ့ auto မယူနိုင်ပါ။ အဲဒါကြောင့် selector ကိုပြန်စစ်/ပြင်ဖို့လိုပါတယ်။

## 15. NodePort Service အတွက် Best Practice

Lesson ထဲက recommendation အရ -

```text
NodePort Service generate လုပ်ချင်ရင် kubectl expose command ကိုပိုသုံးသင့်တယ်။
nodePort လိုအပ်ရင် YAML file generate လုပ်ပြီး nodePort ကို manual ထည့်ပါ။
```

ဥပမာ -

```bash
kubectl expose pod nginx --type=NodePort --port=80 --name=nginx-service --dry-run=client -o yaml > nginx-service.yaml
```

ပြီးရင် YAML ထဲမှာ -

```yaml
ports:
  - port: 80
    targetPort: 80
    nodePort: 30080
```

လိုအပ်သလိုပြင်ပြီး -

```bash
kubectl create -f nginx-service.yaml
kubectl apply -f nginx-service.yaml
```

---

# 16. Useful Command Summary

| Task | Command |
|---|---|
| Create NGINX Pod | `kubectl run nginx --image=nginx` |
| Generate Pod YAML | `kubectl run nginx --image=nginx --dry-run=client -o yaml` |
| Save Pod YAML | `kubectl run nginx --image=nginx --dry-run=client -o yaml > nginx-pod.yaml` |
| Create Deployment | `kubectl create deployment nginx --image=nginx` |
| Generate Deployment YAML | `kubectl create deployment nginx --image=nginx --dry-run=client -o yaml` |
| Save Deployment YAML | `kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > nginx-deployment.yaml` |
| Create Deployment with Replicas | `kubectl create deployment nginx --image=nginx --replicas=4` |
| Scale Deployment | `kubectl scale deployment nginx --replicas=4` |
| Generate ClusterIP Service YAML | `kubectl expose pod redis --port=6379 --name=redis-service --dry-run=client -o yaml` |
| Generate NodePort Service YAML | `kubectl expose pod nginx --type=NodePort --port=80 --name=nginx-service --dry-run=client -o yaml` |

---

# 17. CKA Exam Tips

```text
YAML အစကနေမရေးပါနဲ့။
kubectl command နဲ့ YAML template generate လုပ်ပါ။
--dry-run=client -o yaml ကို အမြဲအသုံးချပါ။
Simple Pod/Deployment တွေကို imperative command နဲ့ create လုပ်ပါ။
Complex configuration လိုရင် YAML generate လုပ်ပြီးပြင်ပါ။
Service selector ကိုအမြဲစစ်ပါ။
NodePort ကို command ကနေ specify မရရင် YAML ထဲမှာ manual ထည့်ပါ။
```

---

# 18. Quick Memory

```text
Pod YAML:
kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml
```

```text
Deployment YAML:
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > deploy.yaml
```

```text
ClusterIP Service YAML:
kubectl expose pod redis --port=6379 --name=redis-service --dry-run=client -o yaml > svc.yaml
```

```text
NodePort Service YAML:
kubectl expose pod nginx --type=NodePort --port=80 --name=nginx-service --dry-run=client -o yaml > svc.yaml
```

---

# Final Summary

CKA exam မှာ အချိန်ချွေတာဖို့ imperative commands တွေကို ကောင်းကောင်းသုံးတတ်ဖို့လိုပါတယ်။ `kubectl run`, `kubectl create deployment`, `kubectl expose`, `kubectl scale` စတဲ့ commands တွေက simple resources တွေကို အမြန် create လုပ်နိုင်ပါတယ်။

YAML file လိုအပ်တဲ့အခါ `--dry-run=client -o yaml` နဲ့ template generate လုပ်ပြီး file ထဲ save လုပ်ကာ လိုသလိုပြင်နိုင်ပါတယ်။ Service create လုပ်တဲ့အခါ selector ကိုသေချာစစ်ပြီး NodePort လို manual field တွေကို YAML ထဲမှာပြင်ထည့်သင့်ပါတယ်။
