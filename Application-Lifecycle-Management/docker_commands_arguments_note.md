# Commands and Arguments in Docker

> Topic: Docker `CMD`, `ENTRYPOINT`, command arguments, and container process behavior

---

## 1. Overview

Docker container တစ်ခု run တဲ့အခါ container ထဲမှာ **process တစ်ခု** run ပါတယ်။  
Container ရဲ့ lifecycle က အဲဒီ process နဲ့ တိုက်ရိုက်ဆက်စပ်နေပါတယ်။

ဆိုလိုတာက —

- Process run နေသေးရင် container လည်း running ဖြစ်နေမယ်
- Process ပြီးသွားရင် container လည်း stop/exited ဖြစ်သွားမယ်

ဒါကြောင့် container ကို VM လိုမျိုး always running machine အဖြစ်မမြင်သင့်ပါဘူး။  
Container က specific task/process တစ်ခု run ဖို့အတွက် optimized လုပ်ထားတာပါ။

---

## 2. Ubuntu Container ဘာကြောင့်ချက်ချင်း Exit ဖြစ်သွားလဲ?

Ubuntu image ကို run ကြည့်ပါ။

```bash
docker run ubuntu
```

ပြီးရင် running containers တွေကိုကြည့်ပါ။

```bash
docker ps
```

Container ကို မတွေ့ရနိုင်ပါဘူး။ ဘာကြောင့်လဲဆိုတော့ Ubuntu container က start လုပ်ပြီး ချက်ချင်း exit ဖြစ်သွားလို့ပါ။

Stopped container တွေပါကြည့်ချင်ရင် —

```bash
docker ps -a
```

Output example:

```text
CONTAINER ID   IMAGE    COMMAND       STATUS
45aacca36850   ubuntu   "/bin/bash"   Exited (0)
```

Ubuntu image ရဲ့ default command က `/bin/bash` ဖြစ်ပါတယ်။  
ဒါပေမယ့် attached terminal မရှိတဲ့အခါ Bash shell က interactive session မရလို့ ချက်ချင်း exit ဖြစ်သွားပါတယ်။

---

## 3. Container Process Concept

Container တွေက usually single process run ဖို့ design လုပ်ထားတာပါ။

ဥပမာ —

| Container Type       | Main Process      |
| -------------------- | ----------------- |
| Nginx container      | `nginx`           |
| MySQL container      | `mysqld`          |
| Redis container      | `redis-server`    |
| Ubuntu container     | `bash`            |
| Custom job container | script or command |

Main process stop ဖြစ်သွားရင် container လည်း stop ဖြစ်သွားပါတယ်။

---

## 4. Default Command in Docker Image

Docker image တစ်ခုမှာ container start လုပ်တဲ့အခါ run မယ့် default command ကို Dockerfile ထဲမှာ `CMD` နဲ့သတ်မှတ်နိုင်ပါတယ်။

Example — Nginx image:

```dockerfile
CMD ["nginx"]
```

Example — Ubuntu image:

```dockerfile
CMD ["bash"]
```

`CMD` က container start ဖြစ်တဲ့အချိန် default run မယ့် command ကိုသတ်မှတ်တာပါ။

---

## 5. Override Default Command

Docker image ထဲမှာ default command ရှိပေမယ့် `docker run` command နောက်မှာ command ထည့်ပြီး override လုပ်နိုင်ပါတယ်။

```bash
docker run ubuntu sleep 5
```

ဒီ command မှာ Ubuntu image ရဲ့ default `bash` ကို override လုပ်ပြီး `sleep 5` ကို run ပါမယ်။

Container က —

1. `sleep 5` ကို run မယ်
2. 5 seconds စောင့်မယ်
3. Process ပြီးသွားရင် container exit ဖြစ်မယ်

---

## 6. CMD in Dockerfile

Container ကို default အနေနဲ့ `sleep 5` run စေချင်ရင် Dockerfile ထဲမှာ `CMD` ထည့်နိုင်ပါတယ်။

### Shell form

```dockerfile
FROM ubuntu
CMD sleep 5
```

### JSON array form

```dockerfile
FROM ubuntu
CMD ["sleep", "5"]
```

