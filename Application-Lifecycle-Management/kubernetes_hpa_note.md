# Horizontal Pod Autoscaler HPA

> Topic: Kubernetes Horizontal Pod Autoscaler (HPA) ကိုသုံးပြီး workload replicas ကို automatic scale လုပ်ခြင်း

---

## 1. Overview

**Horizontal Pod Autoscaler (HPA)** ဆိုတာ Kubernetes မှာ workload တစ်ခုရဲ့ Pod replicas အရေအတွက်ကို metric ပေါ်မူတည်ပြီး automatic တိုး/လျှော့လုပ်ပေးတဲ့ resource ဖြစ်ပါတယ်။

HPA က အဓိကအားဖြင့် —

- CPU usage
- Memory usage
- Custom metrics
- External metrics

တွေကိုကြည့်ပြီး Deployment, StatefulSet, ReplicaSet စတဲ့ workload တွေကို scale လုပ်နိုင်ပါတယ်။

CKA exam အတွက် HPA concept, command, YAML, metrics-server requirement တွေကိုသေချာနားလည်ဖို့လိုပါတယ်။

---

## 2. Manual Horizontal Scaling

HPA မသုံးခင် workload horizontal scaling ကို manually လုပ်နိုင်ပါတယ်။

ဥပမာ Deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: my-app
          image: nginx
          resources:
            requests:
              cpu: "250m"
            limits:
              cpu: "500m"
