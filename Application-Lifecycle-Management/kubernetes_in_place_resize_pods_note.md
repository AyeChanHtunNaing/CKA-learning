# In-place Resize of Pods

> Topic: Pod resources ကို Pod မဖျက်ဘဲ in-place update လုပ်ခြင်း

---

## 1. Overview

**In-place Resize of Pods** ဆိုတာ Kubernetes Pod ထဲက container resource requests/limits တွေကို Pod ကို delete/recreate မလုပ်ဘဲ update လုပ်နိုင်တဲ့ feature ဖြစ်ပါတယ်။

ဒီ feature ရဲ့အဓိကရည်ရွယ်ချက်က —

- Pod downtime လျှော့ချရန်
- Stateful workloads တွေမှာ disruption လျှော့ရန်
- CPU/Memory resource changes တွေကို running Pod ပေါ်မှာပဲ update လုပ်ရန်

---

## 2. Default Behavior

Kubernetes မှာ normally Deployment တစ်ခုရဲ့ Pod template ထဲက resource requests/limits ပြောင်းလိုက်ရင် Pod အသစ် recreate ဖြစ်ပါတယ်။

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
              memory: "256Mi"
            limits:
              cpu: "500m"
              memory: "512Mi"
```

ဒီ Deployment မှာ resource values တွေပြောင်းလိုက်ရင် Kubernetes က —

1. Old Pod ကို terminate လုပ်မယ်
2. New Pod ကို updated resource spec နဲ့ create လုပ်မယ်

ဒါကြောင့် temporary disruption ဖြစ်နိုင်ပါတယ်။

---

## 3. Why In-place Resize is Useful

In-place resize က Pod ကို recreate မလုပ်ဘဲ resource ကို update လုပ်နိုင်အောင်လုပ်ပေးပါတယ်။

အသုံးဝင်တဲ့ case တွေ —

- Stateful workloads
- Long-running applications
- Pod restart မဖြစ်စေချင်တဲ့ workload
- CPU resource ပိုလိုလာတဲ့ app
- Downtime နည်းစေချင်တဲ့ production workload

---

## 4. Default Resource Update vs In-place Resize

| Method          | Behavior                         | Downtime              |
| --------------- | -------------------------------- | --------------------- |
| Default update  | Pod ကို recreate လုပ်            | ရှိနိုင်              |
| In-place resize | Running Pod ပေါ် resource update | နည်းနိုင် / မရှိနိုင် |

---

## 5. In-place Pod Vertical Scaling Feature

In-place resize feature ကို **InPlacePodVerticalScaling** feature gate နဲ့ enable လုပ်ရပါတယ်။

ဒီ feature က Kubernetes 1.27 မှာ alpha အဖြစ်စတင်ပါဝင်ခဲ့ပြီး default enabled မဟုတ်ပါဘူး။

Enable feature gate:

```bash
FEATURE_GATES=InPlacePodVerticalScaling=true
```

> Real cluster မှာ feature gate ကို kubelet / kube-apiserver configuration မှာ enable လုပ်ရနိုင်ပါတယ်။ Lab environment ပေါ်မူတည်ပြီး command/config ကွာနိုင်ပါတယ်။

---

## 6. Example — CPU Resource Increase

CPU request ကို `250m` ကနေ `1` သို့တိုးချင်တယ်ဆိုပါစို့။

Before:

```yaml
resources:
  requests:
    cpu: "250m"
    memory: "256Mi"
  limits:
    cpu: "500m"
    memory: "512Mi"
```

After:

```yaml
resources:
  requests:
    cpu: "1"
    memory: "256Mi"
  limits:
    cpu: "500m"
    memory: "512Mi"
```

In-place resize feature enabled ဖြစ်ရင် CPU change ကို Pod မဖျက်ဘဲ update လုပ်နိုင်ပါတယ်။

---

## 7. Example Deployment with Updated CPU

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
              cpu: "1"
              memory: "256Mi"
            limits:
              cpu: "500m"
              memory: "512Mi"
```

