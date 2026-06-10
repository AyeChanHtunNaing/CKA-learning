# Vertical Pod Autoscaling VPA

> Topic: Kubernetes Vertical Pod Autoscaler (VPA) ကိုသုံးပြီး Pod resource requests/limits ကို automatic optimize လုပ်ခြင်း

---

## 1. Overview

**Vertical Pod Autoscaler (VPA)** ဆိုတာ Kubernetes workload တွေရဲ့ Pod resource allocation ကို automatic optimize လုပ်ပေးတဲ့ autoscaler ဖြစ်ပါတယ်။

VPA က application ရဲ့ CPU/Memory usage ကို monitor လုပ်ပြီး Pod တွေအတွက် သင့်တော်တဲ့ resource requests/limits ကို recommend သို့မဟုတ် update လုပ်ပေးနိုင်ပါတယ်။

VPA ရဲ့ အဓိကရည်ရွယ်ချက် —

- Pod တစ်ခုချင်းစီအတွက် CPU/Memory resources ကို optimize လုပ်ရန်
- Over-provisioning လျှော့ရန်
- Under-provisioning ကြောင့် app slow/crash ဖြစ်တာကိုကာကွယ်ရန်
- Manual resource tuning ကိုလျှော့ရန်

---

## 2. VPA vs HPA Basic Difference

Kubernetes autoscaling မှာ HPA နဲ့ VPA က အဓိကကွာပါတယ်။

| Autoscaler | What it scales                 |
| ---------- | ------------------------------ |
| HPA        | Pod replicas အရေအတွက်          |
| VPA        | Pod CPU/Memory requests/limits |

အလွယ်မှတ်ရန် —

```text
HPA = Pod count ကို scale လုပ်တယ်
VPA = Pod resource size ကို scale လုပ်တယ်
```

---

## 3. Example Deployment with Resource Requests and Limits

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

- CPU request = `250m`
- CPU limit = `500m`

Container က CPU `500m` ထက်ပိုမသုံးနိုင်ပါဘူး။

---

## 4. Monitor Pod Resource Usage

Pod resource usage ကြည့်ရန်:

```bash
kubectl top pod my-app-pod
```

Example output:

```text
NAME         CPU(cores)   MEMORY(bytes)
my-app-pod   450m         350Mi
```

ဒီ output မှာ Pod က CPU `450m` နဲ့ Memory `350Mi` သုံးနေပါတယ်။

> `kubectl top` အလုပ်လုပ်ဖို့ metrics-server install ဖြစ်ရပါမယ်။

---

## 5. Manual Vertical Scaling

Pod CPU usage များလာရင် admin က Deployment resource spec ကို manually edit လုပ်နိုင်ပါတယ်။

```bash
kubectl edit deployment my-app
```

ဥပမာ CPU request ကို `250m` ကနေ `1` သို့တိုးခြင်း:

```yaml
resources:
  requests:
    cpu: "1"
  limits:
    cpu: "500m"
```

ဒါပေမယ့် Deployment template ပြောင်းတာကြောင့် Kubernetes က current Pod ကို terminate လုပ်ပြီး updated resource နဲ့ Pod အသစ် create လုပ်နိုင်ပါတယ်။

---

## 6. Problem with Manual Vertical Scaling

Manual vertical scaling ရဲ့ အားနည်းချက်တွေက —

- Admin က metrics ကိုအမြဲစောင့်ကြည့်ရတယ်
- Resource value မှန်မှန်ခန့်မှန်းရခက်တယ်
- Manual edit မှားနိုင်တယ်
- Pod restart/recreate ဖြစ်နိုင်တယ်
- Stateful workloads တွေမှာ disruption ဖြစ်နိုင်တယ်

ဒါကြောင့် VPA ကိုသုံးပြီး resource tuning ကို automate လုပ်နိုင်ပါတယ်။

---

## 7. Vertical Pod Autoscaler (VPA) ဆိုတာဘာလဲ?

VPA က Pod တွေရဲ့ CPU/Memory usage history နဲ့ current metrics ကိုကြည့်ပြီး resource requests/limits ကို recommend or adjust လုပ်ပေးတဲ့ Kubernetes autoscaler ဖြစ်ပါတယ်။

