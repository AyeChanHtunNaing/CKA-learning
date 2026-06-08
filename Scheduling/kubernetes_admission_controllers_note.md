# Kubernetes Admission Controllers Note

## 1. Admission Controller ဆိုတာဘာလဲ

**Admission Controller** ဆိုတာ Kubernetes API Server ထဲမှာ request တစ်ခုကို etcd ထဲမသိမ်းခင် စစ်ဆေး၊ ပြင်ဆင်၊ reject လုပ်နိုင်တဲ့ mechanism ပါ။

ဥပမာ `kubectl run`, `kubectl create`, `kubectl apply` လုပ်လိုက်တဲ့ request တွေက အရင်ဆုံး API Server ကို ရောက်ပါတယ်။

Request flow က အကြမ်းဖျဉ်း ဒီလိုပါ။

```text
kubectl request
      ↓
API Server
      ↓
Authentication
      ↓
Authorization
      ↓
Admission Controllers
      ↓
etcd
```

Admission Controller က object ကို etcd ထဲ persist မလုပ်ခင် နောက်ဆုံးအဆင့် policy စစ်ဆေးတဲ့နေရာလို့ မှတ်နိုင်ပါတယ်။

---

## 2. API Server Request Flow

Pod တစ်ခု create လုပ်တဲ့အခါ request က အောက်ပါအဆင့်တွေ ဖြတ်သွားပါတယ်။

### Step 1: Authentication

Authentication က user ဘယ်သူလဲဆိုတာ စစ်ပါတယ်။

ဥပမာ `kubectl` သုံးတဲ့အခါ certificate information တွေကို kubeconfig file ထဲကနေ ယူနိုင်ပါတယ်။

```bash
cat ~/.kube/config
```

Example output -

```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVU...
```

အဓိပ္ပါယ်က API Server က request လုပ်လာတဲ့ user/client ကို အတည်ပြုပါတယ်။

---

### Step 2: Authorization

Authorization က authenticated user က ဒီ action လုပ်ခွင့်ရှိမရှိ စစ်ပါတယ်။

Kubernetes မှာ RBAC ကို အများအားဖြင့် သုံးပါတယ်။

ဥပမာ developer role တစ်ခုမှာ pods ကို list/get/create/update/delete လုပ်ခွင့်ပေးထားတာ -

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["list", "get", "create", "update", "delete"]
```

ဒီ Role ကို user တစ်ယောက်နဲ့ bind လုပ်ထားရင် အဲ့ဒီ user က pods ကို အထက်ပါ actions တွေ လုပ်နိုင်ပါတယ်။

---

### Step 3: Admission Control

Authentication နဲ့ Authorization အောင်မြင်ပြီးရင် Admission Controllers က request object ရဲ့ content ကို စစ်ပါတယ်။

ဥပမာ -

- Pod image က allowed registry ကနေ လာတာလား
- `latest` tag သုံးထားလား
- Container က root user နဲ့ run နေလား
- Required labels ပါလား
- Namespace ရှိလား
- Security policy ကိုက်လား

ဒီအဆင့်မှာ object ကို reject လုပ်နိုင်သလို mutate/change လုပ်နိုင်ပါတယ်။

---

## 3. RBAC ရဲ့ Limitation

RBAC က API-level permission ကိုပဲ စစ်ပါတယ်။

ဥပမာ Role ထဲမှာ pods create permission ပေးထားရင် user က Pod create လုပ်နိုင်ပါတယ်။

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["create"]
  resourceNames: ["blue", "orange"]
```

ဒီ config က developer ကို `blue` နဲ့ `orange` ဆိုတဲ့ Pod names တွေပဲ create လုပ်ခွင့်ပေးနိုင်ပါတယ်။

ဒါပေမယ့် RBAC က Pod YAML ထဲက content ကို detail မစစ်နိုင်ပါဘူး။

ဥပမာ RBAC တစ်ခုတည်းနဲ့ အောက်ပါ policy တွေ enforce လုပ်လို့မရပါဘူး။

- Public registry image မသုံးရ
- `latest` image tag မသုံးရ
- Container ကို root user နဲ့ run မလုပ်ရ
- Specific capability မထည့်ရ
- Required labels မပါရင် reject လုပ်ရ

ဒီလို object content-level policy အတွက် Admission Controllers ကို သုံးရပါတယ်။

---

## 4. Admission Controller က ဘာလုပ်နိုင်လဲ

Admission Controller က request ကို etcd ထဲမသိမ်းခင် အောက်ပါအရာတွေ လုပ်နိုင်ပါတယ်။

1. **Validate**  
   Request object မှန်မမှန် စစ်ပြီး မမှန်ရင် reject လုပ်နိုင်ပါတယ်။

