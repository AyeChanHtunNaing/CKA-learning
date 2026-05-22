# Kubernetes Imperative vs Declarative Note

> This note is based on the KodeKloud lesson about **Imperative vs Declarative** Kubernetes object management.

---

## 1. Imperative နဲ့ Declarative ဆိုတာဘာလဲ?

Kubernetes object တွေကို create, update, delete လုပ်တဲ့အခါ နည်းလမ်း ၂ မျိုးရှိပါတယ်။

```text
1. Imperative Approach
2. Declarative Approach
```

အလွယ်မှတ်ရန် -

```text
Imperative = ဘယ်လိုလုပ်ရမလဲဆိုတာ step-by-step command နဲ့ပြောတာ
Declarative = ဘာလိုချင်လဲဆိုတဲ့ desired state ကို YAML ထဲမှာရေးတာ
```

---

## 2. Analogy နဲ့နားလည်ခြင်း

တစ်ယောက်ရဲ့အိမ်ကိုသွားမယ်ဆိုပါစို့။

### Imperative Approach

Taxi driver ကို လမ်းညွှန်တဲ့အခါ -

```text
Street B ကို ညာကွေ့
Street C ကို ဘယ်ကွေ့
Street D ကို ဘယ်ကွေ့
ပြီးရင် အိမ်ရှေ့မှာ ရပ်
```

ဒီနည်းက **step-by-step instruction** ပေးတာဖြစ်ပါတယ်။

### Declarative Approach

Uber / Grab app ထဲမှာ destination ကိုထည့်လိုက်တာနဲ့တူပါတယ်။

```text
Tom's house ကိုသွားမယ်
```

ဘယ်လမ်းကသွားမလဲဆိုတာ system ကဆုံးဖြတ်ပါတယ်။ User က final result ကိုပဲပြောတာပါ။

---

# Part 1: Imperative Approach

## 3. Imperative Approach ဆိုတာဘာလဲ?

**Imperative Approach** ဆိုတာ Kubernetes ကို command တစ်ကြောင်းချင်းစီနဲ့ ဘာလုပ်ရမလဲဆိုတာ တိုက်ရိုက်ပြောတဲ့နည်းလမ်းဖြစ်ပါတယ်။

ဥပမာ -

```bash
kubectl run nginx --image=nginx
```

ဒီ command က Kubernetes ကို -

```text
nginx image သုံးပြီး Pod တစ်ခု create လုပ်ပါ
```

လို့ တိုက်ရိုက်ခိုင်းတာပါ။

---

## 4. Imperative Commands Examples

```bash
kubectl run --image=nginx nginx
kubectl create deployment nginx --image=nginx
kubectl expose deployment nginx --port 80
kubectl edit deployment nginx
kubectl scale deployment nginx --replicas=5
kubectl set image deployment nginx nginx=nginx:1.18
kubectl create -f nginx.yaml
kubectl replace -f nginx.yaml
kubectl delete -f nginx.yaml
```

ဒီ command တွေက resource ကို ချက်ချင်း create, update, delete လုပ်ဖို့ အသုံးဝင်ပါတယ်။

---

## 5. Imperative Approach ကိုဘယ်အချိန်သုံးသင့်လဲ?

Imperative command တွေက **quick task** တွေအတွက်အသုံးဝင်ပါတယ်။

ဥပမာ -

```text
Pod တစ်ခုအမြန် create လုပ်ချင်တယ်
Deployment တစ်ခုအမြန် create လုပ်ချင်တယ်
Exam ထဲမှာ time save လုပ်ချင်တယ်
Simple object တစ်ခုကို command နဲ့ create လုပ်ချင်တယ်
```

CKA exam မှာ speed အတွက် imperative command တွေကိုသုံးတာ အဆင်ပြေပါတယ်။

---

## 6. Imperative Approach ရဲ့ အားသာချက်

| Advantage             | Meaning                                          |
| --------------------- | ------------------------------------------------ |
| Fast                  | Command တစ်ကြောင်းနဲ့ create/update လုပ်နိုင်တယ် |
| Easy for simple tasks | Simple Pod/Deployment တွေအတွက် လွယ်တယ်           |
| Good for exams        | CKA မှာ အချိန်သက်သာတယ်                           |
| Direct action         | Kubernetes ကို တိုက်ရိုက်ခိုင်းတာဖြစ်တယ်         |

---

