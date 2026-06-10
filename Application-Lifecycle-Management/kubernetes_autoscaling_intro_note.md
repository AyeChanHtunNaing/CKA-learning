# Introduction to Autoscaling

> Topic: Kubernetes Autoscaling အခြေခံ — HPA, VPA, Cluster Autoscaler

---

## 1. Overview

Kubernetes မှာ **Autoscaling** ဆိုတာ application load ပေါ်မူတည်ပြီး resource တွေကို အလိုအလျောက် တိုး/လျှော့ လုပ်ပေးတဲ့ concept ဖြစ်ပါတယ်။

CKA exam အတွက် အရေးကြီးတဲ့ Autoscaling topics တွေက —

- Horizontal Pod Autoscaler — HPA
- Vertical Pod Autoscaler — VPA
- Cluster Autoscaler

ဒီ lesson မှာ scaling concept ကို high-level အနေနဲ့ရှင်းထားပါတယ်။

---

## 2. Traditional Scaling Concept

Physical server တွေသုံးတဲ့ traditional environment မှာ application load များလာရင် scaling လုပ်ဖို့နည်း ၂ မျိုးရှိပါတယ်။

1. Vertical Scaling
2. Horizontal Scaling

---

## 3. Vertical Scaling ဆိုတာဘာလဲ?

**Vertical Scaling** ဆိုတာ existing server တစ်လုံးရဲ့ resource ကိုတိုးပေးတာပါ။

ဥပမာ —

- CPU တိုးခြင်း
- Memory တိုးခြင်း
- Disk တိုးခြင်း

```text
Small Server → Bigger Server
```

### Example

Server တစ်လုံးမှာ —

```text
CPU: 2 cores
Memory: 4 GB
```

ရှိတယ်ဆိုပါစို့။ Load များလာတဲ့အခါ —

```text
CPU: 8 cores
Memory: 16 GB
```

အဖြစ်တိုးပေးတာက vertical scaling ဖြစ်ပါတယ်။

### အားနည်းချက်

- Downtime လိုနိုင်တယ်
- Hardware limit ရှိတယ်
- Cost ကြီးနိုင်တယ်
- Single server ပေါ်မူတည်နေတုန်းဖြစ်တယ်

---

## 4. Horizontal Scaling ဆိုတာဘာလဲ?

**Horizontal Scaling** ဆိုတာ server / instance အရေအတွက်ကိုတိုးပေးတာပါ။

ဥပမာ —

```text
1 Server → 2 Servers → 3 Servers
```

Application ကို server အများကြီးပေါ်မှာ run ပြီး load ကို distribute လုပ်တာပါ။

### အားသာချက်

- Downtime မလိုဘဲ scale လုပ်နိုင်တယ်
- High availability ပိုကောင်းတယ်
- Load balancing လုပ်နိုင်တယ်
- Cloud / Kubernetes environment နဲ့ပိုကိုက်တယ်

---

## 5. Horizontal vs Vertical Scaling

| Scaling Type       | Meaning                                       | Example                 |
| ------------------ | --------------------------------------------- | ----------------------- |
| Vertical Scaling   | Existing machine/container resource တိုးခြင်း | CPU/Memory တိုး         |
| Horizontal Scaling | Instance အရေအတွက်တိုးခြင်း                    | Server/Pod အရေအတွက်တိုး |

---

## 6. Scaling in Kubernetes

Kubernetes မှာ scaling ကို အဓိက ၂ level မှာမြင်နိုင်ပါတယ်။

1. Workload Scaling
2. Cluster Infrastructure Scaling

---

## 7. Workload Scaling

Workload ဆိုတာ Kubernetes ထဲမှာ run နေတဲ့ application resources တွေပါ။

ဥပမာ —

- Deployment
- ReplicaSet
- StatefulSet
- Pod

Workload scaling ဆိုတာ application Pods တွေကို scale လုပ်တာပါ။

---

## 8. Cluster Infrastructure Scaling

Cluster infrastructure scaling ဆိုတာ Kubernetes cluster ထဲက node/server resources တွေကို scale လုပ်တာပါ။

ဥပမာ —

- Node အရေအတွက်တိုးခြင်း
- Existing node ရဲ့ CPU/Memory တိုးခြင်း

---

## 9. Kubernetes Scaling Types

Kubernetes မှာ scaling ကို ဒီလိုခွဲမှတ်လို့ရပါတယ်။

| Area                   | Horizontal Scaling | Vertical Scaling                  |
| ---------------------- | ------------------ | --------------------------------- |
| Cluster Infrastructure | Node အရေအတွက်တိုး  | Existing node ရဲ့ CPU/Memory တိုး |
| Workload               | Pod အရေအတွက်တိုး   | Pod resource requests/limits တိုး |

---

## 10. Cluster Infrastructure Horizontal Scaling

Cluster infrastructure horizontal scaling ဆိုတာ cluster ထဲကို node အသစ်တွေထည့်တာပါ။

ဥပမာ —

```text
3 worker nodes → 5 worker nodes
```

Manual command example:

```bash
kubeadm join ...
```

