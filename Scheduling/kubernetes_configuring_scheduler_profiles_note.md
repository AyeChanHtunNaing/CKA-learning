# Kubernetes Configuring Scheduler Profiles Note

## 1. Scheduler Profiles ဆိုတာဘာလဲ

**Scheduler Profiles** ဆိုတာ Kubernetes Scheduler တစ်ခုတည်းထဲမှာ scheduler behavior မတူတဲ့ profile များစွာကို configure လုပ်နိုင်တဲ့ feature ပါ။

အရင် Multiple Schedulers topic မှာ scheduler binary/process တစ်ခုစီကို သီးသန့် run လုပ်တာကို လေ့လာခဲ့ပါတယ်။

ဥပမာ -

```text
default-scheduler
my-scheduler
my-scheduler-2
```

ဒါပေမယ့် Kubernetes 1.18 နောက်ပိုင်းမှာ **single scheduler binary** ထဲမှာ **multiple profiles** ထည့်ပြီး scheduler behavior မတူအောင် configure လုပ်နိုင်ပါတယ်။

အလွယ်မှတ်ရန် -

```text
Multiple Schedulers = scheduler process အများကြီး
Scheduler Profiles  = scheduler process တစ်ခုထဲမှာ profile အများကြီး
```

---

## 2. Kubernetes Scheduling အလုပ်လုပ်ပုံ ပြန်လည်မှတ်ချက်

Pod တစ်ခု create လုပ်လိုက်ရင် scheduler က ချက်ချင်း Node ပေါ်တင်တာ မဟုတ်ပါဘူး။

အရင်ဆုံး Pod က **scheduling queue** ထဲဝင်ပါတယ်။

Scheduler က pending Pods တွေကို queue ထဲကနေယူပြီး suitable Node ရှာပေးပါတယ်။

ဥပမာ Pod တစ်ခုက CPU 10 လိုတယ်ဆိုပါစို့။

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
spec:
  priorityClassName: high-priority
  containers:
    - name: simple-webapp-color
      image: simple-webapp-color
      resources:
        requests:
          memory: "1Gi"
          cpu: 10
```

ဒီ Pod ကို schedule လုပ်ဖို့ Node မှာ available CPU 10 ရှိရပါမယ်။

---

## 3. Priority and Scheduling Queue

Scheduler queue ထဲမှာ Pod တွေကို priority အရ စီနိုင်ပါတယ်။

ဥပမာ -

```yaml
priorityClassName: high-priority
```

ဒီလို priority class သတ်မှတ်ထားတဲ့ Pod က lower priority Pod တွေထက် scheduling queue ရဲ့ အရှေ့ပိုင်းကို ရောက်နိုင်ပါတယ်။

PriorityClass တစ်ခု create လုပ်ပြီး value မြင့်မြင့်ပေးထားရင် အဲ့ဒီ PriorityClass သုံးထားတဲ့ Pod တွေက schedule အရင်လုပ်ခံရနိုင်ပါတယ်။

အလွယ်မှတ်ရန် -

```text
Higher priority Pod = Queue ထဲမှာ အရင် schedule ခံရနိုင်
```

---

## 4. Scheduling Phases

Scheduler က Pod တစ်ခုကို Node ပေါ်တင်တဲ့အခါ အဆင့်များစွာ ဖြတ်သွားပါတယ်။

အဓိက phases ၃ ခုက -

1. Filter Phase
2. Scoring Phase
3. Binding Phase

---

## 5. Filter Phase

**Filter Phase** မှာ Pod ကို run မလုပ်နိုင်တဲ့ Node တွေကို ဖယ်ထုတ်ပါတယ်။

ဥပမာ Pod က CPU 10 လိုတယ်ဆိုရင် -

```text
Node A = available CPU 4  -> filtered out
Node B = available CPU 8  -> filtered out
Node C = available CPU 12 -> passed
```

ဒီ phase မှာ resource မလုံလောက်တဲ့ Node တွေကို scheduler က မရွေးတော့ပါဘူး။

---

## 6. Scoring Phase

**Scoring Phase** မှာ Filter Phase ကို ဖြတ်ကျော်လာတဲ့ Node တွေကို score ပေးပါတယ်။

ဥပမာ Pod က CPU 10 လိုတယ်ဆိုပါစို့။

```text
Node A = CPU 16 available -> after scheduling left 6 CPU
Node B = CPU 12 available -> after scheduling left 2 CPU
```

Resource ပိုကျန်တဲ့ Node က score ပိုကောင်းနိုင်ပါတယ်။

Scoring phase က Node တွေကို reject မလုပ်ဘဲ **ဘယ် Node က ပိုကောင်းလဲ** ဆိုတာ သတ်မှတ်ပေးတာပါ။

---

## 7. Binding Phase

**Binding Phase** မှာ score အကောင်းဆုံး Node ကိုရွေးပြီး Pod ကို အဲ့ဒီ Node နဲ့ bind လုပ်ပါတယ်။

အဓိပ္ပါယ်က -

```text
Pod -> selected Node
```

အဖြစ် assign လုပ်တာပါ။

ဒီအဆင့်ပြီးရင် kubelet က container runtime ကိုသုံးပြီး Pod ကို run စေပါတယ်။

---

## 8. Scheduler Plugins ဆိုတာဘာလဲ

Kubernetes Scheduler က plugin-based architecture ကို သုံးပါတယ်။

Scheduling process ရဲ့ phase တစ်ခုချင်းစီမှာ plugin တွေက အလုပ်လုပ်ပါတယ်။

Plugin တွေက scheduler behavior ကို customize လုပ်နိုင်တဲ့အတွက် Kubernetes scheduler က extensible ဖြစ်ပါတယ်။

အလွယ်မှတ်ရန် -

```text
Scheduler Plugin = Scheduling process ထဲမှာ rule/action တစ်ခုလုပ်ပေးတဲ့ component
```

---

## 9. Key Scheduler Plugins

### 1. Priority Sort Plugin

**Priority Sort Plugin** က scheduling queue ထဲက Pod တွေကို priority အရ စီပေးပါတယ်။

```text
High priority Pod -> Queue အရှေ့
Low priority Pod  -> Queue နောက်
```

---

### 2. Node Resources Fit Plugin

**Node Resources Fit Plugin** က Pod ရဲ့ resource requirements ကို Node က ဖြည့်ဆည်းနိုင်မနိုင် စစ်ပါတယ်။

ဥပမာ -

```text
Pod needs CPU 10
Node has CPU 4
```

ဆိုရင် ဒီ Node ကို filter out လုပ်ပါတယ်။

---

### 3. Node Name Plugin

**Node Name Plugin** က Pod spec ထဲမှာ `nodeName` သတ်မှတ်ထားလား စစ်ပါတယ်။

ဥပမာ -

```yaml
spec:
  nodeName: node01
