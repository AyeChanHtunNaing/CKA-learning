# Kubernetes Multiple Schedulers Note

## 1. Multiple Schedulers ဆိုတာဘာလဲ

**Multiple Schedulers** ဆိုတာ Kubernetes cluster ထဲမှာ default scheduler အပြင် custom scheduler တစ်ခု သို့မဟုတ် တစ်ခုထက်ပိုပြီး run စေတဲ့ concept ပါ။

ပုံမှန် Kubernetes မှာ Pod တွေကို schedule လုပ်တာက default scheduler ဖြစ်ပါတယ်။

Default scheduler name က -

```text
default-scheduler
```

ဖြစ်ပါတယ်။

ဒါပေမယ့် တချို့ use case တွေမှာ default scheduler ရဲ့ scheduling logic မလုံလောက်နိုင်ပါဘူး။

ဥပမာ -

- Custom application placement rule လိုတယ်
- Pod ကို node ပေါ်မတင်ခင် extra verification လုပ်ချင်တယ်
- Specific workload တွေကို custom algorithm နဲ့ schedule လုပ်ချင်တယ်
- Default scheduler နဲ့ မတူတဲ့ scheduling behavior လိုတယ်

ဒီလိုအချိန်မှာ **custom scheduler** ကို deploy လုပ်ပြီး Pod တွေကို အဲ့ဒီ scheduler နဲ့ schedule လုပ်နိုင်ပါတယ်။

---

## 2. Default Scheduler အလုပ်လုပ်ပုံ

Kubernetes default scheduler က Pod တွေကို Node တွေပေါ်မှာ distribute လုပ်ပါတယ်။

Scheduler က Pod placement ဆုံးဖြတ်တဲ့အခါ အောက်ပါအချက်တွေကို စဉ်းစားပါတယ်။

- Node resources
- Taints and tolerations
- Node affinity
- Pod affinity / anti-affinity
- Resource requests
- Other scheduling rules

အလွယ်မှတ်ရန် -

```text
default-scheduler = Kubernetes ရဲ့ built-in scheduler
```

---

## 3. Custom Scheduler ကို ဘာကြောင့်သုံးလဲ

Custom scheduler ကို default scheduler မလုပ်နိုင်တဲ့ custom scheduling logic တွေအတွက် သုံးပါတယ်။

ဥပမာ -

```text
Application တစ်ခုရဲ့ component တွေကို node ပေါ်မတင်ခင် custom verification လုပ်ချင်တယ်
```

ဒီလိုဆိုရင် ကိုယ်ပိုင် scheduler logic ရေးပြီး Kubernetes cluster ထဲမှာ default scheduler နဲ့အတူ run စေနိုင်ပါတယ်။

အရေးကြီးတာက scheduler တစ်ခုချင်းစီမှာ unique name ရှိရပါမယ်။

---

## 4. Scheduler Name

Kubernetes မှာ scheduler ကို name နဲ့ ခွဲခြားပါတယ်။

Default scheduler name က -

```text
default-scheduler
```

Custom scheduler name ကို ကိုယ်ပိုင်နာမည် ပေးနိုင်ပါတယ်။

ဥပမာ -

```text
my-scheduler
my-scheduler-2
my-custom-scheduler
```

အရေးကြီးဆုံး မှတ်ရန် -

```text
Every scheduler must have a unique schedulerName.
```

---

## 5. Scheduler Configuration YAML

Scheduler configuration file ထဲမှာ `profiles` list ကိုသုံးပြီး scheduler name သတ်မှတ်ပါတယ်။

Custom scheduler config example -

```yaml
# my-scheduler-config.yaml
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:
  - schedulerName: my-scheduler
```

Default scheduler config example -

```yaml
# scheduler-config.yaml
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:
  - schedulerName: default-scheduler
```

ဒီမှာ အရေးကြီးတာက -

```yaml
schedulerName: my-scheduler
```

ဖြစ်ပါတယ်။

Pod တွေက ဒီ scheduler ကိုသုံးချင်ရင် Pod spec ထဲမှာ ဒီ name ကို reference လုပ်ရပါမယ်။

---

## 6. Additional Scheduler Deploy လုပ်နည်း Overview

