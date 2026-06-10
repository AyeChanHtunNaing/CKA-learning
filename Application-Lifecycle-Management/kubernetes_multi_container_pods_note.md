# Multi Container Pods

> Topic: Kubernetes Pod တစ်ခုထဲမှာ container အများကြီး run ခြင်း

---

## 1. Overview

Kubernetes မှာ Pod တစ်ခုထဲမှာ container တစ်ခုထက်မက run လုပ်နိုင်ပါတယ်။  
ဒီလို Pod ကို **Multi Container Pod** လို့ခေါ်ပါတယ်။

Normally, application တစ်ခုကို microservices ပုံစံခွဲပြီး service တစ်ခုချင်းစီကို သီးခြား Pod / Deployment အဖြစ် run လုပ်တာက ပိုသင့်တော်ပါတယ်။

ဒါပေမယ့် တချို့ case တွေမှာ container ၂ ခု သို့မဟုတ် အများကြီးကို **အတူတူ run ဖြစ်ရမယ့် tightly coupled services** အဖြစ်ထားဖို့လိုပါတယ်။

ဥပမာ —

- Web application container
- Logging agent container

ဒီ ၂ ခုက အတူတူ run ရမယ်၊ အတူတူ stop ဖြစ်ရမယ်၊ log file တွေကို shared volume မှာအတူသုံးရမယ်ဆိုရင် Multi Container Pod သုံးနိုင်ပါတယ်။

---

## 2. Multi Container Pod ဆိုတာဘာလဲ?

Multi Container Pod ဆိုတာ Pod တစ်ခုထဲမှာ container အများကြီးထည့်ထားတာပါ။

Container တွေက —

- Same Pod ထဲမှာရှိတယ်
- Same lifecycle ကို share လုပ်တယ်
- Same network namespace ကို share လုပ်တယ်
- Same storage volumes တွေကို share လုပ်နိုင်တယ်

---

## 3. Why Use Multi Container Pods?

Application တစ်ခုနဲ့ helper service တစ်ခုဟာ အရမ်းနီးကပ်စွာတွဲပြီး run ရမယ်ဆိုရင် Multi Container Pod သုံးပါတယ်။

Use cases:

| Main Container | Helper Container    |
| -------------- | ------------------- |
| Web app        | Log agent           |
| App server     | Sidecar proxy       |
| App container  | Data sync container |
| Main app       | Monitoring agent    |
| Main app       | File processor      |

---

## 4. Main Benefit

Multi Container Pod ရဲ့ အဓိကအကျိုးကျေးဇူးတွေက —

1. Containers တွေကို same Pod ထဲမှာ group လုပ်နိုင်တယ်
2. Containers တွေက same lifecycle ရှိတယ်
3. `localhost` နဲ့ တစ်ခုနဲ့တစ်ခု communicate လုပ်နိုင်တယ်
4. Shared volume သုံးနိုင်တယ်
5. Networking / storage configuration ကို ပိုရိုးရှင်းစေတယ်

---

## 5. Shared Lifecycle

Multi Container Pod ထဲက container တွေက Pod lifecycle ကို share လုပ်ပါတယ်။

ဆိုလိုတာက —

- Pod create လုပ်ရင် containers တွေ create ဖြစ်မယ်
- Pod delete လုပ်ရင် containers တွေအားလုံး terminate ဖြစ်မယ်
- Pod တစ်ခုရဲ့ scheduling က containers အားလုံးအတွက်တစ်ခါတည်းဖြစ်မယ်

ဒါကြောင့် tightly coupled components တွေအတွက် အသုံးဝင်ပါတယ်။

---

## 6. Shared Network Namespace

Pod ထဲက containers အားလုံးက network namespace တစ်ခုတည်းကို share လုပ်ပါတယ်။

ဒါကြောင့် container တစ်ခုက container နောက်တစ်ခုကို `localhost` နဲ့ access လုပ်နိုင်ပါတယ်။

ဥပမာ —

- app container listens on port `8080`
- sidecar proxy container can access it through `localhost:8080`

```text
Container A -> localhost:8080 -> Container B
```

Important:

Pod တစ်ခုထဲက containers တွေက same IP address ကို share လုပ်ပါတယ်။

---

## 7. Shared Storage Volumes

Pod ထဲက containers တွေက same volume ကို mount လုပ်နိုင်ပါတယ်။

ဥပမာ —

- web app container က log file ကို `/var/log/app.log` မှာရေးတယ်
- log-agent container က same volume ကို mount လုပ်ပြီး log file ကို read/send လုပ်တယ်

ဒါကြောင့် log shipping / sidecar pattern တွေမှာအသုံးများပါတယ်။

---

## 8. Example Diagram