```

ဒီလိုရှိရင် scheduler က `node01` နဲ့ကိုက်တဲ့ Node ကိုပဲ စဉ်းစားပါတယ်။

---

### 4. Node Unschedulable Plugin

**Node Unschedulable Plugin** က unschedulable ဖြစ်နေတဲ့ Node တွေကို filter out လုပ်ပါတယ်။

ဥပမာ node ကို cordon/drain လုပ်ထားရင် Node က unschedulable ဖြစ်နိုင်ပါတယ်။

Example node description -

```bash
kubectl describe node controlplane
```

Output example -

```text
Taints:             node.kubernetes.io/unschedulable:NoSchedule
Unschedulable:      true
```

ဒီလို Node တွေကို scheduler က Pod placement အတွက် မရွေးပါဘူး။

---

### 5. Scoring Plugins

Scoring phase မှာ Node တွေကို score ပေးတဲ့ plugins တွေရှိပါတယ်။

ဥပမာ -

- Node Resources Fit
- Image Locality

ဒီ plugins တွေက Node တွေကို outright reject မလုပ်ဘဲ ဘယ် Node ပိုသင့်တော်လဲဆိုတာ score ပေးပါတယ်။

---

### 6. Default Binder Plugin

**Default Binder Plugin** က scheduling process ရဲ့ နောက်ဆုံးအဆင့်မှာ Pod ကို selected Node နဲ့ bind လုပ်ပါတယ်။

```text
Pod assigned to Node
```

---

## 10. Scheduler Extension Points

Kubernetes Scheduler မှာ extension points များစွာရှိပါတယ်။

ဥပမာ -

- `queueSort`
- `preFilter`
- `filter`
- `postFilter`
- `preScore`
- `score`
- `reserve`
- `permit`
- `preBind`
- `bind`
- `postBind`

ဒီ extension points တွေမှာ plugin တွေကို enable/disable လုပ်နိုင်ပါတယ်။

![Kubernetes Scheduler Extension Points](https://kodekloud.com/kk-media/image/upload/v1752869885/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-Configuring-Scheduler-Profiles/frame_380.jpg)

---

## 11. Scheduler Profiles ကို ဘာကြောင့်သုံးလဲ

Multiple scheduler binaries/processes run လုပ်တာက operational overhead ရှိနိုင်ပါတယ်။

ဥပမာ -

```text
default scheduler process
my-scheduler process
my-scheduler-2 process
```

ဒီလို process အများကြီး run တာက management ခက်နိုင်သလို scheduling race condition တွေလည်း ဖြစ်နိုင်ပါတယ်။

Scheduler Profiles သုံးရင် scheduler binary တစ်ခုတည်းထဲမှာ profiles အများကြီး configure လုပ်နိုင်ပါတယ်။

အကျိုးကျေးဇူး -

- Scheduler process တစ်ခုတည်းနဲ့ multiple scheduling behaviors ရနိုင်တယ်။
- Operational overhead လျော့တယ်။
- Race condition ဖြစ်နိုင်ခြေ လျော့တယ်။
- Profile တစ်ခုချင်းစီမှာ plugin config မတူအောင် configure လုပ်နိုင်တယ်။

---

## 12. Scheduler Profile Configuration

Scheduler profile တစ်ခုချင်းစီကို scheduler configuration file ထဲမှာ `profiles` အောက်မှာရေးပါတယ်။

Example -

```yaml
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:
  - schedulerName: my-scheduler-2
  - schedulerName: my-scheduler-3