Custom scheduler ကို deploy လုပ်နိုင်တဲ့နည်းလမ်းတွေက -

1. kube-scheduler binary ကို service အဖြစ် run ခြင်း
2. Custom scheduler ကို Pod အဖြစ် run ခြင်း
3. Custom scheduler ကို Deployment အဖြစ် run ခြင်း

Modern Kubernetes/kubeadm environment တွေမှာ scheduler ကို Pod သို့မဟုတ် Deployment အဖြစ် run တာကို ပိုတွေ့ရနိုင်ပါတယ်။

---

## 7. kube-scheduler Binary Download လုပ်နည်း

Custom scheduler ကို kube-scheduler binary နဲ့ run ချင်ရင် binary ကို download လုပ်နိုင်ပါတယ်။

Example -

```bash
wget https://storage.googleapis.com/kubernetes-release/release/v1.12.0/bin/linux/amd64/kube-scheduler
```

ပြီးရင် binary ကို executable ဖြစ်အောင် permission ပေးနိုင်ပါတယ်။

```bash
chmod +x kube-scheduler
```

---

## 8. Scheduler Service File Example

Default scheduler service file example -

```bash
# kube-scheduler.service
ExecStart=/usr/local/bin/kube-scheduler --config=/etc/kubernetes/config/kube-scheduler.yaml
```

Custom scheduler service file example -

```bash
# my-scheduler-2.service
ExecStart=/usr/local/bin/kube-scheduler --config=/etc/kubernetes/config/my-scheduler-2-config.yaml
```

ဒီမှာ scheduler တစ်ခုချင်းစီက different config file ကို အသုံးပြုပါတယ်။

---

## 9. Custom Scheduler Config Example

```yaml
# my-scheduler-2-config.yaml
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:
  - schedulerName: my-scheduler-2
```

နောက်ထပ် custom scheduler config example -

```yaml
# my-scheduler-config.yaml
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:
  - schedulerName: my-scheduler
```

အရေးကြီးတာက scheduler name တွေ မတူရပါမယ်။

```text
my-scheduler
my-scheduler-2
default-scheduler
```

---

## 10. Custom Scheduler ကို Pod အဖြစ် Deploy လုပ်နည်း

Custom scheduler ကို Kubernetes cluster ထဲမှာ Pod အဖြစ်လည်း run နိုင်ပါတယ်။

Example Pod definition -

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-custom-scheduler
  namespace: kube-system
spec:
  containers:
    - name: kube-scheduler
      image: k8s.gcr.io/kube-scheduler-amd64:v1.11.3
      command:
        - kube-scheduler
        - --address=127.0.0.1
        - --kubeconfig=/etc/kubernetes/scheduler.conf
        - --config=/etc/kubernetes/my-scheduler-config.yaml
```

ဒီ Pod က `kube-system` namespace ထဲမှာ run ပါမယ်။

အရေးကြီးတဲ့ flag က -

```bash
--config=/etc/kubernetes/my-scheduler-config.yaml
```

ဖြစ်ပါတယ်။

---

## 11. Custom Scheduler Config File

Pod အဖြစ် run မယ့် custom scheduler အတွက် configuration file example -

```yaml
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:
  - schedulerName: my-scheduler