ဒီ example မှာ CPU request ကို `250m` ကနေ `1` သို့ပြောင်းထားပါတယ်။

---

## 8. Resize Policy ဆိုတာဘာလဲ?

`resizePolicy` က resource resize လုပ်တဲ့အခါ container restart လို/မလို control လုပ်ဖို့သုံးပါတယ်။

ဥပမာ CPU resource update လုပ်တဲ့အခါ restart မလိုချင်ရင် —

```yaml
resizePolicy:
  - resourceName: cpu
    restartPolicy: NotRequired
```

ဒီ policy က CPU resource resize လုပ်တဲ့အခါ container restart မလိုဘူးလို့ပြောတာပါ။

---

## 9. Resize Policy Example

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
          resizePolicy:
            - resourceName: cpu
              restartPolicy: NotRequired
          resources:
            requests:
              cpu: "250m"
              memory: "256Mi"
            limits:
              cpu: "500m"
              memory: "512Mi"
```

ဒီ manifest မှာ —

- CPU resource resize ဖြစ်ရင် restart မလိုပါ
- Memory resource အတွက် policy မသတ်မှတ်ထားပါ

---

## 10. resizePolicy Fields

| Field           | Meaning                                  |
| --------------- | ---------------------------------------- |
| `resourceName`  | resize policy သက်ရောက်မယ့် resource name |
| `restartPolicy` | resize လုပ်တဲ့အခါ restart လို/မလို       |

Example:

```yaml
resizePolicy:
  - resourceName: cpu
    restartPolicy: NotRequired
```

---

## 11. Supported Resources

In-place resize feature က current implementation အရ အဓိက resource ၂ မျိုးကို support လုပ်ပါတယ်။

| Resource | Supported |
| -------- | --------- |
| CPU      | Yes       |
| Memory   | Yes       |

GPU or custom extended resources တွေကို in-place resize မလုပ်နိုင်ပါ။

---

## 12. CPU Resize Behavior

CPU resource change က usually Pod restart မလိုဘဲ update လုပ်နိုင်ပါတယ်။

ဥပမာ —

```yaml
resizePolicy:
  - resourceName: cpu
    restartPolicy: NotRequired