VPA က —

- CPU request recommend လုပ်နိုင်တယ်
- Memory request recommend လုပ်နိုင်တယ်
- Policy ပေါ်မူတည်ပြီး Pod ကို update လုပ်နိုင်တယ်
- Pod အသစ် create ဖြစ်တဲ့အခါ recommended values inject လုပ်နိုင်တယ်

---

## 8. VPA is Not Enabled by Default

VPA က Kubernetes core default installation ထဲမှာ automatically enabled မဟုတ်ပါဘူး။  
Manually install လုပ်ရပါတယ်။

Install command:

```bash
kubectl apply -f https://github.com/kubernetes/autoscaler/releases/latest/download/vertical-pod-autoscaler.yaml
```

---

## 9. Verify VPA Components

VPA install ပြီးရင် `kube-system` namespace ထဲမှာ VPA components တွေ running ဖြစ်မဖြစ်စစ်ပါ။

```bash
kubectl get pods -n kube-system | grep vpa
```

Expected output:

```text
vpa-admission-controller-xxxx   Running
vpa-recommender-xxxx            Running
vpa-updater-xxxx                Running
```

---

## 10. VPA Components

VPA မှာ အဓိက component ၃ ခုရှိပါတယ်။

1. VPA Recommender
2. VPA Updater
3. VPA Admission Controller

---

## 11. VPA Recommender

**VPA Recommender** က metrics ကို monitor လုပ်ပြီး resource recommendation တွေတွက်ပေးပါတယ်။

လုပ်ဆောင်ချက် —

- Kubernetes metrics API က data ယူတယ်
- Historical usage နဲ့ live usage ကို analyze လုပ်တယ်
- CPU/Memory recommendation ထုတ်ပေးတယ်

ဥပမာ recommendation:

```text
Recommended CPU: 1.5
Recommended Memory: 512Mi
```

---

## 12. VPA Updater

**VPA Updater** က current Pod resources နဲ့ recommender recommendation ကို compare လုပ်ပါတယ်။

Current Pod resource settings မသင့်တော်တော့ဘူးဆိုရင် Pod ကို evict လုပ်နိုင်ပါတယ်။

Eviction ဖြစ်တဲ့အခါ —

1. Old Pod terminate ဖြစ်မယ်
2. New Pod create ဖြစ်မယ်
3. New Pod မှာ recommended resource values ပါလာမယ်

ဒါကြောင့် VPA Auto mode မှာ Pod restart/recreate ဖြစ်နိုင်ပါတယ်။

---

## 13. VPA Admission Controller

**VPA Admission Controller** က Pod creation request ကို intercept လုပ်ပြီး recommender ရဲ့ suggestion အပေါ်မူတည်ကာ Pod spec ထဲ resource values mutate လုပ်ပေးပါတယ်။

ဆိုလိုတာက —

- New Pod create ဖြစ်တဲ့အခါ
- VPA recommendation ကို Pod spec ထဲ inject လုပ်ပေးနိုင်တယ်
- Pod က optimal resource values နဲ့စတင် run နိုင်တယ်

---

## 14. VPA Architecture Flow

```text
Metrics Server / Metrics API
        ↓
VPA Recommender
        ↓
VPA Recommendation
        ↓
VPA Updater evicts old Pod if needed
        ↓
VPA Admission Controller mutates new Pod
        ↓
New Pod runs with recommended resources
```

---

## 15. Create VPA Resource

VPA ကို imperative command နဲ့ create လုပ်တာမဟုတ်ဘဲ YAML file နဲ့ create လုပ်ပါတယ်။

Example VPA YAML:

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: my-app-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  updatePolicy:
    updateMode: "Auto"
  resourcePolicy:
    containerPolicies:
      - containerName: "my-app"
        minAllowed:
          cpu: "250m"
        maxAllowed:
          cpu: "2"
        controlledResources: ["cpu"]
