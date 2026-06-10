# Commands and Arguments in Kubernetes

> Topic: Kubernetes Pod ထဲမှာ `command` နဲ့ `args` အသုံးပြုပြီး container behavior ကို override လုပ်ခြင်း

---

## 1. Overview

Kubernetes Pod တစ်ခု run တဲ့အခါ container image ထဲမှာသတ်မှတ်ထားတဲ့ default command တွေကို အသုံးပြုပါတယ်။

Dockerfile ထဲမှာ usually —

- `ENTRYPOINT`
- `CMD`

ဆိုပြီး container start တဲ့အချိန် run မယ့် command နဲ့ parameter တွေကို သတ်မှတ်ထားနိုင်ပါတယ်။

Kubernetes မှာတော့ Pod YAML ထဲက —

- `command`
- `args`

ကိုသုံးပြီး Dockerfile ထဲက default behavior ကို override လုပ်နိုင်ပါတယ်။

---

## 2. Docker CMD and ENTRYPOINT Recap

ဥပမာ Dockerfile:

```dockerfile
FROM ubuntu
ENTRYPOINT ["sleep"]
CMD ["5"]
```

ဒီ Dockerfile မှာ —

| Dockerfile Instruction | Meaning                                 |
| ---------------------- | --------------------------------------- |
| `ENTRYPOINT ["sleep"]` | Container start တဲ့အခါ run မယ့် command |
| `CMD ["5"]`            | Default argument / parameter            |

Container run လိုက်ရင် default အနေနဲ့ —

```bash
sleep 5
```

ဖြစ်ပါတယ်။

---

## 3. Docker Runtime Argument

Docker မှာ argument override လုပ်ချင်ရင် image name နောက်မှာ argument ထည့်နိုင်ပါတယ်။

```bash
docker run --name ubuntu-sleeper ubuntu-sleeper
```

Result:

```bash
sleep 5
```

Argument ထည့်ပြီး run မယ်ဆိုရင် —

```bash
docker run --name ubuntu-sleeper ubuntu-sleeper 10
```

Result:

```bash
sleep 10
```

ဒီမှာ `10` က Dockerfile ထဲက `CMD ["5"]` ကို override လုပ်တာပါ။

---

## 4. Kubernetes မှာ args သုံးပြီး CMD Override လုပ်ခြင်း

Docker run command မှာ image name နောက်က argument ထည့်တာက Kubernetes Pod YAML မှာ `args` နဲ့တူပါတယ်။

ဥပမာ `ubuntu-sleeper` image က default `sleep 5` run တယ်ဆိုပါစို့။

Sleep duration ကို 10 seconds ပြောင်းချင်ရင် Pod YAML မှာ `args` ထည့်ပါ။

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper-pod
spec:
  containers:
    - name: ubuntu-sleeper
      image: ubuntu-sleeper
      args: ["10"]
```

ဒီ Pod run လိုက်ရင် —

```bash
sleep 10
```

ဖြစ်ပါတယ်။

### မှတ်ရန်

`args` က Dockerfile ထဲက `CMD` ကို override လုပ်ပါတယ်။

---

## 5. Kubernetes command သုံးပြီး ENTRYPOINT Override လုပ်ခြင်း

Dockerfile ထဲက `ENTRYPOINT` ကို override လုပ်ချင်ရင် Kubernetes Pod YAML မှာ `command` field ကိုသုံးပါတယ်။

Docker မှာဆိုရင် —

```bash
docker run --name ubuntu-sleeper   --entrypoint sleep2.0   ubuntu-sleeper 10
```

Kubernetes မှာ equivalent YAML က —

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper-pod
spec:
  containers:
    - name: ubuntu-sleeper
      image: ubuntu-sleeper
      command: ["sleep2.0"]
      args: ["10"]
```

ဒီမှာ —

- `command: ["sleep2.0"]` က Dockerfile `ENTRYPOINT ["sleep"]` ကို override လုပ်ပါတယ်။
- `args: ["10"]` က Dockerfile `CMD ["5"]` ကို override လုပ်ပါတယ်။

Result:

```bash
sleep2.0 10
```

> `sleep2.0` command က container image ထဲမှာ တကယ်ရှိမှ run လို့ရပါမယ်။