## 7. Imperative Approach ရဲ့ အားနည်းချက်

Imperative command တွေမှာ limitation တွေရှိပါတယ်။

```text
Command run ပြီးသွားရင် history/definition မကျန်နိုင်ဘူး
Team members တွေအတွက် original desired state ကို trace လုပ်ရခက်နိုင်တယ်
Command partial success/failure ဖြစ်ရင် resource already exists ဆိုတဲ့ problem ဖြစ်နိုင်တယ်
Live object ကို edit လုပ်ထားတာ YAML file ထဲမှာမပါနိုင်ဘူး
```

ဥပမာ `kubectl edit deployment nginx` နဲ့ live object ကိုပြင်လိုက်ပြီး local YAML file ကိုမပြင်ထားရင် နောက်တစ်ခါ YAML file apply လုပ်တဲ့အခါ live changes တွေ overwrite ဖြစ်နိုင်ပါတယ်။

---

# Part 2: Declarative Approach

## 8. Declarative Approach ဆိုတာဘာလဲ?

**Declarative Approach** ဆိုတာ Kubernetes object ရဲ့ desired state ကို YAML file ထဲမှာရေးပြီး Kubernetes ကို apply လုပ်ခိုင်းတဲ့နည်းလမ်းဖြစ်ပါတယ်။

ဥပမာ `nginx.yaml` file -

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
    type: front-end
spec:
  containers:
    - name: nginx-container
      image: nginx