```

ဒီ configuration ထဲက `schedulerName` ကို Pod workload တွေမှာ reference လုပ်ရပါမယ်။

---

## 12. Leader Election ဆိုတာဘာလဲ

**Leader Election** ဆိုတာ scheduler instance အများကြီး run နေတဲ့ high availability environment မှာ active scheduler တစ်ခုပဲ Pod scheduling လုပ်အောင် ထိန်းချုပ်တဲ့ mechanism ပါ။

ဥပမာ scheduler replicas ၂ ခုရှိရင် -

```text
scheduler-1 = active leader
scheduler-2 = standby
```

Leader election မရှိရင် scheduler instances များစွာက တပြိုင်နက် scheduling လုပ်ပြီး conflict ဖြစ်နိုင်ပါတယ်။

အလွယ်မှတ်ရန် -

```text
Leader election = scheduler instances အများကြီးထဲက တစ်ခုပဲ active ဖြစ်အောင်လုပ်တာ
```

---

## 13. Custom Scheduler ကို Deployment အဖြစ် Deploy လုပ်နည်း

Modern Kubernetes setup တွေမှာ custom scheduler ကို Deployment အဖြစ် deploy လုပ်နိုင်ပါတယ်။

Steps overview -

1. Custom scheduler image build/push လုပ်မယ်။
2. ServiceAccount create လုပ်မယ်။
3. RBAC permissions ပေးမယ်။
4. ConfigMap ထဲမှာ scheduler config ထည့်မယ်။
5. Deployment create လုပ်မယ်။

---

## 14. Custom Scheduler Dockerfile Example

```dockerfile
FROM busybox
ADD ./.output/local/bin/linux/amd64/kube-scheduler /usr/local/bin/kube-scheduler
```

Image build လုပ်ရန် -

```bash
docker build -t gcr.io/my-gcp-project/my-kube-scheduler:1.0 .
```

Image push လုပ်ရန် -

```bash
gcloud docker -- push gcr.io/my-gcp-project/my-kube-scheduler:1.0
```

---

## 15. ServiceAccount and RBAC Example

Custom scheduler က Kubernetes API ကို access လုပ်ရမှာဖြစ်လို့ ServiceAccount နဲ့ RBAC permission လိုပါတယ်။

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-scheduler
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: my-scheduler-as-kube-scheduler
subjects:
  - kind: ServiceAccount
    name: my-scheduler
    namespace: kube-system
roleRef:
  kind: ClusterRole
  name: system:kube-scheduler
  apiGroup: rbac.authorization.k8s.io
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: my-scheduler-as-volume-scheduler
subjects:
  - kind: ServiceAccount
    name: my-scheduler
    namespace: kube-system
roleRef:
  kind: ClusterRole
  name: system:volume-scheduler
  apiGroup: rbac.authorization.k8s.io
```

ဒီ config က custom scheduler ကို scheduler role နဲ့ volume scheduler role permission ပေးတာပါ။

---

## 16. Scheduler ConfigMap Example

Scheduler configuration ကို ConfigMap ထဲမှာ ထည့်နိုင်ပါတယ်။

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-scheduler-config
  namespace: kube-system
data:
  my-scheduler-config.yaml: |
    apiVersion: kubescheduler.config.k8s.io/v1beta2
    kind: KubeSchedulerConfiguration
    profiles:
      - schedulerName: my-scheduler
        leaderElection:
          leaderElect: false
```

ဒီမှာ scheduler name က -

```yaml
schedulerName: my-scheduler
```

ဖြစ်ပါတယ်။

---

## 17. Custom Scheduler Deployment Example

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-scheduler
  namespace: kube-system
  labels:
    component: scheduler
    tier: control-plane
spec:
  replicas: 1
  selector:
    matchLabels:
      component: scheduler
      tier: control-plane
  template:
    metadata:
      labels:
        component: scheduler
        tier: control-plane
        version: second
    spec:
      serviceAccountName: my-scheduler
      containers:
        - name: kube-second-scheduler
          image: gcr.io/my-gcp-project/my-kube-scheduler:1.0
          command:
            - /usr/local/bin/kube-scheduler
            - --config=/etc/kubernetes/my-scheduler/my-scheduler-config.yaml
          livenessProbe:
            httpGet:
              path: /healthz
              port: 10259
              scheme: HTTPS
            initialDelaySeconds: 15
          readinessProbe:
            httpGet:
              path: /healthz
              port: 10259
              scheme: HTTPS
          volumeMounts:
            - name: config-volume
              mountPath: /etc/kubernetes/my-scheduler
      volumes:
        - name: config-volume
          configMap:
            name: my-scheduler-config
```

ဒီ Deployment မှာ -

- `serviceAccountName: my-scheduler`
- ConfigMap ကို volume အဖြစ် mount လုပ်ထားတယ်။
- Scheduler config ကို `--config` flag နဲ့ reference လုပ်ထားတယ်။
- Health check အတွက် liveness/readiness probe သုံးထားတယ်။

---

## 18. ClusterRole Permission Example