2. **Mutate**  
   Request object ကို modify/change လုပ်နိုင်ပါတယ်။

3. **Enforce policy**  
   Cluster policy တွေကို enforce လုပ်နိုင်ပါတယ်။

ဥပမာ -

```text
If image tag is latest -> reject
If namespace does not exist -> reject
If default storage class missing -> add default storage class
If required label missing -> add or reject
```

---

## 5. Example: Problematic Pod Specification

အောက်က Pod spec ကို ကြည့်ပါ။

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-pod
spec:
  containers:
    - name: ubuntu
      image: ubuntu:latest
      command: ["sleep", "3600"]
      securityContext:
        runAsUser: 0
      capabilities:
        add: ["MAC_ADMIN"]
```

ဒီ Pod မှာ security concern အချို့ရှိပါတယ်။

- `ubuntu:latest` image tag သုံးထားတယ်။
- Container က `runAsUser: 0` ဖြစ်လို့ root user နဲ့ run နေတယ်။
- `MAC_ADMIN` capability ထည့်ထားတယ်။

RBAC တစ်ခုတည်းနဲ့ ဒီလို content-level issue တွေကို စစ်လို့မရပါဘူး။

Admission Controller ကတော့ ဒီ object ကို inspect လုပ်ပြီး reject လုပ်နိုင်ပါတယ်။

---

## 6. Built-in Admission Controllers Examples

Kubernetes မှာ built-in admission controllers များစွာ ပါပါတယ်။

အသုံးများတဲ့ examples -

| Admission Controller | အလုပ် |
|---|---|
| `AlwaysPullImages` | Pod တိုင်း image ကို registry ကနေ အမြဲ pull လုပ်စေတယ် |
| `DefaultStorageClass` | PVC မှာ storageClass မပါရင် default storage class ထည့်ပေးတယ် |
| `EventRateLimit` | API Server request/event rate ကို limit လုပ်တယ် |
| `NamespaceExists` | Namespace ရှိမရှိ စစ်ပြီး မရှိရင် reject လုပ်တယ် |
| `NamespaceLifecycle` | Non-existent namespace ကို reject လုပ်ပြီး default namespaces တွေ delete မလုပ်အောင် ကာကွယ်တယ် |
| `ServiceAccount` | Pod တွေမှာ service account behavior ကို handle လုပ်တယ် |
| `LimitRanger` | Namespace ထဲက resource limits/defaults တွေ enforce လုပ်တယ် |
| `NodeRestriction` | kubelet/node identity နဲ့ဆိုင်တဲ့ access restrictions တွေ enforce လုပ်တယ် |

---

## 7. Namespace Admission Controller

Namespace admission controller က Pod ကို create လုပ်ချင်တဲ့ namespace ရှိ/မရှိ စစ်ပါတယ်။

ဥပမာ -

```bash
kubectl run nginx --image nginx --namespace blue
```

`blue` namespace မရှိရင် error ဖြစ်ပါမယ်။

```bash
Error from server (NotFound): namespaces "blue" not found
```

ဒီ flow မှာ -

```text
Authentication passed
Authorization passed
Namespace admission controller checks namespace
blue namespace does not exist
Request rejected
```

ဖြစ်ပါတယ်။

---

## 8. Namespace Auto-Provision Admission Controller

Namespace auto-provision admission controller က namespace မရှိရင် အလိုအလျောက် create လုပ်ပေးနိုင်တဲ့ controller ပါ။

ဥပမာ auto-provision enabled ဖြစ်နေတဲ့အခါ -

```bash
kubectl run nginx --image nginx --namespace blue
```

လုပ်လိုက်ရင် `blue` namespace ကို automatically create လုပ်ပြီး Pod ကို create လုပ်နိုင်ပါတယ်။

ပြီးရင် namespaces ကြည့်ရင် -

```bash
kubectl get namespaces
```

Output example -

```bash
NAME         STATUS   AGE
blue         Active   3m
default      Active   23m
kube-public  Active   24m
kube-system  Active   24m
```

ဒါပေမယ့် သတိထားရန် -

```text
NamespaceExists နဲ့ NamespaceAutoProvision admission controllers တွေက deprecated ဖြစ်ပါတယ်။
```

အစားထိုးအနေနဲ့ `NamespaceLifecycle` admission controller ကို သုံးပါတယ်။

---

## 9. NamespaceLifecycle Admission Controller

`NamespaceLifecycle` admission controller က namespace နဲ့ဆိုင်တဲ့ important rules တွေ enforce လုပ်ပါတယ်။

အဓိကလုပ်တာတွေက -

- Non-existent namespace ထဲ resource create လုပ်တာကို reject လုပ်တယ်။
- Default namespaces တွေကို delete မလုပ်အောင် ကာကွယ်တယ်။

Protected default namespaces examples -

```text
default
kube-system
kube-public
```

အလွယ်မှတ်ရန် -

```text
NamespaceLifecycle = namespace existence + default namespace protection
```

---

## 10. Enabled Admission Controllers ကြည့်နည်း

API Server မှာ default enabled admission plugins တွေကိုကြည့်ချင်ရင် -

```bash
kube-apiserver -h | grep enable-admission-plugins
```

kubeadm-based cluster တွေမှာ kube-apiserver က static pod အဖြစ် run နေတတ်တဲ့အတွက် control plane pod ထဲမှာ command run ရနိုင်ပါတယ်။

ဥပမာ -

```bash
kubectl exec -n kube-system kube-apiserver-controlplane -- kube-apiserver -h | grep enable-admission-plugins
```

သို့မဟုတ် kube-apiserver manifest ထဲက flag ကို စစ်နိုင်ပါတယ်။

```bash
cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep admission
```

---

## 11. Admission Controller Enable လုပ်နည်း

Admission controller ထည့်ချင်ရင် kube-apiserver ရဲ့ flag ကို update လုပ်ရပါတယ်။

Flag က -

```bash
--enable-admission-plugins
```

Example -

```bash
--enable-admission-plugins=NodeRestriction,NamespaceAutoProvision
```

Systemd service file example -

```plaintext
ExecStart=/usr/local/bin/kube-apiserver   --advertise-address=${INTERNAL_IP}   --allow-privileged=true   --apiserver-count=3   --authorization-mode=Node,RBAC   --bind-address=0.0.0.0   --enable-swagger-ui=true   --etcd-servers=https://127.0.0.1:2379   --event-ttl=1h   --runtime-config=api/all   --service-cluster-ip-range=10.32.0.0/24   --service-node-port-range=30000-32767   --v=2   --enable-admission-plugins=NodeRestriction,NamespaceAutoProvision
```

---

## 12. kubeadm Setup မှာ Enable လုပ်နည်း

kubeadm setup တွေမှာ kube-apiserver က static pod အဖြစ် run နေတာများပါတယ်။

Manifest file path က အများအားဖြင့် -

```text
/etc/kubernetes/manifests/kube-apiserver.yaml
```

Example manifest -

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-apiserver
    - --authorization-mode=Node,RBAC
    - --advertise-address=172.17.0.107
    - --allow-privileged=true
    - --enable-bootstrap-token-auth=true
    - --enable-admission-plugins=NodeRestriction,NamespaceAutoProvision
    image: k8s.gcr.io/kube-apiserver-amd64:v1.11.3
    name: kube-apiserver
```