![Multi Container Pod](https://kodekloud.com/kk-media/image/upload/v1752869666/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-Multi-Container-Pods/frame_90.jpg)

ဒီ diagram မှာ Multi Container Pod က lifecycle, network, storage ကို share လုပ်တာကိုပြထားပါတယ်။

---

## 9. Basic YAML Example

Pod တစ်ခုထဲမှာ container ၂ ခုထည့်ထားတဲ့ example —

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
  labels:
    name: simple-webapp
spec:
  containers:
    - name: simple-webapp
      image: simple-webapp
      ports:
        - containerPort: 8080

    - name: log-agent
      image: log-agent
```

ဒီ Pod ထဲမှာ —

- `simple-webapp` container
- `log-agent` container

ဆိုပြီး container ၂ ခုပါပါတယ်။

---

## 10. Important Point — containers is an Array

Pod YAML ထဲမှာ `containers` field က array ဖြစ်ပါတယ်။

```yaml
containers:
  - name: container-one
    image: image-one

  - name: container-two
    image: image-two
```

Container အသစ်ထည့်ချင်ရင် `containers` list ထဲမှာ item အသစ်ထည့်ရုံပါပဲ။

---

## 11. Multi Container Pod with Shared Volume

Logging agent use case ကို shared volume နဲ့ရေးမယ်ဆိုရင် —

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp-with-logger
spec:
  containers:
    - name: webapp
      image: busybox
      command: ["sh", "-c"]
      args:
        - while true; do echo "$(date) web request" >> /var/log/app/app.log; sleep 5; done
      volumeMounts:
        - name: log-volume
          mountPath: /var/log/app

    - name: log-agent
      image: busybox
      command: ["sh", "-c"]
      args:
        - tail -f /var/log/app/app.log
      volumeMounts:
        - name: log-volume
          mountPath: /var/log/app

  volumes:
    - name: log-volume
      emptyDir: {}
```

ဒီ example မှာ —

- `webapp` container က log file ရေးတယ်
- `log-agent` container က same log file ကို read လုပ်တယ်
- `emptyDir` volume ကို container နှစ်ခုလုံး share လုပ်တယ်

---

## 12. Apply Pod YAML

```bash
kubectl apply -f multi-container-pod.yaml
```

Check Pod:

```bash
kubectl get pods
```

Describe:

```bash
kubectl describe pod webapp-with-logger
```

---

## 13. Logs for Multi Container Pod

Pod ထဲမှာ container တစ်ခုထက်မကရှိရင် `kubectl logs` မှာ container name specify လုပ်ရပါတယ်။

```bash
kubectl logs webapp-with-logger -c webapp
```

Log agent container logs ကြည့်ရန်:

```bash
kubectl logs webapp-with-logger -c log-agent
```

Follow logs:

```bash
kubectl logs -f webapp-with-logger -c log-agent
```

---

## 14. Exec into Specific Container

Multi container pod ထဲက specific container ထဲဝင်ချင်ရင် `-c` သုံးပါ။

```bash
kubectl exec -it webapp-with-logger -c webapp -- sh
```

Log agent container ထဲဝင်ရန်:

```bash
kubectl exec -it webapp-with-logger -c log-agent -- sh
```

---

## 15. Common Multi Container Pod Patterns

Multi container pod မှာ commonly သုံးတဲ့ patterns တွေရှိပါတယ်။

| Pattern    | Meaning                                                              |
| ---------- | -------------------------------------------------------------------- |
| Sidecar    | Main app ကို support လုပ်တဲ့ helper container                        |
| Adapter    | Output format ကို transform လုပ်တဲ့ container                        |
| Ambassador | Network proxy / external service access ကို handle လုပ်တဲ့ container |

---

## 16. Sidecar Pattern

Sidecar container က main application ကို support လုပ်တဲ့ helper container ဖြစ်ပါတယ်။

Examples:

- log collector
- proxy
- config reloader
- monitoring agent
- file syncer

ဥပမာ web app + log agent ဆိုတာ sidecar pattern ပါ။

```text
Main container: webapp
Sidecar container: log-agent
```

---

## 17. Adapter Pattern

Adapter container က main app output ကို standard format ပြောင်းပေးတဲ့ container ဖြစ်ပါတယ်။

ဥပမာ —

- app က custom log format ထုတ်တယ်
- adapter container က log format ကို monitoring system နားလည်တဲ့ format ပြောင်းပေးတယ်

---

## 18. Ambassador Pattern

Ambassador container က application ရဲ့ network communication ကို proxy အဖြစ်လုပ်ပေးပါတယ်။

ဥပမာ —

- app container က localhost proxy ကိုခေါ်တယ်
- ambassador container က external database/service ကို connect လုပ်ပေးတယ်

---

## 19. When Should You Use Multi Container Pod?

Multi Container Pod သုံးသင့်တဲ့အခြေအနေ —

- Containers တွေက tightly coupled ဖြစ်တယ်
- Same lifecycle လိုတယ်
- Same network namespace လိုတယ်
- Same volume share လုပ်ဖို့လိုတယ်
- Helper container က main app နဲ့အမြဲတွဲ run ရမယ်

---

## 20. When Should You Not Use Multi Container Pod?

ဒီလို case တွေမှာ Multi Container Pod မသုံးသင့်ပါဘူး။

- Services တွေကို သီးခြား scale လုပ်ချင်တယ်
- Independent deployment လုပ်ချင်တယ်
- Different lifecycle လိုတယ်
- Different resource requirements အရမ်းကွာတယ်
- Microservice တစ်ခုချင်းစီကို independent manage လုပ်ချင်တယ်

ဥပမာ app server နဲ့ database ကို Pod တစ်ခုထဲမှာထားတာ မသင့်တော်ပါဘူး။  
Database ကို သီးခြား StatefulSet / Deployment နဲ့ run သင့်ပါတယ်။

---

## 21. CKA Exam Tips

### Tip 1 — containers field က array ဖြစ်တာမှတ်ပါ

```yaml
containers:
  - name: app
    image: app-image
  - name: sidecar
    image: sidecar-image
```

---

### Tip 2 — logs ကြည့်ရင် `-c` ထည့်ပါ

```bash
kubectl logs <pod-name> -c <container-name>
```

---

### Tip 3 — exec ဝင်ရင်လည်း `-c` ထည့်ပါ

```bash
kubectl exec -it <pod-name> -c <container-name> -- sh
```

---

### Tip 4 — shared volume use case ကိုမှတ်ပါ

```yaml
volumes:
  - name: shared-data
    emptyDir: {}
```

```yaml
volumeMounts:
  - name: shared-data
    mountPath: /data
```

---

### Tip 5 — Pod ထဲ container အရေအတွက်ကြည့်ရန်

```bash
kubectl get pod <pod-name> -o jsonpath='{.spec.containers[*].name}'
```

---

## 22. Useful Commands

| Task                         | Command                                                               |
| ---------------------------- | --------------------------------------------------------------------- |
| Apply YAML                   | `kubectl apply -f multi-container-pod.yaml`                           |
| Get pods                     | `kubectl get pods`                                                    |
| Describe pod                 | `kubectl describe pod <pod-name>`                                     |
| Logs from specific container | `kubectl logs <pod-name> -c <container-name>`                         |
| Follow logs                  | `kubectl logs -f <pod-name> -c <container-name>`                      |
| Exec into container          | `kubectl exec -it <pod-name> -c <container-name> -- sh`               |
| Get container names          | `kubectl get pod <pod-name> -o jsonpath='{.spec.containers[*].name}'` |

---

## 23. Common Mistakes

### Mistake 1 — containers ကို object တစ်ခုလိုရေးခြင်း

မမှန်:

```yaml
containers:
  name: app
  image: nginx
```

မှန်:

```yaml
containers:
  - name: app
    image: nginx
```

---

### Mistake 2 — Multi container pod logs ကြည့်ရာမှာ `-c` မထည့်ခြင်း

Pod ထဲမှာ containers အများကြီးရှိရင် —

```bash
kubectl logs simple-webapp
```

ဒီလို run တာက error ဖြစ်နိုင်ပါတယ်။

မှန်:

```bash
kubectl logs simple-webapp -c simple-webapp
```

---

### Mistake 3 — Independent services တွေကို Pod တစ်ခုထဲထားခြင်း

App နဲ့ DB ကို Pod တစ်ခုထဲမထားသင့်ပါဘူး။  
သူတို့ကို သီးခြား Pod/Deployment/StatefulSet အဖြစ်ထားတာပိုကောင်းပါတယ်။

---

## 24. Practice Example

### Task

Pod တစ်ခု create လုပ်ပါ။

Requirements:

- Pod name: `simple-webapp`
- Container 1:
  - name: `simple-webapp`
  - image: `simple-webapp`
  - port: `8080`
- Container 2:
  - name: `log-agent`
  - image: `log-agent`

### Solution

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
  labels:
    name: simple-webapp
spec:
  containers:
    - name: simple-webapp
      image: simple-webapp
      ports:
        - containerPort: 8080

    - name: log-agent
      image: log-agent
```

Apply:

```bash
kubectl apply -f pod.yaml
```

Check:

```bash
kubectl get pods
kubectl describe pod simple-webapp
```

---

## 25. Key Takeaways

- Multi Container Pod ဆိုတာ Pod တစ်ခုထဲမှာ container တစ်ခုထက်မက run လုပ်တာပါ။
- Containers တွေက same lifecycle, network namespace, storage volumes ကို share လုပ်ပါတယ်။
- Same Pod ထဲက containers တွေက `localhost` နဲ့ communicate လုပ်နိုင်ပါတယ်။
- Shared volume သုံးပြီး data/log files တွေ share လုပ်နိုင်ပါတယ်။
- `containers` field က array ဖြစ်ပါတယ်။
- Multi container pod logs ကြည့်ရင် `kubectl logs <pod> -c <container>` သုံးပါ။
- Sidecar pattern က CKA မှာအရေးကြီးတဲ့ concept ပါ။