Scheduler က leader election အတွက် leases resource ကို access လုပ်နိုင်ရပါမယ်။

Example -

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: system:kube-scheduler
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
rules:
  - apiGroups:
      - coordination.k8s.io
    resources:
      - leases
    verbs:
      - create
  - apiGroups:
      - coordination.k8s.io
    resourceNames:
      - kube-scheduler
      - my-scheduler
    resources:
      - leases
    verbs:
      - get
      - list
      - watch
```

---

## 19. Workload ကို Custom Scheduler သုံးခိုင်းနည်း

Pod တစ်ခုကို custom scheduler နဲ့ schedule လုပ်ချင်ရင် Pod spec ထဲမှာ `schedulerName` ထည့်ရပါတယ်။

Example -

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
    - name: nginx
      image: nginx
  schedulerName: my-custom-scheduler
```

အရေးကြီးတဲ့ field က -

```yaml
schedulerName: my-custom-scheduler
```

ဖြစ်ပါတယ်။

ဒီ Pod ကို default scheduler မဟုတ်ဘဲ `my-custom-scheduler` က schedule လုပ်ပါမယ်။

---

## 20. Pod Create လုပ်နည်း

```bash
kubectl create -f pod-definition.yaml
```

သို့မဟုတ် -

```bash
kubectl apply -f pod-definition.yaml
```

Custom scheduler config မှားနေရင် Pod က `Pending` ဖြစ်နေနိုင်ပါတယ်။

Scheduler မှန်မှန် run နေရင် Pod က `Running` ဖြစ်သွားပါမယ်။

---

## 21. Custom Scheduler Operation Verify လုပ်နည်း

Pod ကို ဘယ် scheduler က schedule လုပ်သွားလဲ သိချင်ရင် events ကြည့်ရပါတယ်။

```bash
kubectl get events -o wide
```

Output example -

```text
LAST SEEN   COUNT   NAME        KIND   TYPE    REASON      SOURCE                  MESSAGE
9s          1       nginx.15    Pod    Normal  Scheduled   my-custom-scheduler     Successfully assigned default/nginx to node01
8s          1       nginx.15    Pod    Normal  Pulling     kubelet, node01         pulling image "nginx"
2s          1       nginx.15    Pod    Normal  Pulled      kubelet, node01         Successfully pulled image "nginx"
2s          1       nginx.15    Pod    Normal  Created     kubelet, node01         Created container
2s          1       nginx.15    Pod    Normal  Started     kubelet, node01         Started container
```

ဒီ output မှာ -

```text
SOURCE = my-custom-scheduler
```

ဆိုရင် Pod ကို custom scheduler က schedule လုပ်ထားတာ သေချာပါတယ်။

---

## 22. Scheduler Logs ကြည့်နည်း

Custom scheduler issue ရှိရင် logs ကြည့်နိုင်ပါတယ်။

```bash
kubectl logs my-custom-scheduler --namespace=kube-system
```

သို့မဟုတ် short form -

```bash
kubectl logs my-custom-scheduler -n kube-system
```

Log example -

```text
attempting to acquire leader lease kube-system/my-custom-scheduler...
successfully acquired lease kube-system/my-custom-scheduler
```

ဒီ message တွေ့ရင် scheduler က leader lease ရယူပြီး အလုပ်လုပ်နေကြောင်း သိနိုင်ပါတယ်။

---

## 23. Custom Scheduler Pending Issue

Custom scheduler နဲ့ Pod create လုပ်ပြီး Pod က `Pending` ဖြစ်နေရင် စစ်ရမယ့်အချက်တွေက -

1. Custom scheduler Pod/Deployment run နေလား
2. `schedulerName` မှန်လား
3. Scheduler config ထဲက `schedulerName` နဲ့ Pod spec ထဲက `schedulerName` ကိုက်လား
4. Scheduler logs မှာ error ရှိလား
5. RBAC permission ရှိလား
6. Leader election issue ရှိလား

Commands -

```bash
kubectl get pods -n kube-system
kubectl logs <scheduler-pod-name> -n kube-system
kubectl get events -o wide
kubectl describe pod <pod-name>
```

---

## 24. Default Scheduler vs Custom Scheduler

