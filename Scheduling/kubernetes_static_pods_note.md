# Kubernetes Static Pods Note

## 1. Static Pod ဆိုတာဘာလဲ

**Static Pod** ဆိုတာ Kubernetes မှာ `kube-apiserver`, `scheduler`, `controller-manager` စတဲ့ control plane components မပါဘဲ **kubelet က တိုက်ရိုက် create/manage လုပ်တဲ့ Pod** ဖြစ်ပါတယ်။

ပုံမှန် Kubernetes cluster မှာ Pod create လုပ်တဲ့ flow က ဒီလိုပါ။

```text
User -> kube-apiserver -> scheduler -> kubelet -> container runtime -> Pod
```

ဒါပေမယ့် Static Pod မှာတော့ API Server မလိုပါဘူး။

```text
Pod YAML file -> kubelet -> container runtime -> Pod
```

အဓိက မှတ်ရန် -

```text
Static Pod = kubelet က local manifest file ကိုဖတ်ပြီး create လုပ်တဲ့ Pod
```

---

## 2. Static Pod ကို ဘာကြောင့်သုံးလဲ

Static Pod ကို control plane မရှိတဲ့ standalone node မှာလည်း Pod run စေချင်တဲ့အခါ သုံးနိုင်ပါတယ်။

ဥပမာ -

- kube-apiserver မရှိသေးတဲ့အချိန်
- scheduler မရှိသေးတဲ့အချိန်
- etcd မရှိသေးတဲ့အချိန်
- node တစ်ခုတည်းမှာ kubelet နဲ့ container runtime ပဲရှိတဲ့အချိန်

ဒီလိုအခြေအနေမှာ kubelet က local directory ထဲက Pod YAML file တွေကိုဖတ်ပြီး Pod တွေ create လုပ်နိုင်ပါတယ်။

---

## 3. Static Pod အလုပ်လုပ်ပုံ

kubelet ကို static pod manifest directory တစ်ခုကို watch လုပ်ခိုင်းထားပါတယ်။

ဥပမာ -

```text
/etc/kubernetes/manifests
```

kubelet က အဲ့ဒီ directory ထဲကို အချိန်တိုင်း scan လုပ်ပါတယ်။

အလုပ်လုပ်ပုံက -

1. kubelet က manifest directory ကို ကြည့်တယ်။
2. YAML file ရှိရင် ဖတ်တယ်။
3. Pod ကို create လုပ်တယ်။
4. Pod crash ဖြစ်ရင် restart လုပ်တယ်။
5. YAML file update ဖြစ်ရင် Pod ကို recreate လုပ်တယ်။
6. YAML file delete ဖြစ်ရင် Pod ကိုလည်း delete လုပ်တယ်။

အလွယ်မှတ်ရန် -

```text
File exists  -> Pod exists
File changes -> Pod recreated
File removed -> Pod removed
```

---

## 4. Static Pod ကို ဘယ်သူ manage လုပ်လဲ

Static Pod ကို **kubelet** ကပဲ manage လုပ်ပါတယ်။

Static Pod creation မှာ မပါဝင်တဲ့ components တွေက -

- kube-apiserver
- kube-scheduler
- controller-manager
- etcd

ဒါကြောင့် Static Pod က control plane မရှိသေးတဲ့ node မှာလည်း run နိုင်ပါတယ်။

---

## 5. Static Pod နဲ့ Create လုပ်နိုင်တာ

Static Pod mechanism နဲ့ create လုပ်နိုင်တာက **Pod only** ဖြစ်ပါတယ်။

အောက်ပါ resource တွေကို static manifest directory ထဲမှာ ထည့်ပြီး create လုပ်လို့မရပါဘူး။

- ReplicaSet
- Deployment
- Service
- StatefulSet
- DaemonSet

ဘာကြောင့်လဲဆိုတော့ ဒီ resources တွေက controller-manager, API Server စတဲ့ control plane components တွေကို မဖြစ်မနေလိုအပ်လို့ပါ။

---

## 6. Static Pod Manifest Directory

Static Pod YAML files တွေထားတဲ့ directory ကို kubelet မှာ configure လုပ်ရပါတယ်။

အများအားဖြင့် kubeadm cluster တွေမှာ default path က -

```text
/etc/kubernetes/manifests
```

ဖြစ်ပါတယ်။

ဒီ directory ထဲမှာ YAML file ထည့်လိုက်ရင် kubelet က အလိုအလျောက် Pod create လုပ်ပါမယ်။

