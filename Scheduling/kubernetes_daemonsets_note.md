# Kubernetes DaemonSets Note

## 1. DaemonSet ဆိုတာဘာလဲ

**DaemonSet** ဆိုတာ Kubernetes မှာ Cluster ထဲရှိ **Node တိုင်းပေါ်မှာ Pod တစ်ခုစီ အလိုအလျောက် run စေဖို့** သုံးတဲ့ workload controller ပါ။

အဓိက concept က -

```text
Every Node = One Pod
```

ဆိုတဲ့ idea ဖြစ်ပါတယ်။

ဥပမာ Cluster ထဲမှာ Node ၃ ခုရှိရင် DaemonSet က Pod ၃ ခု run စေပါမယ်။

```text
Node 1 -> DaemonSet Pod 1
Node 2 -> DaemonSet Pod 2
Node 3 -> DaemonSet Pod 3
```

Node အသစ်ထပ်ထည့်ရင်လည်း DaemonSet က အဲ့ဒီ Node ပေါ်မှာ Pod အသစ်တစ်ခုကို အလိုအလျောက် create လုပ်ပေးပါတယ်။

Node တစ်ခု remove လုပ်လိုက်ရင် အဲ့ဒီ Node ပေါ်က DaemonSet Pod လည်း remove ဖြစ်သွားပါတယ်။