```

ဒီ Deployment မှာ —

- replicas = `1`
- CPU request = `250m`
- CPU limit = `500m`

---

## 3. Pod Resource Usage ကြည့်ခြင်း

Pod CPU/Memory usage ကိုကြည့်ချင်ရင် `kubectl top` သုံးပါတယ်။

```bash
kubectl top pod my-app-pod
```

Example output:

```text
NAME         CPU(cores)   MEMORY(bytes)
my-app-pod   450m         350Mi
```

ဒီ output မှာ Pod က CPU `450m` သုံးနေတယ်ဆိုတာမြင်ရပါတယ်။

> `kubectl top` အလုပ်လုပ်ဖို့ metrics-server လိုပါတယ်။

---

## 4. Manual Scale Command

CPU usage များလာလို့ Pod replicas တိုးချင်ရင် manually scale လုပ်နိုင်ပါတယ်။

```bash
kubectl scale deployment my-app --replicas=3
```

Short form:

```bash
kubectl scale deploy my-app --replicas=3
```

Check:

```bash
kubectl get deploy my-app
kubectl get pods
```

---

## 5. Problem with Manual Scaling

Manual scaling ရဲ့ အားနည်းချက်တွေက —

- Admin က traffic ကိုအမြဲ monitor လုပ်နေရတယ်
- Traffic spike ဖြစ်တဲ့အချိန် timely response မလုပ်နိုင်ရင် app slow/down ဖြစ်နိုင်တယ်
- Traffic ပြန်နည်းသွားတဲ့အခါ replicas ကို manually လျှော့ရတယ်
- Resource waste ဖြစ်နိုင်တယ်

ဒါကြောင့် automatic scaling လုပ်ဖို့ HPA ကိုသုံးပါတယ်။

---

## 6. Horizontal Pod Autoscaler (HPA) ဆိုတာဘာလဲ?

HPA က workload ရဲ့ current metric usage ကို target metric နဲ့နှိုင်းယှဉ်ပြီး replicas count ကို automatic ပြောင်းပေးပါတယ်။

ဥပမာ —

```text
CPU usage high  → replicas increase
CPU usage low   → replicas decrease
```

HPA က အောက်ပါ workload တွေကို scale လုပ်နိုင်ပါတယ်။

- Deployment
- ReplicaSet
- StatefulSet

---

## 7. HPA Diagram

![Horizontal Pod Autoscaler Diagram](https://kodekloud.com/kk-media/image/upload/v1752869662/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-Horizontal-Pod-Autoscaler-HPA-2025-Updates/horizontal-pod-autoscaler-diagram.jpg)

HPA က metrics ကို observe လုပ်ပြီး threshold ပေါ်မူတည်ပြီး Pods တွေကို add/remove လုပ်ပါတယ်။

---

## 8. HPA ဘယ်လိုအလုပ်လုပ်လဲ?

HPA workflow:

```text
1. HPA watches workload
2. metrics-server က CPU/Memory metrics ပေးတယ်
3. HPA က current usage ကို target usage နဲ့ compare လုပ်တယ်
4. Usage high ဖြစ်ရင် replicas တိုးတယ်
5. Usage low ဖြစ်ရင် replicas လျှော့တယ်
6. replicas count ကို min/max limit ထဲမှာပဲ ထားတယ်
```

---

## 9. HPA Requirement

HPA သုံးဖို့လိုအပ်တာတွေ —

1. Workload ရှိရမယ်
2. Pod containers မှာ resource requests သတ်မှတ်ထားသင့်တယ်
3. metrics-server install ဖြစ်ရမယ်
4. HPA object create လုပ်ထားရမယ်

### metrics-server

HPA က CPU/Memory metrics တွေကို metrics-server ကနေယူပါတယ်။

```bash
kubectl top pods
kubectl top nodes
```

ဒီ commands တွေ အလုပ်လုပ်ရင် metrics-server အလုပ်လုပ်နေတယ်လို့ယူဆနိုင်ပါတယ်။

---

## 10. Create HPA — Imperative Method

Deployment `my-app` ကို CPU utilization 50% target နဲ့ replicas 1 မှ 10 အထိ scale လုပ်စေချင်ရင် —

```bash
kubectl autoscale deployment my-app --cpu-percent=50 --min=1 --max=10
```

Short form:

```bash
kubectl autoscale deploy my-app --cpu-percent=50 --min=1 --max=10
```

ဒီ command က HPA object တစ်ခု create လုပ်ပါတယ်။

---

## 11. HPA Command Meaning

```bash
kubectl autoscale deployment my-app --cpu-percent=50 --min=1 --max=10
```

| Part                | Meaning                            |
| ------------------- | ---------------------------------- |
| `kubectl autoscale` | HPA create လုပ်ရန်                 |
| `deployment my-app` | scale လုပ်မယ့် target workload     |
| `--cpu-percent=50`  | average CPU utilization target 50% |
| `--min=1`           | minimum replicas                   |
| `--max=10`          | maximum replicas                   |

---

## 12. Check HPA Status

HPA ကိုကြည့်ရန်:

```bash
kubectl get hpa
```

Example output:

```text
NAME     REFERENCE           TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
my-app   Deployment/my-app   30%/50%   1         10        2          5m
```

Meaning:

| Column     | Meaning                      |
| ---------- | ---------------------------- |
| `TARGETS`  | current usage / target usage |
| `MINPODS`  | minimum replicas             |
| `MAXPODS`  | maximum replicas             |
| `REPLICAS` | current replicas             |

---

## 13. Describe HPA

Detailed HPA info ကြည့်ရန်:

```bash
kubectl describe hpa my-app
```

ဒီ command နဲ့ —

- target workload
- metric type
- current metric
- desired replicas
- scaling events

တွေကိုကြည့်နိုင်ပါတယ်။

---

## 14. Delete HPA

HPA မလိုတော့ရင် delete လုပ်နိုင်ပါတယ်။

```bash
kubectl delete hpa my-app
```

Delete လုပ်ပြီးနောက် HPA က replicas ကို automatic မပြောင်းတော့ပါဘူး။  
ဒါပေမယ့် current replicas count က ချက်ချင်း original value ပြန်မဖြစ်နိုင်ပါဘူး။

---

## 15. Create HPA — Declarative YAML

HPA ကို YAML file နဲ့လည်း create လုပ်နိုင်ပါတယ်။

`autoscaling/v2` API example:

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  minReplicas: 1
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 50
```

Apply:

```bash
kubectl apply -f hpa.yaml
```

---

## 16. HPA YAML Explanation

| Field                           | Meaning                                   |
| ------------------------------- | ----------------------------------------- |
| `apiVersion: autoscaling/v2`    | HPA API version                           |
| `kind: HorizontalPodAutoscaler` | HPA resource                              |
| `scaleTargetRef`                | HPA က scale လုပ်မယ့် workload             |
| `minReplicas`                   | minimum Pod replicas                      |
| `maxReplicas`                   | maximum Pod replicas                      |
| `metrics`                       | scaling decision အတွက်အသုံးပြုမယ့် metric |
| `averageUtilization`            | target average usage percentage           |

---