---

## 7. `--pod-manifest-path` နဲ့ Configure လုပ်နည်း

kubelet service file ထဲမှာ `--pod-manifest-path` option နဲ့ static pod directory ကို သတ်မှတ်နိုင်ပါတယ်။

Example -

```bash
ExecStart=/usr/local/bin/kubelet \
  --container-runtime=remote \
  --container-runtime-endpoint=unix:///var/run/containerd/containerd.sock \
  --pod-manifest-path=/etc/kubernetes/manifests \
  --kubeconfig=/var/lib/kubelet/kubeconfig \
  --network-plugin=cni \
  --register-node=true \
  --v=2
```

ဒီထဲက အရေးကြီးတာက -

```bash
--pod-manifest-path=/etc/kubernetes/manifests
```

ဖြစ်ပါတယ်။

အဓိပ္ပါယ်က -

```text
kubelet က /etc/kubernetes/manifests ထဲက Pod YAML တွေကို static pod အဖြစ် run မယ်
```

---

## 8. `staticPodPath` နဲ့ Configure လုပ်နည်း

နောက်ထပ်နည်းလမ်းက kubelet config file ထဲမှာ `staticPodPath` သတ်မှတ်တာပါ။

kubelet service file -

```bash
ExecStart=/usr/local/bin/kubelet \
  --container-runtime=remote \
  --container-runtime-endpoint=unix:///var/run/containerd/containerd.sock \
  --config=kubeconfig.yaml \
  --kubeconfig=/var/lib/kubelet/kubeconfig \
  --network-plugin=cni \
  --register-node=true \
  --v=2
```

kubelet config file -

```yaml
staticPodPath: /etc/kubernetes/manifests
```

kubeadm နဲ့ create လုပ်ထားတဲ့ cluster တွေမှာ ဒီ approach ကို အများအားဖြင့် သုံးပါတယ်။

---

## 9. Static Pod Path ရှာနည်း

Existing cluster တစ်ခုမှာ static pod path ကိုရှာချင်ရင် အဆင့် ၂ ဆင့်နဲ့ စစ်နိုင်ပါတယ်။

### Step 1: kubelet service file ထဲမှာ `--pod-manifest-path` ရှိမရှိ စစ်ပါ

```bash
ps -ef | grep kubelet
```

သို့မဟုတ် kubelet service file ကိုကြည့်ပါ။

```bash
systemctl cat kubelet
```

`--pod-manifest-path` ရှိရင် အဲ့ဒီ path က static pod directory ပါ။

### Step 2: `--config` option ရှိရင် config file ထဲက `staticPodPath` ကို စစ်ပါ

```bash
ps -ef | grep kubelet
```

`--config=/var/lib/kubelet/config.yaml` လိုမျိုးတွေ့ရင် -

```bash
grep staticPodPath /var/lib/kubelet/config.yaml
```

---

## 10. Standalone Environment မှာ Static Pod စစ်နည်း

Standalone node မှာ kube-apiserver မရှိတဲ့အတွက် `kubectl` command သုံးလို့မရနိုင်ပါဘူး။

ဒီအခြေအနေမှာ container runtime command နဲ့ စစ်ရပါတယ်။

Docker သုံးရင် -

```bash
docker ps
```

Output example -

```console
CONTAINER ID        IMAGE                  COMMAND                 STATUS
8e5d4c4db7b6        busybox                "sh -c 'echo Hello...'" Up 20 seconds
f6737e1149cb        k8s.gcr.io/pause:3.1   "/pause"                Up 23 seconds
```

containerd သုံးရင် -

```bash
crictl ps
```

CKA exam မှာ containerd/crictl ကို ပိုတွေ့ရနိုင်ပါတယ်။

---

## 11. Cluster ထဲမှာ Static Pod အလုပ်လုပ်ပုံ

Node တစ်ခုက Kubernetes cluster ထဲမှာ ပါနေတဲ့အခါ kubelet က pod sources ၂ မျိုးကို handle လုပ်နိုင်ပါတယ်။

1. API Server ကပေးတဲ့ Pod definitions
2. Local static pod manifest directory ထဲက Pod definitions

Static Pod ကို kubelet က create လုပ်တဲ့အခါ API Server ထဲမှာ **mirror pod object** တစ်ခုလည်း create လုပ်ပေးပါတယ်။

ဒါကြောင့် `kubectl get pods` နဲ့ Static Pod ကို မြင်ရနိုင်ပါတယ်။

Example -