![DaemonSet Overview](https://kodekloud.com/kk-media/image/upload/v1752869886/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-DaemonSets/frame_40.jpg)

---

## 2. DaemonSet ကို ဘာကြောင့်သုံးလဲ

DaemonSet ကို Node တိုင်းမှာ run ဖို့လိုတဲ့ background service / agent တွေအတွက် သုံးပါတယ်။

အသုံးများတဲ့ use cases တွေက -

- Monitoring agents
- Log collectors
- kube-proxy
- Networking agents
- CNI plugins เช่น weave-net

---

## 3. Use Case 1: Monitoring Agent and Log Collector

Cluster ထဲက Node တိုင်းရဲ့ metrics နဲ့ logs တွေကို collect လုပ်ချင်ရင် monitoring agent သို့မဟုတ် log collector ကို Node တိုင်းမှာ run ရပါမယ်။

ဒီလိုအခြေအနေမှာ DaemonSet သုံးရင် Node တိုင်းမှာ agent Pod တစ်ခုစီ automatically run ပေးနိုင်ပါတယ်။

ဥပမာ -

```text
Node 1 -> monitoring-agent
Node 2 -> monitoring-agent
Node 3 -> monitoring-agent
```

![Monitoring and Logs Use Case](https://kodekloud.com/kk-media/image/upload/v1752869888/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-DaemonSets/frame_80.jpg)

---

## 4. Use Case 2: kube-proxy

Kubernetes မှာ `kube-proxy` က Node တိုင်းမှာ run ဖို့လိုတဲ့ essential component တစ်ခုပါ။

`kube-proxy` က Service networking အတွက် အရေးကြီးပါတယ်။

ဒါကြောင့် worker node တိုင်းပေါ်မှာ `kube-proxy` instance တစ်ခုစီ run နေရပါမယ်။

DaemonSet သုံးရင် ဒီလို requirement ကို အလွယ်တကူ handle လုပ်နိုင်ပါတယ်။

![kube-proxy DaemonSet Use Case](https://kodekloud.com/kk-media/image/upload/v1752869889/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-DaemonSets/frame_100.jpg)

---

## 5. Use Case 3: Networking Agents

Networking solution တွေဖြစ်တဲ့ `weave-net` လို CNI component တွေကလည်း Node တိုင်းပေါ်မှာ run ဖို့လိုပါတယ်။

Cluster networking အလုပ်လုပ်ဖို့ Node တိုင်းမှာ network agent ရှိရပါမယ်။

ဒီလိုအတွက် DaemonSet ကို အသုံးပြုပါတယ်။

![Networking Agent Use Case](https://kodekloud.com/kk-media/image/upload/v1752869890/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-DaemonSets/frame_120.jpg)

---

## 6. DaemonSet ရဲ့ အရေးကြီးတဲ့ Behavior

DaemonSet က -

1. Node တိုင်းပေါ်မှာ Pod တစ်ခုစီ run စေတယ်။
2. Node အသစ် add လုပ်ရင် Pod အသစ် auto create လုပ်တယ်။
3. Node remove လုပ်ရင် အဲ့ဒီ Node ပေါ်က Pod လည်း remove ဖြစ်တယ်။
4. Monitoring, logging, networking agent တွေအတွက် အသုံးများတယ်။

---

## 7. DaemonSet YAML Structure

DaemonSet YAML က ReplicaSet နဲ့ ဆင်တူပါတယ်။

အဓိက ပါဝင်တာတွေက -

- `apiVersion`
- `kind`
- `metadata`
- `spec.selector`
- `spec.template`

DaemonSet မှာ `apiVersion` က -

```yaml
apiVersion: apps/v1
```

ဖြစ်ပြီး `kind` က -

```yaml
kind: DaemonSet
```

ဖြစ်ပါတယ်။

---

## 8. DaemonSet Example

အောက်က example က monitoring agent ကို Node တိုင်းမှာ run စေတဲ့ DaemonSet YAML ဖြစ်ပါတယ်။

```yaml
# daemon-set-definition.yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: monitoring-daemon
spec:
  selector:
    matchLabels:
      app: monitoring-agent
  template:
    metadata:
      labels:
        app: monitoring-agent
    spec:
      containers:
        - name: monitoring-agent
          image: monitoring-agent
```

ဒီ YAML မှာ DaemonSet name က -

```yaml
name: monitoring-daemon
```

ဖြစ်ပါတယ်။

Pod template label က -

```yaml
labels:
  app: monitoring-agent
```

ဖြစ်ပါတယ်။

Selector က -

```yaml
selector:
  matchLabels:
    app: monitoring-agent
```

ဖြစ်ပါတယ်။

---

## 9. Selector နဲ့ Template Label ကိုက်ရမယ်

DaemonSet မှာ အရေးကြီးဆုံးက `selector.matchLabels` နဲ့ `template.metadata.labels` ကိုက်ရပါမယ်။

ဥပမာ -

```yaml
selector:
  matchLabels:
    app: monitoring-agent
```

နဲ့

```yaml
template:
  metadata:
    labels:
      app: monitoring-agent
```

ဒီနှစ်ခုကိုက်မှ DaemonSet က သူ့ Pod တွေကို မှန်မှန်ကန်ကန် manage လုပ်နိုင်ပါတယ်။

မကိုက်ရင် DaemonSet အလုပ်မလုပ်နိုင်ပါဘူး။

---

## 10. DaemonSet Create လုပ်နည်း

DaemonSet YAML file ကို `daemon-set-definition.yaml` လို့ save ထားတယ်ဆိုပါစို့။

Create လုပ်ရန် -

```bash
kubectl create -f daemon-set-definition.yaml
```

သို့မဟုတ် -

```bash
kubectl apply -f daemon-set-definition.yaml
```

---

## 11. DaemonSet စစ်နည်း

DaemonSet create ဖြစ်မဖြစ် စစ်ရန် -

```bash
kubectl get daemonsets
```

Short form အနေနဲ့ -

```bash
kubectl get ds
```

Output example -

```console
NAME                DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   AGE
monitoring-daemon   1         1         1       1            1           41s
```

---

## 12. DaemonSet Output Column Meaning

`kubectl get daemonsets` output မှာ ပါတဲ့ column တွေကို နားလည်ထားသင့်ပါတယ်။

| Column | အဓိပ္ပါယ် |
|---|---|
| `DESIRED` | DaemonSet က run စေချင်တဲ့ Pod အရေအတွက် |
| `CURRENT` | လက်ရှိ create ဖြစ်နေတဲ့ Pod အရေအတွက် |
| `READY` | Ready ဖြစ်နေတဲ့ Pod အရေအတွက် |
| `UP-TO-DATE` | Latest template နဲ့ ကိုက်တဲ့ Pod အရေအတွက် |
| `AVAILABLE` | Available ဖြစ်နေတဲ့ Pod အရေအတွက် |
| `AGE` | DaemonSet ရဲ့ အသက် |

DaemonSet က Node တိုင်းမှာ Pod တစ်ခုစီ run စေတဲ့အတွက် `DESIRED` count က usually eligible nodes အရေအတွက်နဲ့ ကိုက်ပါတယ်။

---

## 13. DaemonSet Detail ကြည့်နည်း

DaemonSet အသေးစိတ်ကြည့်ရန် -

```bash
kubectl describe daemonset monitoring-daemon
```

Short form -

```bash
kubectl describe ds monitoring-daemon
```

ဒီ command နဲ့ ကြည့်နိုင်တာတွေက -

- Selector
- Pod template
- Events
- Desired/current/ready pod count
- Node scheduling status

---

## 14. DaemonSet Pod တွေကို ကြည့်နည်း

DaemonSet က create လုပ်ထားတဲ့ Pod တွေကို ကြည့်ရန် -

```bash
kubectl get pods -o wide
```

Label နဲ့ filter လုပ်ချင်ရင် -

```bash
kubectl get pods -l app=monitoring-agent -o wide
```

ဒီ command မှာ `NODE` column ကို ကြည့်ပြီး Pod တွေ Node တိုင်းပေါ် run နေလား စစ်နိုင်ပါတယ်။

---

## 15. DaemonSet Scheduling အလုပ်လုပ်ပုံ

Kubernetes version 1.12 မတိုင်ခင်မှာ DaemonSet Pod ကို specific node ပေါ်တင်ဖို့ `nodeName` ကို အသုံးပြုခဲ့တာတွေရှိပါတယ်။

Kubernetes 1.12 နောက်ပိုင်းမှာတော့ DaemonSet က default scheduler နဲ့ node affinity rules တွေကို အသုံးပြုပြီး Pod ကို Node တိုင်းပေါ် schedule လုပ်ပါတယ်။

အဓိက advantage က manual intervention မလိုဘဲ Node တိုင်းမှာ Pod automatically run နိုင်တာပါ။

![DaemonSet Scheduling](https://kodekloud.com/kk-media/image/upload/v1752869890/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-DaemonSets/frame_240.jpg)

---

## 16. DaemonSet vs ReplicaSet vs Deployment

| Feature | DaemonSet | ReplicaSet | Deployment |
|---|---|---|---|
| Main purpose | Node တိုင်းမှာ Pod တစ်ခုစီ run စေဖို့ | Desired replica count ထိန်းဖို့ | ReplicaSet/Pod rollout manage ဖို့ |
| Pod placement | Node တိုင်းပေါ် | Scheduler ဆုံးဖြတ်တဲ့ Node တွေပေါ် | Scheduler ဆုံးဖြတ်တဲ့ Node တွေပေါ် |
| Use case | Agents, kube-proxy, networking | Simple replica control | App deployment, rolling update |
| Node အသစ် add လုပ်ရင် | Pod auto create ဖြစ်တယ် | Replica count ပေါ်မူတည်တယ် | Replica count ပေါ်မူတည်တယ် |

---

## 17. DaemonSet vs Deployment

Deployment က application replicas တွေကို run ဖို့ သုံးပါတယ်။

ဥပမာ -

```text
nginx app ကို replica 3 ခု run မယ်
```

DaemonSet ကတော့ Node တိုင်းမှာ Pod တစ်ခုစီ run ဖို့ သုံးပါတယ်။

ဥပမာ -

```text
Node တိုင်းမှာ log collector တစ်ခုစီ run မယ်
```

အလွယ်မှတ်ရန် -

```text
Deployment = desired number of pods
DaemonSet  = one pod per node
```

---

## 18. DaemonSet Delete လုပ်နည်း

DaemonSet ကို delete လုပ်ရန် -

```bash
kubectl delete daemonset monitoring-daemon
```

Short form -

```bash
kubectl delete ds monitoring-daemon
```

DaemonSet ကို delete လုပ်ရင် သူ create လုပ်ထားတဲ့ Pod တွေလည်း delete ဖြစ်သွားပါမယ်။

---

## 19. အသုံးများတဲ့ Commands

### DaemonSet create/apply လုပ်ရန်

```bash
kubectl apply -f daemon-set-definition.yaml
```

### DaemonSets list ကြည့်ရန်

```bash
kubectl get daemonsets
```

Short form -

```bash
kubectl get ds
```

### DaemonSet detail ကြည့်ရန်

```bash
kubectl describe daemonset monitoring-daemon
```

Short form -

```bash
kubectl describe ds monitoring-daemon
```

### DaemonSet Pod တွေကြည့်ရန်

```bash
kubectl get pods -o wide
```

### Label နဲ့ DaemonSet Pod တွေကြည့်ရန်

```bash
kubectl get pods -l app=monitoring-agent -o wide
```

### DaemonSet delete လုပ်ရန်

```bash
kubectl delete ds monitoring-daemon
```

---

## 20. CKA Exam အတွက် မှတ်ရန်

CKA exam မှာ DaemonSet က workload controller topic ထဲက အရေးကြီးတဲ့အပိုင်းပါ။

အဓိက မှတ်ထားရမယ့်အချက်တွေက -

- DaemonSet က Node တိုင်းပေါ်မှာ Pod တစ်ခုစီ run စေတယ်။
- Node အသစ် add လုပ်ရင် DaemonSet Pod auto create ဖြစ်တယ်။
- Node remove လုပ်ရင် corresponding Pod remove ဖြစ်တယ်။
- Monitoring agent, log collector, kube-proxy, networking agent တွေအတွက် သုံးတယ်။
- `apiVersion: apps/v1`
- `kind: DaemonSet`
- `selector.matchLabels` နဲ့ `template.metadata.labels` ကိုက်ရမယ်။
- `kubectl get ds`
- `kubectl describe ds <name>`
- `kubectl get pods -o wide`

---

## 21. Simple Summary

DaemonSet ဆိုတာ Kubernetes မှာ **Node တိုင်းပေါ်မှာ Pod တစ်ခုစီ run စေဖို့** သုံးတဲ့ controller ပါ။

```text
1 Node = 1 DaemonSet Pod
```

Node အသစ်ထပ်ထည့်ရင် DaemonSet က Pod အသစ်တစ်ခုကို automatically create လုပ်ပေးပါတယ်။

DaemonSet ကို အများအားဖြင့် -

- Monitoring agent
- Log collector
- kube-proxy
- Networking agent

တွေအတွက် သုံးပါတယ်။

အရေးကြီးဆုံး YAML structure -

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: monitoring-daemon
spec:
  selector:
    matchLabels:
      app: monitoring-agent
  template:
    metadata:
      labels:
        app: monitoring-agent
    spec:
      containers:
        - name: monitoring-agent
          image: monitoring-agent
```

အရေးကြီးဆုံး command တွေ -

```bash
kubectl get ds
kubectl describe ds monitoring-daemon
kubectl get pods -o wide
```

DaemonSet ကို သုံးရတဲ့အဓိကအကြောင်းရင်းက Node တိုင်းမှာ အရေးကြီးတဲ့ background service တစ်ခုစီ အမြဲ run နေစေချင်လို့ ဖြစ်ပါတယ်။