## 17. scaleTargetRef

`scaleTargetRef` က HPA ဘယ် workload ကို scale လုပ်မလဲဆိုတာသတ်မှတ်တာပါ။

```yaml
scaleTargetRef:
  apiVersion: apps/v1
  kind: Deployment
  name: my-app
```

ဒီ example မှာ HPA က `my-app` Deployment ကို scale လုပ်ပါမယ်။

---

## 18. Metrics Section

CPU utilization 50% ကို target ထားထားတဲ့ example:

```yaml
metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```

ဆိုလိုတာက Deployment ထဲက Pods တွေရဲ့ average CPU utilization ကို 50% ဝန်းကျင်မှာထားဖို့ HPA က replicas ကိုတိုး/လျှော့လုပ်ပါမယ်။

---

## 19. How HPA Calculates CPU Utilization

HPA CPU utilization ကို container `requests.cpu` အပေါ်မူတည်ပြီးတွက်ပါတယ်။

ဥပမာ —

```yaml
resources:
  requests:
    cpu: "250m"
```

Pod current CPU usage = `125m` ဆိုရင် —

```text
CPU utilization = 125m / 250m = 50%
```

ဒါကြောင့် HPA အတွက် `resources.requests.cpu` သတ်မှတ်ထားတာ အရေးကြီးပါတယ်။

---

## 20. HPA with Memory Metric

`autoscaling/v2` မှာ memory metric ကိုလည်းသတ်မှတ်နိုင်ပါတယ်။

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  minReplicas: 1
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 70
```

---

## 21. Metrics Sources

HPA က metrics တွေကို source အမျိုးမျိုးကနေယူနိုင်ပါတယ်။

| Metric Source    | Example                           |
| ---------------- | --------------------------------- |
| Resource metrics | CPU, Memory                       |
| Custom metrics   | requests per second, queue length |
| External metrics | Datadog, Dynatrace, cloud metrics |

---

## 22. Metrics Architecture

![Metrics System Architecture](https://kodekloud.com/kk-media/image/upload/v1752869663/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-Horizontal-Pod-Autoscaler-HPA-2025-Updates/metrics-system-architecture-flowchart.jpg)

HPA က metrics-server, custom metrics adapter, external metrics adapter တွေကနေ metric data တွေယူနိုင်ပါတယ်။

---

## 23. Custom Metrics Adapter

Custom metrics adapter က internal application metrics တွေကို Kubernetes HPA အသုံးပြုနိုင်တဲ့ format နဲ့ expose လုပ်ပေးပါတယ်။

ဥပမာ —

- HTTP requests per second
- Queue length
- Active users count
- Processing jobs count

---

## 24. External Metrics Adapter

External metrics adapter က external monitoring providers တွေကနေ metrics တွေယူနိုင်ပါတယ်။

Examples:

- Datadog
- Dynatrace
- Prometheus adapter
- Cloud provider metrics

---

## 25. HPA Scaling Behavior

HPA က replicas ကို min/max range ထဲမှာပဲ scale လုပ်ပါတယ်။

ဥပမာ —

```bash
--min=1 --max=10
```

ဒါဆို —

- replicas 1 ထက်နည်းအောင် မလုပ်ပါဘူး
- replicas 10 ထက်များအောင် မလုပ်ပါဘူး

---

## 26. HPA and Deployment Replicas

HPA attach လုပ်ထားတဲ့ Deployment ကို manual scale လုပ်တာ မကောင်းပါဘူး။  
Manual scale လုပ်လိုက်ရင် HPA က metric ပေါ်မူတည်ပြီး ပြန်ပြောင်းနိုင်ပါတယ်။

HPA ရှိနေတဲ့ workload မှာ desired replicas ကို HPA က manage လုပ်ပါတယ်။

---

## 27. Important Commands

| Task                    | Command                                                                 |
| ----------------------- | ----------------------------------------------------------------------- |
| Check Pod metrics       | `kubectl top pod <pod-name>`                                            |
| Check all Pod metrics   | `kubectl top pods`                                                      |
| Manual scale Deployment | `kubectl scale deployment my-app --replicas=3`                          |
| Create HPA              | `kubectl autoscale deployment my-app --cpu-percent=50 --min=1 --max=10` |
| Get HPA                 | `kubectl get hpa`                                                       |
| Describe HPA            | `kubectl describe hpa my-app`                                           |
| Delete HPA              | `kubectl delete hpa my-app`                                             |
| Apply HPA YAML          | `kubectl apply -f hpa.yaml`                                             |

---

## 28. CKA Exam Tips

### Tip 1 — HPA meaning ကိုမှတ်ပါ

```text
HPA = Horizontal Pod Autoscaler
HPA scales number of Pods
```

---

### Tip 2 — HPA create command မှတ်ပါ

```bash
kubectl autoscale deploy my-app --cpu-percent=50 --min=1 --max=10
```

---

### Tip 3 — HPA status ကြည့်ရန်

```bash
kubectl get hpa
kubectl describe hpa my-app
```

---

### Tip 4 — HPA needs metrics-server

```bash
kubectl top pods
```

ဒီ command မရရင် metrics-server problem ဖြစ်နိုင်ပါတယ်။

---

### Tip 5 — CPU utilization depends on requests.cpu

HPA CPU utilization calculation က CPU request အပေါ်မူတည်ပါတယ်။  
ဒါကြောင့် deployment container မှာ `resources.requests.cpu` ထည့်ထားသင့်ပါတယ်။

---

## 29. Common Mistakes

### Mistake 1 — HPA က CPU limit အပေါ်တွက်တယ်လို့ထင်ခြင်း

HPA CPU utilization က generally `requests.cpu` အပေါ်မူတည်ပြီးတွက်ပါတယ်။

မမှန်:

```text
HPA uses CPU limit for utilization percentage.
```

မှန်:

```text
HPA uses CPU request for utilization percentage.
```

---

### Mistake 2 — metrics-server မရှိဘဲ HPA create လုပ်ခြင်း

HPA object create ဖြစ်နိုင်ပေမယ့် metrics မရရင် scale decision မလုပ်နိုင်ပါဘူး။

Check:

```bash
kubectl top pods
```

---

### Mistake 3 — min/max replicas မသတ်မှတ်ခြင်း

Imperative command မှာ min/max သတ်မှတ်ပါ။

```bash
--min=1 --max=10
```

---

### Mistake 4 — Manual scale နဲ့ HPA ကိုရောပြီး manage လုပ်ခြင်း

HPA ရှိနေတဲ့ Deployment ကို replicas manual ပြောင်းရင် HPA က ပြန် override လုပ်နိုင်ပါတယ်။

---

## 30. Practice Example

### Task

Deployment `my-app` ကို CPU average utilization 50% ထိန်းပြီး replicas 1 မှ 10 အထိ automatic scale ဖြစ်အောင် HPA create လုပ်ပါ။

### Solution — Imperative

```bash
kubectl autoscale deployment my-app --cpu-percent=50 --min=1 --max=10
```

Check:

```bash
kubectl get hpa
kubectl describe hpa my-app
```

---

### Solution — YAML

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  minReplicas: 1
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 50
```