```

Apply:

```bash
kubectl apply -f vpa.yaml
```

---

## 16. VPA YAML Explanation

| Field                               | Meaning                                      |
| ----------------------------------- | -------------------------------------------- |
| `apiVersion: autoscaling.k8s.io/v1` | VPA API version                              |
| `kind: VerticalPodAutoscaler`       | VPA resource                                 |
| `targetRef`                         | VPA က manage လုပ်မယ့် workload               |
| `updatePolicy`                      | VPA က recommendation ကို apply လုပ်မယ့် mode |
| `resourcePolicy`                    | Resource limits/bounds သတ်မှတ်ရန်            |
| `containerPolicies`                 | Container တစ်ခုချင်းစီအတွက် policy           |
| `minAllowed`                        | Minimum allowed resource                     |
| `maxAllowed`                        | Maximum allowed resource                     |
| `controlledResources`               | VPA က control လုပ်မယ့် resources             |

---

## 17. targetRef

`targetRef` က VPA ဘယ် workload ကို monitor/manage လုပ်မလဲဆိုတာသတ်မှတ်ပါတယ်။

```yaml
targetRef:
  apiVersion: apps/v1
  kind: Deployment
  name: my-app
```

ဒီ example မှာ VPA က `my-app` Deployment ကို target လုပ်ပါတယ်။

---

## 18. updatePolicy

`updatePolicy` က VPA recommendation ကို ဘယ်လို apply လုပ်မလဲဆိုတာသတ်မှတ်ပါတယ်။

```yaml
updatePolicy:
  updateMode: "Auto"
```

Common update modes:

| Mode       | Meaning                                          |
| ---------- | ------------------------------------------------ |
| `Off`      | Recommendation ပဲပေးမယ်၊ update မလုပ်            |
| `Initial`  | Pod create ဖြစ်တဲ့အချိန်ပဲ resource set လုပ်     |
| `Auto`     | Recommendation အရ Pod ကို evict/update လုပ်နိုင် |
| `Recreate` | Pod recreate လုပ်ပြီး update                     |

---

## 19. Auto Mode

`Auto` mode မှာ VPA updater က suboptimal resource နဲ့ run နေတဲ့ Pod တွေကို evict လုပ်နိုင်ပါတယ်။

Flow:

```text
VPA sees better resource recommendation
        ↓
Updater evicts current Pod
        ↓
ReplicaSet/Deployment creates new Pod
        ↓
Admission Controller injects recommended resources
```

Important:

Auto mode မှာ temporary downtime/restart ဖြစ်နိုင်ပါတယ်။

---

## 20. resourcePolicy

`resourcePolicy` က VPA က resource ကို ဘယ် range ထဲမှာပဲ recommend/apply လုပ်ရမလဲဆိုတာသတ်မှတ်ပါတယ်။

Example:

```yaml
resourcePolicy:
  containerPolicies:
    - containerName: "my-app"
      minAllowed:
        cpu: "250m"
      maxAllowed:
        cpu: "2"
      controlledResources: ["cpu"]
```

ဒီမှာ —

- CPU minimum = `250m`
- CPU maximum = `2`
- VPA က CPU ကိုပဲ control လုပ်မယ်

---

## 21. controlledResources

VPA က CPU နဲ့ Memory နှစ်မျိုးလုံး control လုပ်နိုင်ပါတယ်။

CPU only:

```yaml
controlledResources: ["cpu"]
```

Memory only:

```yaml
controlledResources: ["memory"]
```

CPU and Memory:

```yaml
controlledResources: ["cpu", "memory"]
```

---

## 22. Check VPA Recommendations

VPA recommendation ကြည့်ရန်:

```bash
kubectl describe vpa my-app-vpa
```

Example output:

```text
Recommendations:
  Target:
    Cpu: 1.5
```

ဒီမှာ VPA က CPU target recommendation ကို `1.5` လို့ recommend လုပ်ထားပါတယ်။

---

## 23. VPA and Pod Recreation

Current implementation မှာ VPA Auto mode က Pod resource values ပြောင်းဖို့ Pod ကို evict/recreate လုပ်နိုင်ပါတယ်။

ဒါကြောင့် VPA က HPA လို seamless scale-out မဟုတ်ပါဘူး။

Future Kubernetes in-place resize feature mature ဖြစ်လာရင် VPA က Pod restart မလိုဘဲ resources update လုပ်နိုင်လာနိုင်ပါတယ်။

---

## 24. VPA and In-place Resize

In-place Pod resize feature နဲ့ VPA ကိုတွဲသုံးနိုင်လာရင် —

```text
VPA recommends new resources
        ↓