ဒီ file ကို edit လုပ်ပြီး save လုပ်လိုက်ရင် kubelet က kube-apiserver static pod ကို recreate လုပ်ပါမယ်။

---

## 13. Admission Controller Disable လုပ်နည်း

Admission controller တချို့ကို disable လုပ်ချင်ရင် kube-apiserver flag ကို သုံးပါတယ်။

```bash
--disable-admission-plugins
```

Example -

```bash
--disable-admission-plugins=DefaultStorageClass
```

kubeadm cluster မှာ kube-apiserver manifest ထဲမှာ ထည့်ပြင်ရပါမယ်။

---

## 14. Admission Controller Types

Admission Controllers ကို အဓိက ၂ မျိုးခွဲနိုင်ပါတယ်။

### 1. Mutating Admission Controller

Request object ကို modify လုပ်နိုင်ပါတယ်။

ဥပမာ -

```text
PVC မှာ storageClass မပါရင် default storageClass ထည့်ပေးတယ်
Pod မှာ default serviceAccount ထည့်ပေးတယ်
```

### 2. Validating Admission Controller

Request object ကို စစ်ပြီး allow/reject လုပ်ပါတယ်။

ဥပမာ -

```text
Namespace မရှိရင် reject
Policy မကိုက်ရင် reject
Security rule မကိုက်ရင် reject
```

အလွယ်မှတ်ရန် -

```text
Mutating = object ကို ပြင်နိုင်တယ်
Validating = object ကို စစ်ပြီး accept/reject လုပ်တယ်
```

---

## 15. Admission Controller Use Cases

Admission Controllers ကို အောက်ပါ policy enforcement တွေအတွက် အသုံးပြုနိုင်ပါတယ်။

- Public image registry မသုံးစေချင်
- Image tag `latest` မသုံးစေချင်
- Container ကို root user နဲ့ မ run စေချင်
- Required labels/annotations တွေ မဖြစ်မနေ ထည့်စေချင်
- Default storage class automatically ထည့်ချင်
- Namespace lifecycle ကို control လုပ်ချင်
- ServiceAccount behavior ကို enforce လုပ်ချင်
- Resource limits/defaults enforce လုပ်ချင်