Apply:

```bash
kubectl apply -f hpa.yaml
```

---

## 31. Quick Reference

```bash
# Check metrics
kubectl top pods

# Manual scale
kubectl scale deploy my-app --replicas=3

# Create HPA
kubectl autoscale deploy my-app --cpu-percent=50 --min=1 --max=10

# View HPA
kubectl get hpa

# Detailed HPA info
kubectl describe hpa my-app

# Delete HPA
kubectl delete hpa my-app
```

---

## 32. Key Takeaways

- HPA က workload replicas ကို automatic scale လုပ်ပေးပါတယ်။
- HPA သည် horizontal scaling ဖြစ်ပြီး Pod count ကိုတိုး/လျှော့လုပ်ပါတယ်။
- HPA က CPU, memory, custom metrics, external metrics တွေကိုအသုံးပြုနိုင်ပါတယ်။
- CPU/Memory metrics အတွက် metrics-server လိုပါတယ်။
- `kubectl autoscale` နဲ့ HPA ကို command line ကနေ create လုပ်နိုင်ပါတယ်။
- `autoscaling/v2` API နဲ့ HPA YAML ရေးနိုင်ပါတယ်။
- HPA က minReplicas နဲ့ maxReplicas range ထဲမှာပဲ scale လုပ်ပါတယ်။
- CPU utilization percentage က CPU request အပေါ်မူတည်ပါတယ်။