Pod resources updated in-place
        ↓
Pod restart/recreation မလိုနိုင်
```

ဒါက future vertical scaling ကို ပို smooth ဖြစ်စေပါမယ်။

---

## 25. HPA vs VPA Comparison

![VPA vs HPA Comparison](https://kodekloud.com/kk-media/image/upload/v1752869688/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-Vertical-Pod-Autoscaling-VPA-2025-Updates/vpa-hpa-comparison-chart-kubernetes.jpg)

| Feature                | VPA                                       | HPA                                         |
| ---------------------- | ----------------------------------------- | ------------------------------------------- |
| Scaling Method         | Pod CPU/Memory settings ကို adjust        | Pod replicas အရေအတွက်တိုး/လျှော့            |
| What changes           | Resource requests/limits                  | Replicas count                              |
| Pod behavior           | Pod restart/recreate ဖြစ်နိုင်            | Existing Pods မထိဘဲ new Pods ထည့်နိုင်      |
| Traffic spike handling | Sudden spike အတွက်နည်းနည်းနှေးနိုင်       | Rapid traffic spikes အတွက်သင့်တော်          |
| Cost optimization      | Over-provisioning လျှော့နိုင်             | Underused Pods လျှော့နိုင်                  |
| Best for               | Stateful workloads, DB, JVM, AI workloads | Stateless apps, web services, microservices |

---

## 26. When to Use VPA

VPA သုံးသင့်တဲ့ workloads —

- Stateful applications
- Databases
- JVM-based applications
- AI/ML workloads
- Resource usage ခန့်မှန်းရခက်တဲ့ apps
- Pod တစ်ခုချင်း resource tuning လိုတဲ့ workloads

---

## 27. When to Use HPA

HPA သုံးသင့်တဲ့ workloads —

- Stateless web apps
- Microservices
- Traffic spike များတဲ့ apps
- Request count များလာရင် Pod replicas တိုးချင်တဲ့ apps
- Existing Pod restart မဖြစ်စေချင်တဲ့ apps

---

## 28. Can HPA and VPA Be Used Together?

HPA နဲ့ VPA ကို သုံးလို့ရပေမယ့် သတိထားရပါမယ်။

Important:

- HPA က CPU utilization percentage ပေါ်မူတည်ပြီး scale လုပ်တယ်
- VPA က CPU request ကိုပြောင်းနိုင်တယ်
- CPU request ပြောင်းသွားရင် HPA calculation ပေါ်သက်ရောက်နိုင်တယ်

ဒါကြောင့် HPA နဲ့ VPA ကို CPU metric တူတူ control လုပ်စေခြင်းက conflict ဖြစ်နိုင်ပါတယ်။

သုံးချင်ရင် —

- HPA ကို custom/external metric နဲ့ scale လုပ်
- VPA ကို CPU/Memory request recommendation အတွက်သုံး

ဆိုတာပိုကောင်းပါတယ်။

---

## 29. Important Commands

| Task                     | Command                                                                                                           |
| ------------------------ | ----------------------------------------------------------------------------------------------------------------- |
| Check Pod metrics        | `kubectl top pod <pod-name>`                                                                                      |
| Install VPA              | `kubectl apply -f https://github.com/kubernetes/autoscaler/releases/latest/download/vertical-pod-autoscaler.yaml` |
| Check VPA pods           | `kubectl get pods -n kube-system \| grep vpa`                                                                     |
| Apply VPA YAML           | `kubectl apply -f vpa.yaml`                                                                                       |
| Describe VPA             | `kubectl describe vpa <vpa-name>`                                                                                 |
| Edit deployment manually | `kubectl edit deployment <deployment-name>`                                                                       |
| Get VPA                  | `kubectl get vpa`                                                                                                 |
| Delete VPA               | `kubectl delete vpa <vpa-name>`                                                                                   |

---

## 30. CKA Exam Tips

### Tip 1 — VPA meaning ကိုမှတ်ပါ

```text
VPA = Vertical Pod Autoscaler
VPA adjusts Pod CPU/Memory resources
```

---