```

Apply လုပ်ရန် -

```bash
kubectl apply -f nginx.yaml
```

ဒီမှာ user က -

```text
ငါလိုချင်တာ ဒီ YAML ထဲက state ဖြစ်တယ်
```

လို့ပြောတာပါ။ Kubernetes က လက်ရှိ state နဲ့ YAML ထဲက desired state ကိုနှိုင်းယှဉ်ပြီး လိုအပ်တာကို create/update လုပ်ပေးပါတယ်။

---

## 9. Declarative Approach ကိုဘယ်အချိန်သုံးသင့်လဲ?

Declarative approach ကို production/team environment မှာ ပိုသင့်တော်ပါတယ်။

```text
Long-term management လုပ်ချင်တယ်
YAML file တွေကို Git မှာ version control ထားချင်တယ်
Team members တွေနဲ့ resource definition ကို share လုပ်ချင်တယ်
Change history ကို track လုပ်ချင်တယ်
Complex configuration တွေ manage လုပ်ချင်တယ်
```

---

## 10. Declarative Approach ရဲ့ အားသာချက်

| Advantage       | Meaning                                                     |
| --------------- | ----------------------------------------------------------- |
| Source of truth | YAML file က desired state ဖြစ်တယ်                           |
| Version control | Git ထဲမှာ changes တွေ track လုပ်နိုင်တယ်                    |
| Team-friendly   | Team members တွေ resource definition ကိုမြင်နိုင်တယ်        |
| Repeatable      | Same YAML ကို environment အမျိုးမျိုးမှာ reuse လုပ်နိုင်တယ် |
| Safer updates   | Config file ပြင်ပြီး apply လုပ်နိုင်တယ်                     |

---

## 11. Declarative Commands Examples

```bash
kubectl apply -f nginx.yaml
kubectl apply -f /path/to/config-files
kubectl replace -f nginx.yaml
kubectl delete -f nginx.yaml
```

`kubectl apply` က declarative style မှာ အရေးကြီးဆုံး command တစ်ခုပါ။

---

# Part 3: Imperative vs Declarative Comparison

## 12. Difference Table

| Topic              | Imperative                        | Declarative                   |
| ------------------ | --------------------------------- | ----------------------------- |
| Main idea          | ဘယ်လိုလုပ်ရမလဲ command နဲ့ပြော    | ဘာလိုချင်လဲ YAML နဲ့ပြော      |
| Usage              | Quick tasks                       | Long-term management          |
| File required      | မလိုနိုင်                         | YAML file လိုတယ်              |
| Version control    | ခက်နိုင်                          | လွယ်တယ်                       |
| Team collaboration | နည်းနည်းခက်                       | ကောင်းတယ်                     |
| Exam speed         | မြန်                              | Config အတွက်ကောင်း            |
| Example            | `kubectl run nginx --image=nginx` | `kubectl apply -f nginx.yaml` |

---

## 13. Create / Update / Delete Comparison

### Imperative

```bash
kubectl run nginx --image=nginx
kubectl scale deployment nginx --replicas=5
kubectl delete pod nginx
```

### Declarative

```bash
kubectl apply -f nginx.yaml
kubectl replace -f nginx.yaml
kubectl delete -f nginx.yaml
```

---

## 14. Update Dilemma

Imperative နဲ့ live object ကိုပြင်တဲ့အခါ သတိထားရမယ့်အချက်ရှိပါတယ်။

ဥပမာ -

```bash
kubectl edit deployment nginx
```

ဒီ command က live object ကိုတိုက်ရိုက်ပြင်တာပါ။ ဒါပေမယ့် local YAML file ကိုမပြင်ထားဘူးဆိုရင် source of truth က မကိုက်တော့ပါဘူး။

နောက်တစ်ခါ -

```bash
kubectl apply -f nginx.yaml
```

လုပ်တဲ့အခါ live object ထဲက manual changes တွေ overwrite ဖြစ်နိုင်ပါတယ်။

အကောင်းဆုံး practice -

```text
YAML file ကိုအရင်ပြင်
ပြီးရင် kubectl apply / replace လုပ်
```

---

## 15. Recommended Workflow

Team environment မှာ အောက်ပါ workflow ကိုသုံးတာကောင်းပါတယ်။

```text
1. YAML file ရေး
2. kubectl create -f nginx.yaml သို့မဟုတ် kubectl apply -f nginx.yaml လုပ်
3. Change လုပ်ချင်ရင် local YAML file ကိုပြင်
4. kubectl apply -f nginx.yaml သို့မဟုတ် kubectl replace -f nginx.yaml လုပ်
5. YAML file ကို Git ထဲမှာ version control ထား
```

Example -

```bash
kubectl create -f nginx.yaml
```

Image version ပြောင်းချင်ရင် YAML file ထဲမှာ image ကိုပြင်ပါ။

```yaml
image: nginx:1.18
```

ပြီးရင် -

```bash
kubectl replace -f nginx.yaml
```

သို့မဟုတ် -

```bash
kubectl apply -f nginx.yaml
```

---

## 16. CKA Exam Tips

CKA exam မှာ အချိန်အရေးကြီးတဲ့အတွက် command နဲ့ YAML နှစ်မျိုးလုံးသုံးတတ်ဖို့လိုပါတယ်။

```text
Simple object create လုပ်ချင်ရင် imperative command သုံးပါ။
Complex configuration လိုရင် YAML file ရေးပြီး apply လုပ်ပါ။
kubectl run / create deployment / expose / scale ကိုလေ့ကျင့်ပါ။
YAML generate လုပ်ဖို့ --dry-run=client -o yaml ကိုသုံးပါ။
Live edit လုပ်ရင် local YAML file နဲ့မကိုက်တော့နိုင်တာ သတိထားပါ။
```

Useful command -

```bash
kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml
```

Deployment YAML generate -

```bash
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > deployment.yaml
```

Service expose command -

```bash
kubectl expose deployment nginx --port=80 --target-port=80 --dry-run=client -o yaml > service.yaml
```

---

## 17. Quick Memory

```text
Imperative = Do this now
Declarative = Make the cluster look like this YAML
```

```text
Imperative command:
kubectl run nginx --image=nginx

Declarative command:
kubectl apply -f nginx.yaml
```

---

## 18. Final Summary

Imperative approach ဆိုတာ Kubernetes ကို command တစ်ကြောင်းချင်းစီနဲ့ ဘာလုပ်ရမလဲဆိုတာ တိုက်ရိုက်ခိုင်းတဲ့နည်းလမ်းဖြစ်ပါတယ်။ Simple task တွေအတွက်မြန်ပြီး CKA exam မှာအသုံးဝင်ပါတယ်။

Declarative approach ကတော့ YAML file ထဲမှာ desired state ကိုရေးပြီး Kubernetes ကို apply လုပ်ခိုင်းတာဖြစ်ပါတယ်။ Long-term management, production environment, team collaboration နဲ့ version control အတွက် declarative approach ကပိုသင့်တော်ပါတယ်။

CKA အတွက် imperative command တွေကို speed အတွက်သုံးပြီး complex configuration တွေမှာ YAML file နဲ့ declarative approach ကိုသုံးနိုင်အောင် လေ့ကျင့်ထားသင့်ပါတယ်။