ဒီ command ကို new worker node မှာ run လုပ်ပြီး cluster ထဲ join လုပ်နိုင်ပါတယ်။

---

## 11. Cluster Infrastructure Vertical Scaling

Cluster infrastructure vertical scaling ဆိုတာ existing node ရဲ့ CPU/Memory ကိုတိုးပေးတာပါ။

ဥပမာ —

```text
Worker Node 1:
CPU 2 cores → 8 cores
Memory 4 GB → 16 GB
```

Kubernetes မှာ node vertical scaling က သိပ်မသုံးပါဘူး။  
အကြောင်းရင်းက downtime လိုနိုင်ပြီး cloud/VM environment မှာ node အသစ်တစ်လုံးတင်ပြီး old node ကို drain/decommission လုပ်တာပိုလွယ်ပါတယ်။

---

## 12. Workload Horizontal Scaling

Workload horizontal scaling ဆိုတာ Pod အရေအတွက်ကိုတိုး/လျှော့ လုပ်တာပါ။

ဥပမာ —

```text
Deployment replicas: 2 → 5
```

Manual command:

```bash
kubectl scale --replicas=<number> <workload-type>/<workload-name>
```

Example:

```bash
kubectl scale --replicas=5 deployment/myapp
```

Short form:

```bash
kubectl scale --replicas=5 deploy/myapp
```

---

## 13. Workload Vertical Scaling

Workload vertical scaling ဆိုတာ existing Pod တွေရဲ့ resource requests/limits ကိုတိုး/လျှော့လုပ်တာပါ။

ဥပမာ —

```yaml
resources:
  requests:
    cpu: "250m"
    memory: "128Mi"
  limits:
    cpu: "500m"
    memory: "256Mi"
```

Load များလာလို့ resource တိုးချင်ရင် —

```yaml
resources:
  requests:
    cpu: "500m"
    memory: "256Mi"
  limits:
    cpu: "1"
    memory: "512Mi"
```

Manual edit command:

```bash
kubectl edit <workload-type>/<workload-name>
```

Example:

```bash
kubectl edit deployment/myapp
```

---

## 14. Manual Scaling

Manual scaling ဆိုတာ admin/operator က command run ပြီး resource တွေကို တိုက်ရိုက် scale လုပ်တာပါ။

### 14.1 Manual Cluster Horizontal Scaling

New node ကို cluster ထဲ join လုပ်ခြင်း:

```bash
kubeadm join ...
```

---

### 14.2 Manual Workload Horizontal Scaling

Deployment replicas ကိုပြောင်းခြင်း:

```bash
kubectl scale --replicas=3 deployment/myapp
```

Check:

```bash
kubectl get deployment myapp
kubectl get pods
```

---

### 14.3 Manual Workload Vertical Scaling

Deployment resource requests/limits ကို edit လုပ်ခြင်း:

```bash
kubectl edit deployment/myapp
```

Then update:

```yaml
resources:
  requests:
    cpu: "500m"
    memory: "256Mi"
  limits:
    cpu: "1"
    memory: "512Mi"
```

---

## 15. Automated Scaling

Automated scaling ဆိုတာ Kubernetes controller တွေက metric/load ပေါ်မူတည်ပြီး အလိုအလျောက် scale လုပ်ပေးတာပါ။

အဓိက ၃ မျိုးရှိပါတယ်။

| Area                           | Autoscaler                      |
| ------------------------------ | ------------------------------- |
| Workload Horizontal Scaling    | Horizontal Pod Autoscaler — HPA |
| Workload Vertical Scaling      | Vertical Pod Autoscaler — VPA   |
| Cluster Infrastructure Scaling | Cluster Autoscaler              |

---

## 16. Horizontal Pod Autoscaler — HPA

HPA က Pod replicas အရေအတွက်ကို automatic တိုး/လျှော့လုပ်ပေးပါတယ်။

ဥပမာ CPU usage 70% ကျော်ရင် Pod replicas တိုးစေချင်တယ်ဆိုရင် HPA သုံးနိုင်ပါတယ်။

```text
Low traffic  → 2 Pods
High traffic → 6 Pods
```

HPA က workload horizontal scaling အတွက်ပါ။

---

## 17. Vertical Pod Autoscaler — VPA

VPA က Pod resource requests/limits ကို recommend သို့မဟုတ် adjust လုပ်ပေးပါတယ်။

ဥပမာ Pod တစ်ခုက memory မလုံလောက်ဘူးဆိုရင် VPA က memory request ကိုတိုးဖို့ recommend လုပ်နိုင်ပါတယ်။

```text
Pod memory request: 128Mi → 512Mi
```

VPA က workload vertical scaling အတွက်ပါ။

---

## 18. Cluster Autoscaler

Cluster Autoscaler က cluster ထဲက node အရေအတွက်ကို automatic တိုး/လျှော့လုပ်ပေးပါတယ်။

ဥပမာ —

- Pod တွေ schedule မဖြစ်ဘူး
- Node resource မလုံလောက်ဘူး
- Pending Pods တွေရှိတယ်

