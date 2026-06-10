# Kubernetes Rolling Updates and Rollbacks

> Topic: Kubernetes Deployment update, rollout, revision, strategy, and rollback

---

## 1. Overview

Kubernetes မှာ `Deployment` တစ်ခုကို update လုပ်တဲ့အခါ Kubernetes က **rollout** process ကို စတင်လုပ်ဆောင်ပါတယ်။

Deployment ကို ပထမဆုံး create လုပ်တဲ့အချိန်မှာ **Revision 1** ဖြစ်ပါတယ်။  
နောက်ပိုင်း container image version ပြောင်းတာ၊ label ပြောင်းတာ၊ template spec ပြောင်းတာတွေ လုပ်လိုက်ရင် **Revision 2, Revision 3** စသဖြင့် revision အသစ်တွေ ဖြစ်လာပါတယ်။

ဒီ revision တွေကြောင့် update မှားသွားရင် previous version ကို ပြန် rollback လုပ်နိုင်ပါတယ်။

---

## 2. Rollout and Versioning

Deployment တစ်ခုမှာ version အသစ် update လုပ်တိုင်း Kubernetes က rollout revision အသစ်တစ်ခု create လုပ်ပါတယ်။

ဥပမာ —

- `nginx:1.7.0` → Revision 1
- `nginx:1.7.1` → Revision 2