```

CPU request ကို `250m` ကနေ `1` သို့တိုးလိုက်ရင် Pod ကို terminate မလုပ်ဘဲ update လုပ်နိုင်ပါတယ်။

---

## 13. Memory Resize Behavior

Memory resource change မှာ restart လိုနိုင်တဲ့ case တွေရှိပါတယ်။

အရေးကြီးတဲ့ limitation:

```text
Container memory limit ကို current memory usage ထက်နည်းအောင် reduce လုပ်လို့မရနိုင်ပါ။
```

ဥပမာ container က memory `400Mi` သုံးနေပြီး memory limit ကို `256Mi` သို့လျှော့ချင်ရင် resize operation က pending/in progress ဖြစ်နိုင်ပါတယ်။

---

## 14. Limitations

In-place Pod resizing မှာ limitations တွေရှိပါတယ်။

1. CPU နဲ့ memory resources ပဲ in-place update လုပ်နိုင်ပါတယ်။
2. Pod QoS class ပြောင်းသွားစေမယ့် changes တွေကို support မလုပ်ပါ။
3. Init containers တွေကို in-place resize မလုပ်နိုင်ပါ။
4. Ephemeral containers တွေကို in-place resize မလုပ်နိုင်ပါ။
5. Container တစ်ခုရဲ့ resource ကို container နောက်တစ်ခုဆီ shift လုပ်လို့မရပါ။
6. Memory limit ကို current usage ထက်နည်းအောင် reduce လုပ်လို့မရနိုင်ပါ။
7. Windows Pods တွေကို support မလုပ်ပါ။

---

## 15. QoS Class Limitation

Kubernetes Pod QoS classes တွေက —

- Guaranteed
- Burstable
- BestEffort

In-place resize change ကြောင့် Pod QoS class ပြောင်းသွားမယ်ဆိုရင် support မလုပ်နိုင်ပါ။

ဥပမာ BestEffort Pod ကို resource requests/limits ထည့်ပြီး Burstable ဖြစ်အောင်ပြောင်းတာမျိုးက in-place resize နဲ့မကိုက်နိုင်ပါ။

---

## 16. Init Containers and Ephemeral Containers

In-place resize feature က normal application containers အတွက်သာ mainly အသုံးပြုပါတယ်။

မ support လုပ်တာတွေ —

- Init containers
- Ephemeral containers

---

## 17. In-place Resize vs VPA

In-place resize က **manual resource change** ကို Pod မ recreate လုပ်ဘဲ update လုပ်နိုင်အောင်လုပ်ပေးတာပါ။

VPA ကတော့ resource usage ကို monitor လုပ်ပြီး Pod resource requests/limits ကို automatic recommend/adjust လုပ်ပေးတဲ့ autoscaler ဖြစ်ပါတယ်။

| Feature         | Purpose                                                     |
| --------------- | ----------------------------------------------------------- |
| In-place Resize | Running Pod resource ကို recreate မလုပ်ဘဲ update            |
| VPA             | Pod resource requests/limits ကို automatic adjust/recommend |

---

## 18. Vertical Pod Autoscaler Connection

VPA က Pod ရဲ့ CPU/Memory usage ကို monitor လုပ်ပြီး resource requests/limits ကို adjust လုပ်နိုင်ပါတယ်။

In-place resize feature mature ဖြစ်လာရင် VPA နဲ့တွဲပြီး Pod recreation လျှော့ချနိုင်ပါတယ်။

ဥပမာ —

```text
VPA detects Pod needs more CPU
        ↓
Resource request updated
        ↓
In-place resize applies update without full Pod recreation
```

---

## 19. Useful Commands

### Deployment ကို edit လုပ်ရန်

```bash
kubectl edit deployment my-app
```

### Deployment YAML apply

```bash
kubectl apply -f deployment.yaml
```

### Pod status ကြည့်ရန်

```bash
kubectl get pods
```

### Pod details ကြည့်ရန်

```bash
kubectl describe pod <pod-name>
```

### Deployment rollout status

```bash
kubectl rollout status deployment/my-app
```

Short form:

```bash
kubectl rollout status deploy/my-app
```

---

## 20. Check Pod Resource Requests and Limits

Pod YAML ထဲက resources ကြည့်ရန်:

```bash
kubectl get pod <pod-name> -o yaml
```

Specific resources ကို jsonpath နဲ့ကြည့်ရန်:

```bash
kubectl get pod <pod-name> -o jsonpath='{.spec.containers[0].resources}'
```

---

## 21. Example Lab Flow

```text
1. Create Deployment with CPU request 250m
2. Check Pod is running
3. Enable InPlacePodVerticalScaling feature gate
4. Add resizePolicy for CPU
5. Update CPU request from 250m to 1
6. Apply updated YAML
7. Check Pod was not recreated
8. Verify resource changes
```

---

## 22. How to Check Pod Was Not Recreated

Pod recreate ဖြစ်/မဖြစ် သိချင်ရင် Pod age/name ကိုစစ်ပါ။

Before update:

```bash
kubectl get pods
```

After update:

```bash
kubectl get pods
```

Pod name/age မပြောင်းဘူးဆိုရင် Pod recreation မဖြစ်တာကိုညွှန်ပြနိုင်ပါတယ်။

Detailed check:

```bash
kubectl describe pod <pod-name>
```

---

## 23. CKA Exam Tips

### Tip 1 — Default behavior ကိုမှတ်ပါ

```text
Resource requests/limits update → Pod recreate ဖြစ်နိုင်
```

---

### Tip 2 — In-place resize purpose

```text
In-place resize = Pod မဖျက်ဘဲ CPU/Memory resource update
```

---

### Tip 3 — Feature gate name

```text
InPlacePodVerticalScaling
```

---

### Tip 4 — resizePolicy example

```yaml
resizePolicy:
  - resourceName: cpu
    restartPolicy: NotRequired