ဆိုရင် Cluster Autoscaler က new node ထည့်ပေးနိုင်ပါတယ်။

```text
3 nodes → 5 nodes
```

Cloud-managed Kubernetes တွေမှာ အသုံးများပါတယ်။

---

## 19. Manual vs Automated Scaling

| Scaling Method    | Description                                   | Example                      |
| ----------------- | --------------------------------------------- | ---------------------------- |
| Manual Scaling    | User/Admin က command run လုပ်                 | `kubectl scale`              |
| Automated Scaling | Controller က metrics ပေါ်မူတည်ပြီး scale လုပ် | HPA, VPA, Cluster Autoscaler |

---

## 20. Scaling Summary Table

| Scaling Target      | Horizontal Scaling  | Vertical Scaling                  |
| ------------------- | ------------------- | --------------------------------- |
| Physical Server     | Server အရေအတွက်တိုး | CPU/Memory တိုး                   |
| Kubernetes Node     | Node အရေအတွက်တိုး   | Node CPU/Memory တိုး              |
| Kubernetes Workload | Pod replicas တိုး   | Pod resource requests/limits တိုး |

---

## 21. Important Commands

### Add Node to Cluster

```bash
kubeadm join ...
```

### Scale Deployment Replicas

```bash
kubectl scale --replicas=<number> deployment/<deployment-name>
```

Example:

```bash
kubectl scale --replicas=4 deployment/nginx
```

### Edit Workload Resources

```bash
kubectl edit deployment/<deployment-name>
```

Example:

```bash
kubectl edit deployment/nginx
```

---

## 22. CKA Exam Tips

### Tip 1 — Horizontal vs Vertical ကိုသေချာခွဲပါ

```text
Horizontal = number တိုးခြင်း
Vertical   = resource size တိုးခြင်း
```

---

### Tip 2 — Workload Horizontal Scaling command

```bash
kubectl scale --replicas=5 deploy/myapp
```

---

### Tip 3 — Workload Vertical Scaling command

```bash
kubectl edit deploy/myapp
```

ပြီးရင် `resources.requests` / `resources.limits` ပြင်ပါ။

---

### Tip 4 — Autoscaler mapping မှတ်ပါ

```text
HPA = Pod replicas scale
VPA = Pod resources scale
Cluster Autoscaler = Node scale
```

---

### Tip 5 — CKA မှာ HPA basics အရေးကြီး

HPA ကိုအသုံးပြုဖို့ metrics data လိုပါတယ်။  
CPU/Memory metrics တွေကို metrics-server ကနေယူပါတယ်။

---

## 23. Common Mistakes

### Mistake 1 — HPA နဲ့ VPA ကိုရောထွေးခြင်း

မမှန်:

```text
HPA increases CPU and memory of Pods
```

မှန်:

```text
HPA increases/decreases number of Pods
```

VPA က resource requests/limits ကို adjust လုပ်တာပါ။

---

### Mistake 2 — Horizontal scaling ကို resource တိုးခြင်းလို့ထင်ခြင်း

Horizontal scaling ဆိုတာ instance အရေအတွက်တိုးတာပါ။  
Resource size တိုးတာက vertical scaling ပါ။

---

### Mistake 3 — Cluster Autoscaler နဲ့ HPA ကိုရောထွေးခြင်း

| Autoscaler         | What it scales |
| ------------------ | -------------- |
| HPA                | Pods           |
| Cluster Autoscaler | Nodes          |

---

## 24. Practice Example

### Task 1

Deployment `webapp` ကို replicas 5 ခုဖြစ်အောင် manual scale လုပ်ပါ။

Solution:

```bash
kubectl scale --replicas=5 deployment/webapp
```

Check:

```bash
kubectl get deploy webapp
kubectl get pods
```

---

### Task 2

Deployment `api` ရဲ့ resource requests/limits ကိုပြင်ပါ။

Solution:

```bash
kubectl edit deployment/api
```

Add or update:

```yaml
resources:
  requests:
    cpu: "500m"
    memory: "256Mi"
  limits:
    cpu: "1"
    memory: "512Mi"
```

---

## 25. Key Takeaways

- Autoscaling က Kubernetes မှာ application load ပေါ်မူတည်ပြီး resources တွေကို scale လုပ်တဲ့ concept ဖြစ်ပါတယ်။
- Horizontal scaling = instance အရေအတွက်တိုးခြင်း။
- Vertical scaling = resource size တိုးခြင်း။
- Kubernetes workload horizontal scaling = Pod replicas တိုး/လျှော့ခြင်း။
- Kubernetes workload vertical scaling = Pod requests/limits တိုး/လျှော့ခြင်း။
- Cluster horizontal scaling = Node အရေအတွက်တိုးခြင်း။
- Manual scaling မှာ `kubectl scale`, `kubectl edit`, `kubeadm join` သုံးနိုင်ပါတယ်။
- Automated scaling မှာ HPA, VPA, Cluster Autoscaler သုံးပါတယ်။
- HPA = Pod replicas scale။
- VPA = Pod resources scale။
- Cluster Autoscaler = Node count scale။