```

ဒီ example မှာ scheduler binary တစ်ခုတည်းထဲမှာ profile ၂ ခုရှိပါတယ်။

```text
my-scheduler-2
my-scheduler-3
```

Pod တွေက `schedulerName` နဲ့ profile ကို ရွေးသုံးနိုင်ပါတယ်။

---

## 13. Single Profile Example

```yaml
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:
  - schedulerName: my-scheduler
```

ဒီ config က `my-scheduler` ဆိုတဲ့ scheduler profile တစ်ခုတည်း configure လုပ်တာပါ။

---

## 14. Default Scheduler Profile Example

```yaml
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:
  - schedulerName: default-scheduler
```

Default scheduler ကိုလည်း scheduler profile အနေနဲ့ configure လုပ်နိုင်ပါတယ်။

Default scheduler name က -

```text
default-scheduler
```

ဖြစ်ပါတယ်။

---

## 15. Plugin Enable/Disable လုပ်နည်း

Scheduler profiles မှာ plugin တွေကို enable/disable လုပ်နိုင်ပါတယ်။

Example -

```yaml
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:
  - schedulerName: my-scheduler-2
    plugins:
      score:
        disabled:
          - name: TaintToleration
        enabled:
          - name: MyCustomPluginA
          - name: MyCustomPluginB
  - schedulerName: my-scheduler-3
    plugins:
      preScore:
        disabled:
          - name: '*'
      score:
        disabled:
          - name: '*'
  - schedulerName: my-scheduler-4
```

ဒီ YAML မှာ profile ၃ ခုရှိပါတယ်။

- `my-scheduler-2`
- `my-scheduler-3`
- `my-scheduler-4`

---

## 16. Example: `my-scheduler-2`

```yaml
- schedulerName: my-scheduler-2
  plugins:
    score:
      disabled:
        - name: TaintToleration
      enabled:
        - name: MyCustomPluginA
        - name: MyCustomPluginB
```

ဒီ profile မှာ -

```text
score extension point
```

အတွက် configuration လုပ်ထားပါတယ်။

လုပ်ထားတာက -

- `TaintToleration` plugin ကို disable လုပ်ထားတယ်။
- `MyCustomPluginA` ကို enable လုပ်ထားတယ်။
- `MyCustomPluginB` ကို enable လုပ်ထားတယ်။

---

## 17. Example: `my-scheduler-3`

```yaml
- schedulerName: my-scheduler-3
  plugins:
    preScore:
      disabled:
        - name: '*'
    score:
      disabled:
        - name: '*'
```

ဒီ profile မှာ -

```text
preScore plugins အားလုံး disable
score plugins အားလုံး disable
```

လုပ်ထားပါတယ်။

`*` wildcard က plugins အားလုံးကို ဆိုလိုပါတယ်။

---

## 18. Example: `my-scheduler-4`

```yaml
- schedulerName: my-scheduler-4
```

ဒီ profile မှာ plugin customization မရှိပါဘူး။

ဒါကြောင့် default scheduler plugin behavior ကို သုံးနိုင်ပါတယ်။

---

## 19. Workload ကို Scheduler Profile သုံးခိုင်းနည်း

Pod တစ်ခုကို specific scheduler profile သုံးစေချင်ရင် Pod spec ထဲမှာ `schedulerName` ထည့်ရပါတယ်။

ဥပမာ -

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
    - name: nginx
      image: nginx
  schedulerName: my-scheduler-2
```

ဒီ Pod ကို `my-scheduler-2` profile က schedule လုပ်ပါမယ်။

`schedulerName` မထည့်ထားရင် default scheduler သုံးပါမယ်။

---

## 20. Scheduler Profiles vs Multiple Schedulers