CKA / Docker best practice အနေနဲ့ JSON array form ကို ပို recommend လုပ်ပါတယ်။

Build image:

```bash
docker build -t ubuntu-sleeper .
```

Run container:

```bash
docker run ubuntu-sleeper
```

ဒီ container က default အနေနဲ့ `sleep 5` run ပါမယ်။

---

## 7. CMD ရဲ့ Limitation

`CMD` သုံးထားတဲ့ image မှာ runtime argument ထည့်လိုက်ရင် default command တစ်ခုလုံး replace ဖြစ်သွားနိုင်ပါတယ်။

ဥပမာ Dockerfile:

```dockerfile
FROM ubuntu
CMD ["sleep", "5"]
```

Run:

```bash
docker run ubuntu-sleeper 10
```

ဒီမှာ `10` က `sleep 5` ရဲ့ argument အဖြစ် append မဖြစ်ပါဘူး။  
Instead, command ကို `10` နဲ့ replace လုပ်ဖို့ကြိုးစားနိုင်ပြီး error ဖြစ်နိုင်ပါတယ်။

ဒါကြောင့် command executable ကို fixed ထားပြီး parameter ပဲ runtime မှာပြောင်းချင်ရင် `ENTRYPOINT` သုံးရပါတယ်။

---

## 8. ENTRYPOINT

`ENTRYPOINT` က container start ဖြစ်တဲ့အခါ run မယ့် executable ကို fixed လုပ်ပေးပါတယ်။

Runtime မှာ ထည့်တဲ့ arguments တွေက `ENTRYPOINT` နောက်မှာ append ဖြစ်ပါတယ်။

Example:

```dockerfile
FROM ubuntu
ENTRYPOINT ["sleep"]
```

Build:

```bash
docker build -t ubuntu-sleeper .
```

Run with argument:

```bash
docker run ubuntu-sleeper 10
```

Result:

```bash
sleep 10
```

ဒီမှာ `10` က `sleep` ရဲ့ argument အဖြစ် append ဖြစ်ပါတယ်။

---

## 9. ENTRYPOINT + CMD Together

`ENTRYPOINT` နဲ့ `CMD` ကို အတူသုံးရင် —

- `ENTRYPOINT` = executable
- `CMD` = default argument

Example:

```dockerfile
FROM ubuntu
ENTRYPOINT ["sleep"]
CMD ["5"]
```

Run without argument:

```bash
docker run ubuntu-sleeper
```

Result:

```bash
sleep 5
```

Run with argument:

```bash
docker run ubuntu-sleeper 10
```

Result:

```bash
sleep 10
```

ဒီ pattern က runtime မှာ parameter ပြောင်းချင်တဲ့ container တွေအတွက် အသုံးများပါတယ်။

---

## 10. CMD vs ENTRYPOINT

| Feature                   | CMD                                  | ENTRYPOINT              |
| ------------------------- | ------------------------------------ | ----------------------- |
| Purpose                   | Default command or default arguments | Fixed executable        |
| Runtime argument behavior | Usually replaces CMD                 | Appends to ENTRYPOINT   |
| Best use case             | Default behavior သတ်မှတ်ရန်          | Executable fixed ထားရန် |
| Example                   | `CMD ["sleep", "5"]`                 | `ENTRYPOINT ["sleep"]`  |

---

## 11. ENTRYPOINT + CMD Comparison

| Dockerfile                           | `docker run image`     | `docker run image 10`    |
| ------------------------------------ | ---------------------- | ------------------------ |
| `CMD ["sleep", "5"]`                 | `sleep 5`              | command replaced by `10` |
| `ENTRYPOINT ["sleep"]`               | error: missing operand | `sleep 10`               |
| `ENTRYPOINT ["sleep"]` + `CMD ["5"]` | `sleep 5`              | `sleep 10`               |

---

## 12. Override ENTRYPOINT at Runtime

တစ်ခါတစ်လေ image ထဲက `ENTRYPOINT` ကိုလုံးဝ override လုပ်ချင်ရင် `--entrypoint` flag သုံးနိုင်ပါတယ်။

Example Dockerfile:

