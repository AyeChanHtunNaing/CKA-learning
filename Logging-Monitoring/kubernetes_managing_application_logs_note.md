# Kubernetes Managing Application Logs Note

## 1. Managing Application Logs ဆိုတာဘာလဲ

**Application Logs** ဆိုတာ application/container တွေ run နေစဉ်မှာ ဖြစ်ပျက်နေတဲ့ events, errors, requests, status messages တွေကို record လုပ်ထားတဲ့ output တွေပါ။

Kubernetes မှာ application troubleshooting လုပ်တဲ့အခါ logs ကြည့်တတ်ဖို့ အရေးကြီးပါတယ်။

Logs ကိုသုံးပြီး စစ်နိုင်တာတွေက -

- Application start ဖြစ်လား
- Error ဖြစ်နေလား
- User request တွေဝင်နေလား
- Container crash ဖြစ်လား
- Application behavior မှန်လား
- Debugging information ရှိလား

အလွယ်မှတ်ရန် -

```text
Logs = Application ဘာဖြစ်နေလဲ သိဖို့ အရေးကြီးတဲ့ output
```

---

## 2. Docker မှာ Logging အလုပ်လုပ်ပုံ

Docker container တွေက ပုံမှန်အားဖြင့် logs ကို **standard output (stdout)** နဲ့ **standard error (stderr)** ကို ရေးပါတယ်။

ဥပမာ `kodekloud/event-simulator` container ကို run လုပ်ရင် terminal မှာ logs တွေ တိုက်ရိုက်ပြပါမယ်။

```bash
docker run kodekloud/event-simulator
```

Output example -

```text
2018-10-06 15:57:15,937 - root - INFO - USER1 logged in
2018-10-06 15:57:16,943 - root - INFO - USER2 logged out
2018-10-06 15:57:17,944 - root - INFO - USER3 is viewing page3
2018-10-06 15:57:18,951 - root - INFO - USER4 is viewing page1
2018-10-06 15:57:19,954 - root - INFO - USER1 logged out
```

ဒီ logs တွေက application က stdout ကိုရေးထားတဲ့ output တွေပါ။

---

## 3. Docker Detached Mode Logs

Container ကို detached mode နဲ့ run လုပ်ရင် terminal မှာ logs တိုက်ရိုက်မပြပါဘူး။

Detached mode နဲ့ run ရန် -

```bash
docker run -d kodekloud/event-simulator
```

ဒီ command က container ကို background မှာ run စေပါတယ်။

Logs ကြည့်ရန် -

```bash
docker logs <container_id>
```

Live logs stream ကြည့်ချင်ရင် `-f` flag သုံးပါတယ်။

```bash
docker logs -f <container_id>
```

အလွယ်မှတ်ရန် -

```text
docker logs -f = Follow/live stream logs
```

---

## 4. Kubernetes မှာ Logging အလုပ်လုပ်ပုံ

Docker image တစ်ခုကို Kubernetes Pod အဖြစ် run လုပ်ရင် Kubernetes က Pod/container logs ကို ကြည့်နိုင်အောင် `kubectl logs` command ပေးထားပါတယ်။

ဥပမာ event simulator Pod YAML -

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: event-simulator-pod
spec:
  containers:
    - name: event-simulator
      image: kodekloud/event-simulator
```

ဒီ Pod ကို create လုပ်ရန် -

```bash
kubectl create -f event-simulator.yaml
```

သို့မဟုတ် -

```bash
kubectl apply -f event-simulator.yaml
```

---

## 5. Pod Logs ကြည့်နည်း

Pod run နေပြီဆိုရင် logs ကြည့်ရန် -

```bash
kubectl logs event-simulator-pod
```

Live logs stream ကြည့်ချင်ရင် -

```bash
kubectl logs -f event-simulator-pod
```

Output example -

```text
2018-10-06 15:57:15,937 - root - INFO - USER1 logged in
2018-10-06 15:57:16,943 - root - INFO - USER2 logged out
2018-10-06 15:57:17,944 - root - INFO - USER2 is viewing page2
2018-10-06 15:57:18,951 - root - INFO - USER3 is viewing page3
2018-10-06 15:57:20,095 - root - INFO - USER4 is viewing page1
```

အလွယ်မှတ်ရန် -

```text
kubectl logs <pod-name> = Pod logs ကြည့်ရန်
kubectl logs -f <pod-name> = Live logs ကြည့်ရန်
```

---

## 6. `kubectl logs -f` ဆိုတာဘာလဲ

`-f` က **follow** ကို ဆိုလိုပါတယ်။

Linux မှာ `tail -f` နဲ့ဆင်တူပါတယ်။

```bash
kubectl logs -f event-simulator-pod
```

ဒီ command က logs အသစ်တွေ ထပ်ထွက်လာသမျှ terminal မှာ live ပြပေးပါတယ်။

Troubleshooting လုပ်တဲ့အခါ အသုံးများပါတယ်။

---

## 7. Multiple Containers in a Pod

Kubernetes Pod တစ်ခုထဲမှာ container တစ်ခုထက်ပိုပြီး run နိုင်ပါတယ်။

ဥပမာ -

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: event-simulator-pod
spec:
  containers:
    - name: event-simulator
      image: kodekloud/event-simulator
    - name: image-processor
      image: some-image-processor
```