---

## 6. Dockerfile to Kubernetes Mapping

ဒီ table က CKA exam မှာ အရမ်းအရေးကြီးပါတယ်။

| Dockerfile   | Kubernetes Pod YAML | Purpose                                            |
| ------------ | ------------------- | -------------------------------------------------- |
| `ENTRYPOINT` | `command`           | Container start တဲ့အခါ run မယ့် command/executable |
| `CMD`        | `args`              | Command အတွက် default arguments/parameters         |

### Easy Memory

```text
Docker ENTRYPOINT = Kubernetes command
Docker CMD        = Kubernetes args
```

---

## 7. command vs args

| Field     | Meaning                      | Overrides               |
| --------- | ---------------------------- | ----------------------- |
| `command` | Main executable command      | Dockerfile `ENTRYPOINT` |
| `args`    | Parameters passed to command | Dockerfile `CMD`        |

### Important Point

- `command` ထည့်လိုက်ရင် Dockerfile ထဲက `ENTRYPOINT` ကို လုံးဝ replace လုပ်ပါတယ်။
- `args` ထည့်လိုက်ရင် Dockerfile ထဲက `CMD` ကို override လုပ်ပါတယ်။

---

## 8. Example 1 — Only args Override

Dockerfile:

```dockerfile
FROM ubuntu
ENTRYPOINT ["sleep"]
CMD ["5"]
```

Pod YAML:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper-pod
spec:
  containers:
    - name: ubuntu-sleeper
      image: ubuntu-sleeper
      args: ["10"]
```

Result:

```bash
sleep 10
```

ဒီမှာ `ENTRYPOINT` က unchanged ဖြစ်ပြီး `CMD` ပဲ override ဖြစ်ပါတယ်။

---

## 9. Example 2 — command and args Override

Dockerfile:

```dockerfile
FROM ubuntu
ENTRYPOINT ["sleep"]
CMD ["5"]
```

Pod YAML:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper-pod
spec:
  containers:
    - name: ubuntu-sleeper
      image: ubuntu-sleeper
      command: ["sleep"]
      args: ["3600"]
```

Result:

```bash
sleep 3600
```

---

## 10. Example 3 — Ubuntu Pod Sleep 3600

CKA exam မှာ Ubuntu pod ကို running state ဖြစ်နေအောင် `sleep 3600` ပေးခိုင်းတာများပါတယ်။

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sleeper
spec:
  containers:
    - name: sleeper
      image: ubuntu
      command: ["sleep"]
      args: ["3600"]
