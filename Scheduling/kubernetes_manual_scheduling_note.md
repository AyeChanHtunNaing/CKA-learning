# Kubernetes Manual Scheduling

## 1. Manual Scheduling ဆိုတာဘာလဲ

**Manual Scheduling** ဆိုတာ Kubernetes ရဲ့ default scheduler ကို မသုံးဘဲ Pod ကို ဘယ် Node ပေါ်မှာ run မလဲဆိုတာ ကိုယ်တိုင် သတ်မှတ်ပေးတာပါ။

ပုံမှန်အားဖြင့် Kubernetes scheduler က Pod တွေကို အလိုအလျောက် Node တစ်ခုခုမှာ တင်ပေးပါတယ်။ ဒါပေမယ့် တချို့ special case တွေမှာ Pod ကို သတ်မှတ်ထားတဲ့ Node ပေါ်မှာပဲ run စေချင်ရင် manual scheduling ကို သုံးနိုင်ပါတယ်။

---

## 2. Default Scheduler Behavior

Pod manifest တစ်ခုမှာ `nodeName` ဆိုတဲ့ field ရှိပါတယ်။

ပုံမှန် Pod YAML မှာတော့ `nodeName` မထည့်ထားပါဘူး။

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    name: nginx
spec:
  containers:
    - name: nginx
      image: nginx
      ports:
        - containerPort: 8080
```

ဒီလို `nodeName` မပါတဲ့ Pod ကို create လုပ်လိုက်ရင် Kubernetes Scheduler က:

1. `nodeName` မရှိတဲ့ Pod တွေကို ရှာတယ်။
2. သင့်တော်တဲ့ Node တစ်ခုကို ရွေးတယ်။
3. Pod ကို အဲ့ဒီ Node ပေါ်မှာ assign လုပ်တယ်။
4. `nodeName` field ကို update လုပ်ပေးတယ်။

---

## 3. Pod ကို Node တစ်ခုပေါ်မှာ ကိုယ်တိုင် တင်ချင်ရင်

Pod ကို create လုပ်တဲ့အချိန်မှာ `spec.nodeName` ထည့်ပေးရပါတယ်။

ဥပမာ `node02` ပေါ်မှာ nginx Pod ကို run ချင်ရင်:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    name: nginx
spec:
  nodeName: node02
  containers:
    - name: nginx
      image: nginx
      ports:
        - containerPort: 8080
```

ဒီ YAML ကို apply လုပ်လိုက်ရင် Pod က `node02` ပေါ်မှာ run ပါမယ်။

```bash
kubectl apply -f pod.yaml
```

စစ်ကြည့်ရန်:

```bash
kubectl get pods -o wide
```

Output example:

```bash
NAME    READY   STATUS    RESTARTS   AGE   IP          NODE
nginx   1/1     Running   0          9s    10.40.0.4   node02
```

ဒီမှာ `NODE` column မှာ `node02` လို့ ပြရင် Pod က `node02` ပေါ်မှာ run နေတာပါ။

---

## 4. အရေးကြီးတဲ့ Point

`nodeName` ကို **Pod create လုပ်တဲ့အချိန်မှာပဲ** ထည့်လို့ရပါတယ်။

Pod running ဖြစ်ပြီးသွားတဲ့နောက်မှာ `nodeName` ကို ပြန်ပြင်လို့ မရပါဘူး။

ဥပမာ ဒီလိုလုပ်ရင် မရပါဘူး:

```bash
kubectl edit pod nginx
```

ပြီးတော့ `nodeName` ပြောင်းချင်တာမျိုး Kubernetes က ခွင့်မပြုပါဘူး။

---

## 5. Running Pod ကို နောက် Node တစ်ခုသို့ ပြောင်းချင်ရင်

Pod က running ဖြစ်ပြီးသားဆိုရင် `nodeName` ကို တိုက်ရိုက်ပြောင်းလို့ မရပါဘူး။

ဒီလိုအခြေအနေမှာ **Binding Object** ကို သုံးနိုင်ပါတယ်။

Binding object က scheduler လုပ်ပုံအတိုင်း Pod ကို Node တစ်ခုဆီ bind လုပ်ပေးတဲ့ object ပါ။

Binding YAML example:

```yaml
apiVersion: v1
kind: Binding
metadata:
  name: nginx
target:
  apiVersion: v1
  kind: Node
  name: node02
```

ဒီမှာ:

- `metadata.name: nginx` က bind လုပ်မယ့် Pod name ဖြစ်ပါတယ်။
- `target.name: node02` က assign လုပ်ချင်တဲ့ Node name ဖြစ်ပါတယ်။

---

## 6. Binding Object ကို API နဲ့ ပို့ခြင်း

Binding object ကို YAML ကနေ JSON format ပြောင်းပြီး Kubernetes API ကို POST request ပို့ရပါတယ်။

Example:

```bash
curl --header "Content-Type: application/json" \
  --request POST \
  --data @binding.json \
  http://$SERVER/api/v1/namespaces/default/pods/nginx/binding
```

ဒါက `nginx` Pod ကို `node02` Node ပေါ်မှာ bind လုပ်ဖို့ Kubernetes ကို ပြောတာပါ။

---

## 7. Manual Scheduling နည်းလမ်း ၂ ခု

| နည်းလမ်း          | ဘယ်အချိန်သုံးမလဲ                               | အဓိက Field/Object |
| ----------------- | ---------------------------------------------- | ----------------- |
| Direct Assignment | Pod create လုပ်တဲ့အချိန် Node သတ်မှတ်ချင်ရင်   | `nodeName`        |
| Binding Object    | Pod ကို Node တစ်ခုဆီ manually bind လုပ်ချင်ရင် | `Binding` object  |

---

## 8. CKA Exam အတွက် မှတ်ရန်

CKA မှာ ဒီ topic ထဲက အရေးကြီးတာတွေက:

### မှတ်ထားရမယ့် command

```bash
kubectl get pods -o wide
```

Pod ဘယ် Node ပေါ်မှာ run နေလဲ စစ်ဖို့ သုံးပါတယ်။

### မှတ်ထားရမယ့် YAML field

```yaml
spec:
  nodeName: node02
```

Pod ကို specific node ပေါ်မှာ run ချင်ရင် သုံးပါတယ်။

### အရေးကြီးဆုံး Concept

Pod create မလုပ်ခင်:

```yaml
nodeName: node02
```

ထည့်လို့ရတယ်။

Pod running ဖြစ်ပြီးနောက်:

```yaml
nodeName
```

ပြောင်းလို့ မရပါဘူး။

---

## 9. Simple Summary

Manual Scheduling ဆိုတာ Pod ကို Kubernetes scheduler အလိုအလျောက် မရွေးစေဘဲ ကိုယ်တိုင် Node သတ်မှတ်ပေးတာပါ။

Pod create လုပ်တဲ့အချိန်မှာ `nodeName` ထည့်ပေးရင် Pod က အဲ့ဒီ Node ပေါ်မှာ run ပါမယ်။

```yaml
spec:
  nodeName: node02
```

Pod running ဖြစ်ပြီးသွားရင် `nodeName` ကို ပြောင်းလို့မရပါဘူး။ Running Pod ကို manually assign လုပ်ချင်ရင် Binding object ကို သုံးရပါတယ်။

---

## 10. Quick Practice

### Create pod manually on node02

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  nodeName: node02
  containers:
    - name: nginx
      image: nginx
```

Apply:

```bash
kubectl apply -f nginx-pod.yaml
```

Check:

```bash
kubectl get pods -o wide
```

Expected: `NODE` column မှာ `node02` ပြရပါမယ်။