ဒီ Pod မှာ containers ၂ ခုရှိပါတယ်။

```text
event-simulator
image-processor
```

---

## 8. Multi-Container Pod Logs ကြည့်နည်း

Pod တစ်ခုထဲမှာ container တစ်ခုထက်ပိုရှိရင် `kubectl logs <pod-name>` တစ်ခုတည်းနဲ့ ကြည့်လို့မရနိုင်ပါဘူး။

Container name ကိုပါ specify လုပ်ရပါတယ်။

```bash
kubectl logs -f event-simulator-pod event-simulator
```

သို့မဟုတ် `-c` flag သုံးနိုင်ပါတယ်။

```bash
kubectl logs -f event-simulator-pod -c event-simulator
```

အရေးကြီးဆုံး မှတ်ရန် -

```text
Multi-container pod မှာ logs ကြည့်ရင် container name ထည့်ရမယ်။
```

---

## 9. Container Name မထည့်ရင် ဘာဖြစ်မလဲ

Pod ထဲမှာ container များနေတဲ့အခါ container name မထည့်ဘဲ logs ကြည့်ရင် error ဖြစ်နိုင်ပါတယ်။

ဥပမာ -

```bash
kubectl logs event-simulator-pod
```

Kubernetes က ဘယ် container ရဲ့ logs ကို ပြရမလဲ မသိလို့ container name တောင်းပါလိမ့်မယ်။

ဖြေရှင်းနည်း -

```bash
kubectl logs event-simulator-pod -c event-simulator
```

သို့မဟုတ် -

```bash
kubectl logs event-simulator-pod event-simulator
```

---

## 10. Previous Container Logs ကြည့်နည်း

Container crash ဖြစ်ပြီး restart ဖြစ်သွားတဲ့အခါ လက်ရှိ container logs မဟုတ်ဘဲ previous container logs ကို ကြည့်ချင်ရင် `--previous` flag သုံးနိုင်ပါတယ်။

```bash
kubectl logs <pod-name> --previous
```

Container name ပါလိုရင် -

```bash
kubectl logs <pod-name> -c <container-name> --previous
```

ဒီ command က CrashLoopBackOff troubleshooting မှာ အလွန်အသုံးဝင်ပါတယ်။

---

## 11. Recent Logs ကြည့်နည်း

နောက်ဆုံး logs အချို့ပဲ ကြည့်ချင်ရင် `--tail` သုံးနိုင်ပါတယ်။

```bash
kubectl logs event-simulator-pod --tail=20
```

အဓိပ္ပါယ် -

```text
နောက်ဆုံး log line 20 ကြောင်းပဲ ပြပါ
```

Live stream နဲ့ပေါင်းသုံးနိုင်ပါတယ်။

```bash
kubectl logs -f event-simulator-pod --tail=20
```

---

## 12. Time-based Logs ကြည့်နည်း

နောက်ဆုံး ၁ နာရီအတွင်း logs ကြည့်ချင်ရင် `--since` သုံးနိုင်ပါတယ်။

```bash
kubectl logs event-simulator-pod --since=1h
```

နောက်ဆုံး ၁၀ မိနစ်အတွင်း logs ကြည့်ချင်ရင် -

```bash
kubectl logs event-simulator-pod --since=10m
```

---

## 13. Deployment Logs ကြည့်နည်း

Pod name မသိရင် Deployment resource ကနေ logs ကြည့်နိုင်ပါတယ်။

```bash
kubectl logs deployment/<deployment-name>
```

ဥပမာ -

```bash
kubectl logs deployment/nginx
```

Live logs -

```bash
kubectl logs -f deployment/nginx
```

Deployment ထဲမှာ Pod replicas များရင် Kubernetes က matching Pod တစ်ခုရဲ့ logs ကို ပြနိုင်ပါတယ်။

---

## 14. Label Selector နဲ့ Logs ကြည့်နည်း

Label selector သုံးပြီး Pod logs တွေကြည့်နိုင်ပါတယ်။

```bash
kubectl logs -l app=nginx
```