```

---

### Tip 5 — Supported resources

```text
Only CPU and memory
```

---

## 24. Common Mistakes

### Mistake 1 — In-place resize ကို default enabled လို့ထင်ခြင်း

ဒီ feature က alpha state မှာ default enabled မဟုတ်နိုင်ပါဘူး။  
Feature gate enable လုပ်ဖို့လိုနိုင်ပါတယ်။

---

### Mistake 2 — Pod QoS class change ကို support လုပ်တယ်လို့ထင်ခြင်း

QoS class ပြောင်းစေတဲ့ changes တွေကို support မလုပ်နိုင်ပါ။

---

### Mistake 3 — Memory limit ကို current usage ထက်လျှော့ခြင်း

Container က memory 많이 သုံးနေချိန်မှာ memory limit ကို current usage ထက်နည်းအောင်လျှော့လို့မရနိုင်ပါ။

---

### Mistake 4 — Init container ကို resize လုပ်ဖို့ကြိုးစားခြင်း

Init containers နဲ့ ephemeral containers တွေကို in-place resize မလုပ်နိုင်ပါ။

---

## 25. Practice Example

### Task

Deployment `my-app` မှာ CPU resize ကို restart မလိုအောင် `resizePolicy` ထည့်ပါ။

### Solution

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
          resizePolicy:
            - resourceName: cpu
              restartPolicy: NotRequired
          resources:
            requests:
              cpu: "250m"
              memory: "256Mi"
            limits:
              cpu: "500m"
              memory: "512Mi"
```

Apply:

```bash
kubectl apply -f deployment.yaml
```

Update CPU request:

```yaml
resources:
  requests:
    cpu: "1"
    memory: "256Mi"
```

Apply again:

```bash
kubectl apply -f deployment.yaml
```

Check:

```bash
kubectl get pods
kubectl describe pod <pod-name>
```

---

## 26. Quick Reference

| Task                  | Example                                        |
| --------------------- | ---------------------------------------------- |
| Enable feature gate   | `FEATURE_GATES=InPlacePodVerticalScaling=true` |
| CPU resize no restart | `restartPolicy: NotRequired`                   |
| Edit deployment       | `kubectl edit deployment my-app`               |
| Apply YAML            | `kubectl apply -f deployment.yaml`             |
| Check Pod resources   | `kubectl get pod <pod-name> -o yaml`           |
| Check Pod recreation  | `kubectl get pods`                             |

---

## 27. Key Takeaways

- In-place resize က Pod resources ကို Pod recreate မလုပ်ဘဲ update လုပ်နိုင်တဲ့ feature ဖြစ်ပါတယ်။
- Default behavior မှာ resource changes လုပ်ရင် Pod recreate ဖြစ်နိုင်ပါတယ်။
- In-place resize က downtime လျှော့ချဖို့အသုံးဝင်ပါတယ်။
- Feature gate `InPlacePodVerticalScaling` enable လုပ်ဖို့လိုနိုင်ပါတယ်။
- `resizePolicy` နဲ့ resource update အတွက် restart လို/မလို control လုပ်နိုင်ပါတယ်။
- CPU နဲ့ memory resources ကိုပဲ support လုပ်ပါတယ်။
- QoS class changes, init containers, ephemeral containers, Windows Pods တွေကို support မလုပ်ပါ။
- VPA နဲ့ပေါင်းပြီး future မှာ vertical scaling ကို ပို smooth လုပ်နိုင်ပါတယ်။