```bash
kubectl get pods
```

Output -

```console
NAME                READY   STATUS              RESTARTS   AGE
static-web-node01   0/1     ContainerCreating   0          29s
```

Pod name မှာ node name ပါလာတတ်ပါတယ်။

```text
static-web-node01
```

ဒီမှာ `node01` က Pod ကို run နေတဲ့ node name ဖြစ်ပါတယ်။

![Static Pods Architecture](https://kodekloud.com/kk-media/image/upload/v1752869910/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-Static-Pods/frame_340.jpg)

---

## 12. Mirror Pod ဆိုတာဘာလဲ

**Mirror Pod** ဆိုတာ kubelet က Static Pod ကို cluster API Server မှာ read-only object အဖြစ် ပြန်ပြီး register လုပ်ထားတာပါ။

Mirror Pod ရဲ့အချက်တွေက -

- `kubectl get pods` နဲ့ မြင်ရတယ်။
- API Server က create လုပ်တာ မဟုတ်ဘူး။
- kubelet က local manifest ကနေ create လုပ်ထားတဲ့ Static Pod ကို represent လုပ်တာပါ။
- `kubectl edit` နဲ့ ပြင်လို့မရဘူး။
- `kubectl delete` နဲ့ ဖျက်လို့မရဘူး။
- ပြင်ချင်ရင် node ပေါ်က manifest YAML file ကိုပဲ ပြင်ရမယ်။
- ဖျက်ချင်ရင် manifest YAML file ကို ဖျက်ရမယ်။

အလွယ်မှတ်ရန် -

```text
Static Pod ကို manage လုပ်ချင်ရင် API မဟုတ်ဘဲ manifest file ကို ပြင်ရမယ်
```

---

## 13. Static Pod Update လုပ်နည်း

Static Pod ကို update လုပ်ချင်ရင် manifest file ကို edit လုပ်ရပါတယ်။

ဥပမာ -

```bash
vi /etc/kubernetes/manifests/static-web.yaml
```

File ကို save လုပ်လိုက်ရင် kubelet က change ကို detect လုပ်ပြီး Pod ကို recreate လုပ်ပါမယ်။

---

## 14. Static Pod Delete လုပ်နည်း

Static Pod ကို delete လုပ်ချင်ရင် manifest file ကို remove လုပ်ရပါတယ်။

ဥပမာ -

```bash
rm /etc/kubernetes/manifests/static-web.yaml
```

File ဖယ်လိုက်တာနဲ့ kubelet က corresponding Pod ကို delete လုပ်ပါမယ်။

`kubectl delete pod <pod-name>` နဲ့ delete လုပ်လို့မရပါဘူး။ Delete လုပ်သွားသလို မြင်ရနိုင်ပေမယ့် kubelet က manifest file ရှိနေသေးရင် Pod ကို ပြန် create လုပ်ပါမယ်။

---

## 15. Static Pods and Control Plane Components

Static Pod ကို Kubernetes control plane components တွေ deploy လုပ်ရာမှာ အများကြီးသုံးပါတယ်။

kubeadm cluster တွေမှာ အောက်ပါ components တွေက static pod အဖြစ် run တတ်ပါတယ်။

- kube-apiserver
- kube-controller-manager
- kube-scheduler
- etcd

အဲဒီ manifest files တွေကို အများအားဖြင့် ဒီ path မှာ တွေ့ရပါတယ်။

```text
/etc/kubernetes/manifests
```

ဥပမာ -

```bash
ls /etc/kubernetes/manifests
```

Output example -

```text
etcd.yaml
kube-apiserver.yaml
kube-controller-manager.yaml
kube-scheduler.yaml
```

---

## 16. Static Pods vs DaemonSets

| Feature | Static Pods | DaemonSets |
|---|---|---|
| Creation Source | kubelet က local manifest file ကနေ create လုပ်တယ် | DaemonSet controller က API Server ကနေ create လုပ်တယ် |
| Control Plane Involvement | API Server မလိုပါ | API Server လိုပါတယ် |
| Main Use Case | Critical control plane components | Node တိုင်းမှာ Pod တစ်ခုစီ run စေချင်တဲ့ agents |
| Scheduler | kube-scheduler မသုံးပါ | kube-scheduler ကို bypass/ignore လုပ်တဲ့ behavior ရှိတယ် |
| Manage Method | Node ပေါ်က YAML file ပြင်ရတယ် | `kubectl apply/edit/delete ds` နဲ့ manage လုပ်တယ် |
| Example | kube-apiserver, etcd | kube-proxy, log collector, monitoring agent |

![Static Pods vs DaemonSets](https://kodekloud.com/kk-media/image/upload/v1752869911/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-Static-Pods/frame_490.jpg)

---

## 17. Static Pod vs Normal Pod

| Feature | Static Pod | Normal Pod |
|---|---|---|
| Created by | kubelet directly | API Server |
| Stored in etcd | Mirror object only | Yes |
| Scheduler involved | No | Yes |
| Delete method | Delete manifest file | `kubectl delete pod` |
| Update method | Edit manifest file | `kubectl edit/apply` |
| Use case | Control plane/bootstrap workloads | General application workloads |

---

## 18. Static Pod Example Manifest

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: static-web
spec:
  containers:
    - name: nginx
      image: nginx
      ports:
        - containerPort: 80
```

ဒီ file ကို static pod path ထဲမှာ ထည့်ပါ။

```bash
cp static-web.yaml /etc/kubernetes/manifests/
```

kubelet က အလိုအလျောက် Pod create လုပ်ပါမယ်။

စစ်ရန် -

```bash
kubectl get pods
```

သို့မဟုတ် standalone node မှာ -

```bash
crictl ps
```

---

## 19. CKA Exam အတွက် အသုံးများတဲ့ Commands

### kubelet process ကြည့်ရန်

```bash
ps -ef | grep kubelet
```

### kubelet service config ကြည့်ရန်

```bash
systemctl cat kubelet
```

### staticPodPath ရှာရန်

```bash
grep staticPodPath /var/lib/kubelet/config.yaml
```

### Static pod manifests ကြည့်ရန်

```bash
ls /etc/kubernetes/manifests
```

### Manifest file edit လုပ်ရန်

```bash
vi /etc/kubernetes/manifests/<file-name>.yaml
```

### Static pod delete လုပ်ရန်

```bash
rm /etc/kubernetes/manifests/<file-name>.yaml
```

### Pods ကြည့်ရန်

```bash
kubectl get pods -A
```

### Container runtime နဲ့ ကြည့်ရန်

```bash
crictl ps
```

သို့မဟုတ် Docker သုံးထားရင် -

```bash
docker ps
```

---

## 20. CKA Exam အတွက် မှတ်ရန်

CKA မှာ Static Pods က အရေးကြီးတဲ့ topic တစ်ခုပါ။

အဓိက မှတ်ထားရမယ့်အချက်တွေက -

- Static Pod ကို kubelet က local manifest file ကနေ create လုပ်တယ်။
- API Server မလိုဘဲ run နိုင်တယ်။
- Static Pod file path ကို kubelet config ထဲက `staticPodPath` မှာ သတ်မှတ်တယ်။
- Common path က `/etc/kubernetes/manifests`
- Static Pod ကို `kubectl edit/delete` နဲ့ တကယ် manage မလုပ်နိုင်ဘူး။
- Update လုပ်ချင်ရင် manifest file ကို edit လုပ်ရမယ်။
- Delete လုပ်ချင်ရင် manifest file ကို remove လုပ်ရမယ်။
- kubeadm cluster မှာ control plane components တွေ static pod အဖြစ် run တတ်တယ်။
- Static Pod က mirror pod အဖြစ် API Server မှာ မြင်ရနိုင်တယ်။
- Pod name မှာ node name suffix ပါတတ်တယ်။

---

## 21. Simple Summary

Static Pod ဆိုတာ **kubelet က local YAML manifest file ကိုဖတ်ပြီး တိုက်ရိုက် create/manage လုပ်တဲ့ Pod** ပါ။

အများဆုံးတွေ့ရတဲ့ static pod path က -

```text
/etc/kubernetes/manifests
```

ဒီ directory ထဲမှာ Pod YAML file ထည့်ရင် kubelet က Pod ကို create လုပ်မယ်။

File update လုပ်ရင် Pod recreate ဖြစ်မယ်။

File delete လုပ်ရင် Pod delete ဖြစ်မယ်။

အရေးကြီးဆုံး မှတ်ရန် -

```text
Static Pod = kubelet managed pod
```

```text
Manage Static Pod = edit/delete manifest file
```

```text
Common path = /etc/kubernetes/manifests
```

Static Pod ကို kubeadm cluster တွေမှာ control plane components ဖြစ်တဲ့ kube-apiserver, controller-manager, scheduler, etcd တို့ကို run ဖို့ အသုံးပြုပါတယ်။