![Rollout and Versioning](https://kodekloud.com/kk-media/image/upload/v1752869667/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-Rolling-Updates-and-Rollbacks/frame_40.jpg)

### Rollout status ကြည့်ရန်

```bash
kubectl rollout status deployment/myapp-deployment
```

Short form:

```bash
kubectl rollout status deploy/myapp-deployment
```

### Rollout history ကြည့်ရန်

```bash
kubectl rollout history deployment/myapp-deployment
```

Short form:

```bash
kubectl rollout history deploy/myapp-deployment
```

---

## 3. Deployment Strategies

Kubernetes Deployment update လုပ်တဲ့အခါ strategy အဓိက ၂ မျိုးရှိပါတယ်။

### 3.1 Recreate Strategy

`Recreate` strategy မှာ old pods အားလုံးကို အရင် stop လုပ်ပြီးမှ new pods တွေကို create လုပ်ပါတယ်။

ဒီနည်းလမ်းမှာ application downtime ရှိနိုင်ပါတယ်။

![Recreate Strategy Downtime](https://kodekloud.com/kk-media/image/upload/v1752869668/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-Rolling-Updates-and-Rollbacks/frame_120.jpg)

#### Example

```yaml
strategy:
  type: Recreate
```

### Recreate Strategy Flow

1. Old ReplicaSet ကို scale down လုပ်မယ်
2. Old Pods အားလုံး terminate ဖြစ်မယ်
3. New ReplicaSet ကို scale up လုပ်မယ်
4. New Pods တွေ running ဖြစ်လာမယ်

အားနည်းချက် — Downtime ဖြစ်နိုင်ပါတယ်။

---

### 3.2 RollingUpdate Strategy

`RollingUpdate` strategy မှာ old pods တွေကို တစ်ခုပြီးတစ်ခု replace လုပ်ပါတယ်။

ဒီနည်းလမ်းက application ကို downtime မဖြစ်အောင် update လုပ်နိုင်ပါတယ်။

Kubernetes Deployment ရဲ့ default strategy က `RollingUpdate` ဖြစ်ပါတယ်။

![Recreate vs RollingUpdate](https://kodekloud.com/kk-media/image/upload/v1752869670/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-Rolling-Updates-and-Rollbacks/frame_140.jpg)

#### Example

```yaml
strategy:
  type: RollingUpdate
```

### RollingUpdate Strategy Flow

1. New ReplicaSet create လုပ်မယ်
2. New Pod တစ်ချို့ကို create လုပ်မယ်
3. Old Pod တစ်ချို့ကို terminate လုပ်မယ်
4. New Pod တွေ ready ဖြစ်လာတာနဲ့ old pod တွေကို တဖြည်းဖြည်းချင်း replace လုပ်မယ်
5. Update ပြီးသွားတဲ့အခါ old ReplicaSet ကို scale down လုပ်ထားမယ်

---

## 4. Updating a Deployment

Deployment ကို update လုပ်နိုင်တဲ့နည်းတွေက —

- YAML file ထဲက image version ပြောင်းပြီး `kubectl apply` လုပ်ခြင်း
- `kubectl set image` command နဲ့ image ကို update လုပ်ခြင်း
- replicas count ပြောင်းခြင်း
- labels / pod template spec ပြောင်းခြင်း

---

## 5. Update by YAML File

ဥပမာ deployment YAML:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
  labels:
    app: myapp
    type: front-end
spec:
  replicas: 3
  selector:
    matchLabels:
      type: front-end
  template:
    metadata:
      labels:
        app: myapp
        type: front-end
    spec:
      containers:
        - name: nginx-container
          image: nginx:1.7.1
```

Apply လုပ်ရန်:

```bash
kubectl apply -f deployment-definition.yml
```

ဒီ command ကို run လိုက်ရင် Kubernetes က rollout အသစ် create လုပ်ပါတယ်။

---

## 6. Update by `kubectl set image`

Container image ကို command နဲ့ တိုက်ရိုက် update လုပ်နိုင်ပါတယ်။

```bash
kubectl set image deployment/myapp-deployment nginx-container=nginx:1.9.1
```

Short form:

```bash
kubectl set image deploy/myapp-deployment nginx-container=nginx:1.9.1
```

### သတိထားရန်

`kubectl set image` နဲ့ update လုပ်တာက running deployment ကိုပဲ update လုပ်တာဖြစ်ပါတယ်။  
Local YAML file ထဲက image version ကိုတော့ မပြောင်းပေးပါ။

ဒါကြောင့် Git repo / YAML file ကိုပါ ပြန် update လုပ်ထားသင့်ပါတယ်။

---

## 7. Viewing Deployment Details

Deployment strategy, revision, ReplicaSet scaling event တွေကိုကြည့်ချင်ရင် —

```bash
kubectl describe deployment myapp-deployment
```

Short form:

```bash
kubectl describe deploy myapp-deployment
```

ဒီ output ထဲမှာ အရေးကြီးတာတွေက —

- `StrategyType`
- `RollingUpdateStrategy`
- `OldReplicaSets`
- `NewReplicaSet`
- `Events`
- `deployment.kubernetes.io/revision`

---

## 8. Recreate vs RollingUpdate Comparison

| Feature             | Recreate                     | RollingUpdate                              |
| ------------------- | ---------------------------- | ------------------------------------------ |
| Update Method       | Old Pods အားလုံးကို အရင်ဖျက် | တစ်ခုပြီးတစ်ခု replace                     |
| Downtime            | ရှိနိုင်                     | များသောအားဖြင့် မရှိ                       |
| Default Strategy    | မဟုတ်                        | ဟုတ်                                       |
| Use Case            | Downtime ခံနိုင်တဲ့ app      | Production app                             |
| ReplicaSet Behavior | Old RS scale down first      | Old RS and New RS run together temporarily |

---

## 9. Upgrade Process

Deployment upgrade လုပ်တဲ့အခါ Kubernetes က new ReplicaSet တစ်ခု create လုပ်ပါတယ်။

Old ReplicaSet က old version pods တွေကို ထိန်းထားပြီး  
New ReplicaSet က new version pods တွေကို တဖြည်းဖြည်း create လုပ်ပါတယ်။

![Deployment Upgrade with ReplicaSets](https://kodekloud.com/kk-media/image/upload/v1752869671/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-Rolling-Updates-and-Rollbacks/frame_300.jpg)

ReplicaSet တွေကိုကြည့်ရန်:

```bash
kubectl get replicasets
```

Short form:

```bash
kubectl get rs
```

---

## 10. Rollback

Update လုပ်ပြီး application မှာ issue ဖြစ်သွားရင် previous revision ကို rollback လုပ်နိုင်ပါတယ်။

```bash
kubectl rollout undo deployment/myapp-deployment
```

Short form:

```bash
kubectl rollout undo deploy/myapp-deployment
```

Rollback လုပ်ပြီး status ပြန်စစ်ရန်:

```bash
kubectl rollout status deploy/myapp-deployment
kubectl get rs
kubectl get pods
```

---

## 11. Rollback to Specific Revision

Specific revision ကို rollback လုပ်ချင်ရင် အရင် history ကြည့်ပါ။

```bash
kubectl rollout history deploy/myapp-deployment
```

Revision number သိပြီးရင် —

```bash
kubectl rollout undo deploy/myapp-deployment --to-revision=2
```

---

## 12. Useful Commands Summary

| Task                        | Command                                                                 |
| --------------------------- | ----------------------------------------------------------------------- |
| Create deployment from YAML | `kubectl create -f deployment-definition.yml`                           |
| Apply update from YAML      | `kubectl apply -f deployment-definition.yml`                            |
| Get deployments             | `kubectl get deployments`                                               |
| Get deployment short form   | `kubectl get deploy`                                                    |
| Update image                | `kubectl set image deploy/myapp-deployment nginx-container=nginx:1.9.1` |
| Check rollout status        | `kubectl rollout status deploy/myapp-deployment`                        |
| View rollout history        | `kubectl rollout history deploy/myapp-deployment`                       |
| Rollback deployment         | `kubectl rollout undo deploy/myapp-deployment`                          |
| Rollback to revision        | `kubectl rollout undo deploy/myapp-deployment --to-revision=2`          |
| Get ReplicaSets             | `kubectl get rs`                                                        |
| Describe deployment         | `kubectl describe deploy myapp-deployment`                              |

---

## 13. CKA Exam Tips

### Tip 1 — Short names သုံးပါ

CKA exam မှာ command မြန်မြန်ရေးဖို့ short form တွေသုံးပါ။

```bash
deploy = deployment
rs = replicasets
po = pods
svc = service
ns = namespace
```

ဥပမာ —

```bash
kubectl get deploy
kubectl get rs
kubectl get po
```

---

### Tip 2 — Update ပြီးတိုင်း rollout status စစ်ပါ

```bash
kubectl rollout status deploy/myapp-deployment
```

---

### Tip 3 — Error ဖြစ်ရင် rollback ချက်ချင်းလုပ်နိုင်ပါစေ

```bash
kubectl rollout undo deploy/myapp-deployment
```

---

### Tip 4 — Image update လုပ်ပြီး Pod တွေ running ဖြစ်မဖြစ်စစ်ပါ

```bash
kubectl get pods
kubectl describe pod <pod-name>
```

---

## 14. Practice Example

### Task

`myapp-deployment` ရဲ့ `nginx-container` image ကို `nginx:1.9.1` သို့ update လုပ်ပါ။  
Rollout status စစ်ပါ။  
Issue ရှိရင် rollback လုပ်ပါ။

### Solution

```bash
kubectl set image deploy/myapp-deployment nginx-container=nginx:1.9.1
kubectl rollout status deploy/myapp-deployment
kubectl get rs
kubectl get po
```

Rollback လုပ်ရန်:

```bash
kubectl rollout undo deploy/myapp-deployment
kubectl rollout status deploy/myapp-deployment
```

---

## 15. Key Takeaways

- Deployment update လုပ်တိုင်း Kubernetes က rollout revision အသစ် create လုပ်ပါတယ်။
- `RollingUpdate` က default strategy ဖြစ်ပြီး downtime မဖြစ်အောင် update လုပ်နိုင်ပါတယ်။
- `Recreate` strategy မှာ downtime ရှိနိုင်ပါတယ်။
- `kubectl rollout status` နဲ့ update progress စစ်နိုင်ပါတယ်။
- `kubectl rollout history` နဲ့ revision history ကြည့်နိုင်ပါတယ်။
- `kubectl rollout undo` နဲ့ previous version ကို rollback လုပ်နိုင်ပါတယ်။
- CKA exam မှာ `deploy`, `rs`, `po` short forms တွေသုံးတာ အချိန်သက်သာပါတယ်။