Live follow -

```bash
kubectl logs -f -l app=nginx
```

Container name ပါလိုရင် -

```bash
kubectl logs -l app=nginx -c nginx
```

---

## 15. Namespace ထဲက Logs ကြည့်နည်း

Pod က default namespace မဟုတ်တဲ့ namespace ထဲမှာရှိရင် `-n` flag သုံးရပါတယ်။

```bash
kubectl logs event-simulator-pod -n dev
```

Live logs -

```bash
kubectl logs -f event-simulator-pod -n dev
```

Multi-container pod -

```bash
kubectl logs -f event-simulator-pod -c event-simulator -n dev
```

---

## 16. Logs and Troubleshooting Flow

Application issue ဖြစ်တဲ့အခါ အောက်ပါ flow နဲ့ စစ်နိုင်ပါတယ်။

### Step 1: Pod status ကြည့်ရန်

```bash
kubectl get pods
```

### Step 2: Pod detail ကြည့်ရန်

```bash
kubectl describe pod <pod-name>
```

### Step 3: Pod logs ကြည့်ရန်

```bash
kubectl logs <pod-name>
```

### Step 4: Live logs ကြည့်ရန်

```bash
kubectl logs -f <pod-name>
```

### Step 5: Crash ဖြစ်ခဲ့ရင် previous logs ကြည့်ရန်

```bash
kubectl logs <pod-name> --previous
```

---

## 17. Docker Logs vs Kubernetes Logs

| Feature | Docker | Kubernetes |
|---|---|---|
| Normal logs command | `docker logs <container_id>` | `kubectl logs <pod-name>` |
| Live logs | `docker logs -f <container_id>` | `kubectl logs -f <pod-name>` |
| Multi-container handling | Container ID တစ်ခုစီ | Pod + container name လိုတတ် |
| Namespace support | မရှိ | `-n <namespace>` |
| Previous crashed container logs | Docker container state ပေါ်မူတည် | `--previous` သုံးနိုင် |

---

## 18. CKA Exam အတွက် မှတ်ရန်

CKA exam မှာ application logs ကြည့်တာက troubleshooting tasks တွေမှာ အလွန်အရေးကြီးပါတယ်။

မှတ်ထားရမယ့် commands -

```bash
kubectl logs <pod-name>
```

```bash
kubectl logs -f <pod-name>
```

```bash
kubectl logs <pod-name> -c <container-name>
```

```bash
kubectl logs <pod-name> --previous
```

```bash
kubectl logs <pod-name> -n <namespace>
```

Multi-container Pod တွေမှာ container name ထည့်ဖို့ မမေ့ရပါဘူး။

---

## 19. အသုံးများတဲ့ Commands

### Pod logs ကြည့်ရန်

```bash
kubectl logs event-simulator-pod
```

### Live logs ကြည့်ရန်

```bash
kubectl logs -f event-simulator-pod
```

### Multi-container pod logs ကြည့်ရန်

```bash
kubectl logs event-simulator-pod -c event-simulator
```

### Previous container logs ကြည့်ရန်

```bash
kubectl logs event-simulator-pod --previous
```

### Namespace ထဲက Pod logs ကြည့်ရန်

```bash
kubectl logs event-simulator-pod -n dev
```

### Last 20 lines only ကြည့်ရန်

```bash
kubectl logs event-simulator-pod --tail=20
```

### Last 10 minutes logs ကြည့်ရန်

```bash
kubectl logs event-simulator-pod --since=10m
```

### Deployment logs ကြည့်ရန်

```bash
kubectl logs deployment/<deployment-name>
```

### Label selector logs ကြည့်ရန်

```bash
kubectl logs -l app=nginx
```

---

## 20. Simple Summary

Docker container logs ကြည့်ရန် -

```bash
docker logs -f <container_id>
```

Kubernetes Pod logs ကြည့်ရန် -

```bash
kubectl logs -f <pod-name>
```

Multi-container Pod မှာ logs ကြည့်ရန် -

```bash
kubectl logs -f <pod-name> -c <container-name>
```

Crash ဖြစ်ခဲ့တဲ့ previous container logs ကြည့်ရန် -

```bash
kubectl logs <pod-name> --previous
```

အရေးကြီးဆုံး မှတ်ရန် -

```text
Single container pod  -> kubectl logs <pod-name>
Multi-container pod   -> kubectl logs <pod-name> -c <container-name>
Live logs             -> kubectl logs -f <pod-name>
Previous crash logs   -> kubectl logs <pod-name> --previous
```

Application troubleshooting အတွက် logs ကြည့်တတ်တာက CKA exam မှာလည်း practical skill တစ်ခုဖြစ်ပါတယ်။