```dockerfile
FROM ubuntu
ENTRYPOINT ["sleep"]
CMD ["5"]
```

Override ENTRYPOINT:

```bash
docker run --entrypoint sleep2.0 ubuntu-sleeper 10
```

Result:

```bash
sleep2.0 10
```

> Note: `sleep2.0` ဆိုတဲ့ command က container ထဲမှာ တကယ်ရှိမှ run လို့ရပါမယ်။

---

## 13. Important Docker Commands

| Task                       | Command                                              |
| -------------------------- | ---------------------------------------------------- |
| Run Ubuntu container       | `docker run ubuntu`                                  |
| Show running containers    | `docker ps`                                          |
| Show all containers        | `docker ps -a`                                       |
| Run container with command | `docker run ubuntu sleep 5`                          |
| Build image                | `docker build -t ubuntu-sleeper .`                   |
| Run custom image           | `docker run ubuntu-sleeper`                          |
| Pass runtime argument      | `docker run ubuntu-sleeper 10`                       |
| Override entrypoint        | `docker run --entrypoint sleep2.0 ubuntu-sleeper 10` |

---

## 14. Kubernetes Connection

Docker `CMD` နဲ့ `ENTRYPOINT` concept က Kubernetes Pod definition မှာလည်း ဆက်သုံးပါတယ်။

Kubernetes မှာ —

| Docker       | Kubernetes |
| ------------ | ---------- |
| `ENTRYPOINT` | `command`  |
| `CMD`        | `args`     |

Example Dockerfile:

```dockerfile
ENTRYPOINT ["sleep"]
CMD ["5"]
```

Kubernetes Pod equivalent:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper
spec:
  containers:
    - name: ubuntu-sleeper
      image: ubuntu-sleeper
      command: ["sleep"]
      args: ["5"]
```

Runtime argument ပြောင်းချင်ရင် `args` ကိုပြောင်းနိုင်ပါတယ်။

```yaml
args: ["10"]
```

---

## 15. CKA Exam Tips

### Tip 1 — Docker CMD / ENTRYPOINT ကို Kubernetes command / args နဲ့ချိတ်မှတ်ပါ

```text
Docker ENTRYPOINT = Kubernetes command
Docker CMD        = Kubernetes args
```

---

### Tip 2 — Pod တစ်ခု sleep command run ခိုင်းချင်ရင်

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

---

### Tip 3 — Imperative command နဲ့ Pod create

```bash
kubectl run sleeper --image=ubuntu -- sleep 3600
```

---

### Tip 4 — Pod command စစ်ရန်

```bash
kubectl describe pod sleeper
```

or

```bash
kubectl get pod sleeper -o yaml
```

---

## 16. Practice Example

### Task

Ubuntu image ကိုသုံးပြီး `sleep 3600` run မယ့် Pod တစ်ခု create လုပ်ပါ။

### Solution YAML

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
      args: ["3600"]
```

Apply:

```bash
kubectl apply -f pod.yaml
```

Check:

```bash
kubectl get pods
kubectl describe pod ubuntu-sleeper
```

---

## 17. Key Takeaways

- Container lifecycle က main process နဲ့ချိတ်ဆက်နေပါတယ်။
- Main process stop ဖြစ်သွားရင် container လည်း stop ဖြစ်ပါတယ်။
- Ubuntu container က default `bash` command run တာဖြစ်ပြီး terminal မရှိရင် ချက်ချင်း exit ဖြစ်နိုင်ပါတယ်။
- `CMD` က default command or default arguments ကိုသတ်မှတ်ပါတယ်။
- `ENTRYPOINT` က executable ကို fixed လုပ်ပါတယ်။
- Runtime arguments တွေက `ENTRYPOINT` နောက်မှာ append ဖြစ်ပါတယ်။
- `ENTRYPOINT ["sleep"]` + `CMD ["5"]` ဆိုရင် default `sleep 5` ဖြစ်ပါတယ်။
- `docker run image 10` ဆိုရင် `sleep 10` ဖြစ်ပါတယ်။
- Kubernetes မှာ Docker `ENTRYPOINT` က `command` နဲ့တူပြီး Docker `CMD` က `args` နဲ့တူပါတယ်။