```

Create:

```bash
kubectl create -f pod-definition.yml
```

or

```bash
kubectl apply -f pod-definition.yml
```

Check:

```bash
kubectl get pods
```

---

## 11. Imperative Command with kubectl

YAML မရေးဘဲ command line နဲ့ Pod create လုပ်ချင်ရင် —

```bash
kubectl run sleeper --image=ubuntu -- sleep 3600
```

ဒီ command က Pod တစ်ခု create လုပ်ပြီး container ထဲမှာ `sleep 3600` run စေပါတယ်။

### YAML generate လုပ်ချင်ရင်

```bash
kubectl run sleeper --image=ubuntu --dry-run=client -o yaml -- sleep 3600
```

File ထဲသိမ်းရန်:

```bash
kubectl run sleeper --image=ubuntu --dry-run=client -o yaml -- sleep 3600 > pod.yaml
```

---

## 12. Pod Definition Apply

Pod YAML file အသင့်ဖြစ်ပြီဆိုရင် create လုပ်ရန် —

```bash
kubectl create -f pod-definition.yml
```

သို့မဟုတ် update/apply လုပ်ရန် —

```bash
kubectl apply -f pod-definition.yml
```

---

## 13. Check Pod Command and Args

Pod ထဲမှာ command / args မှန်မမှန်ကြည့်ရန် —

```bash
kubectl describe pod ubuntu-sleeper-pod
```

YAML output နဲ့ကြည့်ရန် —

```bash
kubectl get pod ubuntu-sleeper-pod -o yaml
```

Specific field ကြည့်ချင်ရင် —

```bash
kubectl get pod ubuntu-sleeper-pod -o jsonpath='{.spec.containers[0].command}'
```

```bash
kubectl get pod ubuntu-sleeper-pod -o jsonpath='{.spec.containers[0].args}'
```

---

## 14. Common Mistakes

### Mistake 1 — command နဲ့ args ကို string တစ်ကြောင်းတည်းရေးခြင်း

မမှန်:

```yaml
command: "sleep 3600"
```

မှန်:

```yaml
command: ["sleep"]
args: ["3600"]
```

---

### Mistake 2 — args ကို command အစားသုံးခြင်း

Image ထဲမှာ ENTRYPOINT မရှိရင် `args` တစ်ခုတည်းထည့်တာနဲ့ အလုပ်မလုပ်နိုင်ပါဘူး။

ဥပမာ Ubuntu image မှာ default command က `bash` ဖြစ်နိုင်ပါတယ်။  
ဒါကြောင့် sleep run စေချင်ရင် `command` ကိုပါထည့်ပါ။

```yaml
command: ["sleep"]
args: ["3600"]
```

---

### Mistake 3 — Container Exit ဖြစ်သွားခြင်း

Ubuntu Pod ကို command မပေးဘဲ run လိုက်ရင် Bash shell က terminal မရှိလို့ exit ဖြစ်နိုင်ပါတယ်။

မှန်တဲ့နည်း:

```yaml
command: ["sleep"]
args: ["3600"]
```

---

## 15. CKA Exam Tips

### Tip 1 — Mapping ကို အလွတ်မှတ်ပါ

```text
ENTRYPOINT = command
CMD        = args
```

---

### Tip 2 — Pod running ထားချင်ရင် sleep သုံးပါ

```bash
kubectl run sleeper --image=ubuntu -- sleep 3600
```

---

### Tip 3 — YAML generate လုပ်ပြီး edit လုပ်ပါ

```bash
kubectl run sleeper --image=ubuntu --dry-run=client -o yaml -- sleep 3600 > pod.yaml
vi pod.yaml
kubectl apply -f pod.yaml
```

---

### Tip 4 — Troubleshooting

Pod crash ဖြစ်နေရင် —

```bash
kubectl get pods
kubectl describe pod <pod-name>
kubectl logs <pod-name>
```

Command/args မှားနေတာဖြစ်နိုင်ပါတယ်။

---

## 16. Quick Reference

| Requirement              | YAML                                    |
| ------------------------ | --------------------------------------- |
| Override CMD only        | `args: ["10"]`                          |
| Override ENTRYPOINT only | `command: ["sleep"]`                    |
| Override both            | `command: ["sleep"]` + `args: ["10"]`   |
| Run Ubuntu sleep         | `command: ["sleep"]` + `args: ["3600"]` |

---

## 17. Practice Questions

### Question 1

Create a Pod named `ubuntu-sleeper` using image `ubuntu` and make it sleep for `4800` seconds.

Answer:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper
spec:
  containers:
    - name: ubuntu-sleeper
      image: ubuntu
      command: ["sleep"]
      args: ["4800"]
```

---

### Question 2

Dockerfile has:

```dockerfile
ENTRYPOINT ["sleep"]
CMD ["5"]
```

In Kubernetes, change only the sleep duration to `20`.

Answer:

```yaml
args: ["20"]
```

---

### Question 3

Override both command and argument to run `sleep 1000`.

Answer:

```yaml
command: ["sleep"]
args: ["1000"]
```

---

## 18. Key Takeaways

- Kubernetes Pod YAML မှာ `command` နဲ့ `args` ကိုသုံးပြီး container behavior ကို override လုပ်နိုင်ပါတယ်။
- Docker `ENTRYPOINT` က Kubernetes `command` နဲ့တူပါတယ်။
- Docker `CMD` က Kubernetes `args` နဲ့တူပါတယ်။
- `args` သုံးရင် Dockerfile CMD ကို override လုပ်ပါတယ်။
- `command` သုံးရင် Dockerfile ENTRYPOINT ကို replace လုပ်ပါတယ်။
- Ubuntu Pod ကို running ထားချင်ရင် `command: ["sleep"]` နဲ့ `args: ["3600"]` သုံးပါ။
- CKA exam မှာ `kubectl run ... -- sleep 3600` command က အရမ်းအသုံးဝင်ပါတယ်။