---

## 16. Admission Controller vs RBAC

| Feature | RBAC | Admission Controller |
|---|---|---|
| Main purpose | User/action permission စစ်ခြင်း | Object content/policy စစ်ခြင်း |
| Works at | API permission level | Object admission level |
| Example | User can create pods | Pod must not use `latest` image |
| Can inspect Pod spec? | မလုပ်နိုင် | လုပ်နိုင် |
| Can mutate object? | မလုပ်နိုင် | လုပ်နိုင် |
| Can reject policy violation? | Limited | လုပ်နိုင် |

အလွယ်မှတ်ရန် -

```text
RBAC = Who can do what?
Admission Controller = Is the object allowed?
```

---

## 17. Admission Controller Flow

```text
User runs kubectl
      ↓
API Server receives request
      ↓
Authentication
      ↓
Authorization / RBAC
      ↓
Admission Controllers
      ↓
Object stored in etcd
```

Admission Controller က etcd ထဲ သိမ်းမယ့် object ကို နောက်ဆုံးစစ်တဲ့ policy gate ဖြစ်ပါတယ်။

---

## 18. CKA Exam အတွက် အသုံးများတဲ့ Commands

### kube-apiserver manifest ကြည့်ရန်

```bash
cat /etc/kubernetes/manifests/kube-apiserver.yaml
```

### Admission plugins flag ရှာရန်

```bash
grep enable-admission-plugins /etc/kubernetes/manifests/kube-apiserver.yaml
```

### Disable plugins flag ရှာရန်

```bash
grep disable-admission-plugins /etc/kubernetes/manifests/kube-apiserver.yaml
```

### kube-apiserver help ကြည့်ရန်

```bash
kube-apiserver -h | grep enable-admission-plugins
```

### kube-apiserver pod ကြည့်ရန်

```bash
kubectl get pods -n kube-system | grep kube-apiserver
```

### kube-apiserver logs ကြည့်ရန်

```bash
kubectl logs -n kube-system kube-apiserver-controlplane
```

### Namespace create လုပ်ရန်

```bash
kubectl create namespace blue
```

### Namespace မရှိဘဲ Pod create စမ်းရန်

```bash
kubectl run nginx --image nginx --namespace blue
```

---

## 19. CKA Exam အတွက် မှတ်ရန်

CKA exam မှာ Admission Controllers နဲ့ပတ်သက်ပြီး အောက်ပါအချက်တွေကို သိထားသင့်ပါတယ်။

- Admission Controller က API Server request ကို etcd ထဲမသိမ်းခင် စစ်တယ်။
- Request flow က Authentication → Authorization → Admission Control → etcd ဖြစ်တယ်။
- RBAC က API-level permission စစ်တယ်။
- Admission Controller က object content/policy စစ်နိုင်တယ်။
- Admission Controller က request ကို mutate သို့မဟုတ် validate လုပ်နိုင်တယ်။
- Enable လုပ်ဖို့ kube-apiserver မှာ `--enable-admission-plugins` flag သုံးတယ်။
- Disable လုပ်ဖို့ `--disable-admission-plugins` flag သုံးတယ်။
- kubeadm setup မှာ `/etc/kubernetes/manifests/kube-apiserver.yaml` ကို edit လုပ်ရတတ်တယ်။
- `NamespaceExists` နဲ့ `NamespaceAutoProvision` က deprecated ဖြစ်ပြီး `NamespaceLifecycle` နဲ့ အစားထိုးထားတယ်။
- `NamespaceLifecycle` က non-existent namespace ကို reject လုပ်ပြီး default namespaces တွေကို delete မလုပ်အောင်ကာကွယ်တယ်။

---

## 20. Simple Summary

Admission Controller ဆိုတာ Kubernetes API Server ထဲမှာ request object ကို etcd ထဲမသိမ်းခင် စစ်ဆေးတဲ့ policy enforcement layer ပါ။

Request flow -

```text
kubectl -> API Server -> Authentication -> Authorization -> Admission Controller -> etcd
```

RBAC က -

```text
User က action လုပ်ခွင့်ရှိလား
```

ကို စစ်ပါတယ်။

Admission Controller က -

```text
Object content က policy ကိုက်လား
```

ကို စစ်ပါတယ်။

Enable လုပ်ရန် -

```bash
--enable-admission-plugins=NodeRestriction,NamespaceLifecycle
```

Disable လုပ်ရန် -

```bash
--disable-admission-plugins=<plugin-name>
```

kubeadm cluster မှာ config ပြင်ရမယ့် file က -

```text
/etc/kubernetes/manifests/kube-apiserver.yaml
```

အရေးကြီးဆုံး မှတ်ရန် -

```text
Admission Controller = Validate or mutate requests before saving to etcd
```