| Feature | Default Scheduler | Custom Scheduler |
|---|---|---|
| Name | `default-scheduler` | ကိုယ်ပေးထားတဲ့ name |
| Purpose | General pod scheduling | Custom scheduling logic |
| Built-in | Yes | User-created |
| Pod uses it by default | Yes | `schedulerName` ထည့်မှ သုံးမယ် |
| Config | Kubernetes default config | Custom KubeSchedulerConfiguration |
| Use case | Normal workloads | Special placement rules |

---

## 25. Multiple Schedulers အတွက် အရေးကြီးတဲ့ Points

- Default scheduler နဲ့ custom scheduler တို့ကို cluster ထဲမှာ အတူ run နိုင်ပါတယ်။
- Scheduler တစ်ခုချင်းစီမှာ unique name ရှိရပါမယ်။
- Pod တစ်ခုကို custom scheduler သုံးစေချင်ရင် `schedulerName` field ထည့်ရပါတယ်။
- `schedulerName` မထည့်ထားရင် default scheduler က schedule လုပ်ပါတယ်။
- Custom scheduler config မှားရင် Pod `Pending` ဖြစ်နိုင်ပါတယ်။
- Scheduler operation ကို events နဲ့ logs ကြည့်ပြီး verify လုပ်နိုင်ပါတယ်။

---

## 26. အသုံးများတဲ့ Commands

### kube-system ထဲက scheduler pods ကြည့်ရန်

```bash
kubectl get pods -n kube-system
```

### Events ကြည့်ရန်

```bash
kubectl get events -o wide
```

### Custom scheduler logs ကြည့်ရန်

```bash
kubectl logs my-custom-scheduler -n kube-system
```

### Pod details ကြည့်ရန်

```bash
kubectl describe pod nginx
```

### Pod create/apply လုပ်ရန်

```bash
kubectl apply -f pod-definition.yaml
```

### Scheduler Deployment ကြည့်ရန်

```bash
kubectl get deployment -n kube-system
```

---

## 27. CKA Exam အတွက် မှတ်ရန်

CKA exam မှာ Multiple Schedulers topic မှာ အောက်ပါအချက်တွေကို သိထားသင့်ပါတယ်။

- Default scheduler name က `default-scheduler`
- Custom scheduler မှာ unique `schedulerName` ရှိရမယ်။
- Scheduler config kind က `KubeSchedulerConfiguration`
- Pod spec ထဲမှာ `schedulerName` ထည့်ပြီး custom scheduler သုံးခိုင်းနိုင်တယ်။
- Pod `Pending` ဖြစ်ရင် scheduler config/name/logs/events စစ်ရမယ်။
- `kubectl get events -o wide` နဲ့ Pod ကို ဘယ် scheduler က schedule လုပ်လဲကြည့်နိုင်တယ်။
- Scheduler logs ကို `kubectl logs <scheduler-pod> -n kube-system` နဲ့ ကြည့်နိုင်တယ်။
- Custom scheduler ကို Pod/Deployment/Service file အဖြစ် run နိုင်တယ်။

---

## 28. Simple Summary

Multiple Schedulers ဆိုတာ Kubernetes cluster ထဲမှာ default scheduler အပြင် custom scheduler တွေကိုပါ run စေတဲ့ concept ဖြစ်ပါတယ်။

Default scheduler name -

```text
default-scheduler
```

Custom scheduler config example -

```yaml
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:
  - schedulerName: my-scheduler
```

Pod ကို custom scheduler သုံးစေချင်ရင် -

```yaml
schedulerName: my-scheduler
```

ထည့်ရပါတယ်။

Verify လုပ်ရန် -

```bash
kubectl get events -o wide
```

Logs ကြည့်ရန် -

```bash
kubectl logs my-scheduler -n kube-system
```

အရေးကြီးဆုံး မှတ်ရန် -

```text
schedulerName မထည့်ထားရင် default scheduler သုံးမယ်။
schedulerName ထည့်ထားရင် အဲ့ဒီ custom scheduler က schedule လုပ်မယ်။
schedulerName မှားရင် Pod Pending ဖြစ်နိုင်တယ်။
```