### Tip 2 — VPA components ၃ ခုမှတ်ပါ

```text
Recommender
Updater
Admission Controller
```

---

### Tip 3 — VPA YAML မှာ targetRef ပါရမယ်

```yaml
targetRef:
  apiVersion: apps/v1
  kind: Deployment
  name: my-app
```

---

### Tip 4 — VPA ကို imperative autoscale command နဲ့မလုပ်ပါ

HPA:

```bash
kubectl autoscale deploy my-app --cpu-percent=50 --min=1 --max=10
```

VPA:

```bash
kubectl apply -f vpa.yaml
```

---

### Tip 5 — Auto mode မှာ Pod recreate ဖြစ်နိုင်

```text
VPA Auto mode may evict/recreate Pods.
```

---

## 31. Common Mistakes

### Mistake 1 — VPA က Pod replicas တိုးတယ်လို့ထင်ခြင်း

မမှန်:

```text
VPA increases number of Pods.
```

မှန်:

```text
VPA adjusts CPU/Memory requests/limits.
```

---

### Mistake 2 — VPA default installed လို့ထင်ခြင်း

VPA က default enabled မဟုတ်ပါဘူး။ Install လုပ်ဖို့လိုပါတယ်။

---

### Mistake 3 — HPA and VPA both on CPU metric

HPA က CPU utilization ကိုသုံးပြီး VPA က CPU request ကိုပြောင်းနေရင် scaling behavior မထင်မှတ်တာတွေဖြစ်နိုင်ပါတယ်။

---

### Mistake 4 — Auto mode မှာ downtime မရှိဘူးလို့ထင်ခြင်း

VPA Auto mode က Pod evict/recreate လုပ်နိုင်တာကြောင့် downtime/restart ဖြစ်နိုင်ပါတယ်။

---

## 32. Practice Example

### Task

Deployment `my-app` အတွက် VPA create လုပ်ပါ။

Requirements:

- VPA name: `my-app-vpa`
- Target: Deployment `my-app`
- Update mode: `Auto`
- Controlled resource: CPU
- minAllowed CPU: `250m`
- maxAllowed CPU: `2`

### Solution

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: my-app-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  updatePolicy:
    updateMode: "Auto"
  resourcePolicy:
    containerPolicies:
      - containerName: "my-app"
        minAllowed:
          cpu: "250m"
        maxAllowed:
          cpu: "2"
        controlledResources: ["cpu"]
```

Apply:

```bash
kubectl apply -f vpa.yaml
```

Check:

```bash
kubectl get vpa
kubectl describe vpa my-app-vpa
```

---

## 33. Quick Reference

```bash
# Check metrics
kubectl top pod my-app-pod

# Install VPA
kubectl apply -f https://github.com/kubernetes/autoscaler/releases/latest/download/vertical-pod-autoscaler.yaml

# Check VPA components
kubectl get pods -n kube-system | grep vpa

# Apply VPA
kubectl apply -f vpa.yaml

# Get VPA
kubectl get vpa

# Describe VPA recommendation
kubectl describe vpa my-app-vpa

# Delete VPA
kubectl delete vpa my-app-vpa
```

---

## 34. Key Takeaways

- VPA က Pod CPU/Memory resource allocation ကို optimize လုပ်ပေးပါတယ်။
- HPA က Pod count ကို scale လုပ်ပြီး VPA က Pod resource size ကို scale လုပ်ပါတယ်။
- VPA က default enabled မဟုတ်ပါဘူး။ Install လုပ်ရပါတယ်။
- VPA မှာ Recommender, Updater, Admission Controller ဆိုတဲ့ components ၃ ခုရှိပါတယ်။
- VPA YAML မှာ `targetRef`, `updatePolicy`, `resourcePolicy` တွေကိုသုံးပါတယ်။
- `Auto` mode မှာ Pod eviction/recreation ဖြစ်နိုင်ပါတယ်။
- VPA recommendation ကို `kubectl describe vpa <name>` နဲ့ကြည့်နိုင်ပါတယ်။
- HPA နဲ့ VPA ကို တူညီတဲ့ CPU metric ပေါ်မှာအတူသုံးရင် conflict ဖြစ်နိုင်ပါတယ်။