| Feature | Multiple Schedulers | Scheduler Profiles |
|---|---|---|
| Scheduler process | အများကြီး run တတ်တယ် | Process တစ်ခုတည်း |
| Config | Scheduler တစ်ခုချင်းစီ config သီးသန့် | Config file တစ်ခုထဲ profiles များစွာ |
| Operational overhead | ပိုများ | ပိုနည်း |
| Race condition risk | ပိုရှိနိုင် | ပိုနည်း |
| Plugin customization | Scheduler တစ်ခုချင်းစီ | Profile တစ်ခုချင်းစီ |
| Kubernetes version | Older approach | Kubernetes 1.18+ |

---

## 21. Scheduler Profile မကိုက်ရင် ဘာဖြစ်မလဲ

Pod မှာ `schedulerName` ထည့်ထားပြီး cluster scheduler config ထဲမှာ အဲ့ဒီ profile မရှိရင် Pod က schedule မလုပ်နိုင်ပါဘူး။

Pod status က `Pending` ဖြစ်နေနိုင်ပါတယ်။

စစ်ရန် -

```bash
kubectl get pods
```

Detail ကြည့်ရန် -

```bash
kubectl describe pod <pod-name>
```

Events ကြည့်ရန် -

```bash
kubectl get events -o wide
```

---

## 22. အသုံးများတဲ့ Commands

### Scheduler config ကြည့်ရန်

Scheduler config file path က cluster setup ပေါ်မူတည်ပါတယ်။

kubeadm setup တွေမှာ kube-scheduler static pod manifest ကို ဒီ path မှာ တွေ့နိုင်ပါတယ်။

```bash
ls /etc/kubernetes/manifests
```

Scheduler manifest ကြည့်ရန် -

```bash
cat /etc/kubernetes/manifests/kube-scheduler.yaml
```

### Pod events ကြည့်ရန်

```bash
kubectl get events -o wide
```

### Pod details ကြည့်ရန်

```bash
kubectl describe pod <pod-name>
```

### kube-system scheduler pods ကြည့်ရန်

```bash
kubectl get pods -n kube-system
```

### Scheduler logs ကြည့်ရန်

```bash
kubectl logs <scheduler-pod-name> -n kube-system
```

---

## 23. CKA Exam အတွက် မှတ်ရန်

CKA exam အတွက် Scheduler Profiles topic မှာ အောက်ပါအချက်တွေကို နားလည်ထားသင့်ပါတယ်။

- Scheduler က queueing, filtering, scoring, binding phases တွေသုံးတယ်။
- Priority Sort Plugin က queue ထဲမှာ Pod priority အလိုက် စီပေးတယ်။
- Node Resources Fit Plugin က resource မလုံလောက်တဲ့ Node တွေကို filter လုပ်တယ်။
- Node Name Plugin က `nodeName` ကို စစ်တယ်။
- Node Unschedulable Plugin က cordon/drain လုပ်ထားတဲ့ Node တွေကို filter လုပ်တယ်။
- Default Binder Plugin က Pod ကို selected Node နဲ့ bind လုပ်တယ်။
- Scheduler Profiles က scheduler binary တစ်ခုထဲမှာ multiple scheduling behaviors configure လုပ်နိုင်တယ်။
- Profile တစ်ခုချင်းစီမှာ unique `schedulerName` ရှိရမယ်။
- Plugin တွေကို extension point အလိုက် enable/disable လုပ်နိုင်တယ်။
- `*` wildcard သုံးပြီး plugins အားလုံး disable လုပ်နိုင်တယ်။
- Pod spec ထဲမှာ `schedulerName` ထည့်ပြီး specific profile သုံးခိုင်းနိုင်တယ်။

---

## 24. Simple Summary

Scheduler Profiles ဆိုတာ Kubernetes scheduler binary တစ်ခုတည်းထဲမှာ scheduler behavior မတူတဲ့ profiles များစွာ configure လုပ်နိုင်တဲ့ feature ပါ။

Scheduler process အများကြီး run စရာမလိုဘဲ -

```yaml
profiles:
  - schedulerName: my-scheduler-2
  - schedulerName: my-scheduler-3
```

လို profile များစွာ configure လုပ်နိုင်ပါတယ်။

Profile တစ်ခုချင်းစီမှာ plugin behavior မတူအောင် configure လုပ်နိုင်ပါတယ်။

```yaml
plugins:
  score:
    disabled:
      - name: TaintToleration
    enabled:
      - name: MyCustomPluginA
```

Pod ကို specific profile သုံးစေချင်ရင် -

```yaml
schedulerName: my-scheduler-2
```

ထည့်ရပါတယ်။

အရေးကြီးဆုံး မှတ်ရန် -

```text
Scheduler Profiles = one scheduler binary, multiple scheduler behaviors
```

```text
schedulerName = Pod က ဘယ် scheduler/profile သုံးမလဲ သတ်မှတ်တာ
```

```text
plugins = scheduling phases မှာ behavior ကို customize လုပ်တာ
```
