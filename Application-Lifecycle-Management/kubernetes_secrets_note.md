# Kubernetes Secrets

> Topic: Kubernetes Secrets ကိုအသုံးပြုပြီး password, key, token စတဲ့ sensitive data တွေကို manage လုပ်ခြင်း

---

## 1. Overview

Kubernetes မှာ application configuration data တွေကို manage လုပ်တဲ့အခါ —

- non-sensitive data → `ConfigMap`
- sensitive data → `Secret`

ကိုသုံးပါတယ်။

Sensitive data ဆိုတာတွေက —

- database password
- API key
- token
- certificate
- private key
- username/password

စတာတွေပါ။

Password တွေကို application code ထဲ hardcode လုပ်တာ၊ ConfigMap ထဲ plain text ထည့်တာတွေက security risk အရမ်းကြီးပါတယ်။

---

## 2. Problem with Hardcoding Sensitive Data

ဥပမာ Python app တစ်ခု MySQL database ကို connect လုပ်တယ်ဆိုပါစို့။

```python
import os
from flask import Flask, render_template
import mysql.connector

app = Flask(__name__)

@app.route("/")
def main():
    mysql.connector.connect(
        host="mysql",
        database="mysql",
        user="root",
        password="paswrd"
    )
    return render_template('hello.html')

if __name__ == "__main__":
    app.run(host="0.0.0.0", port="8080")
```

ဒီ code ထဲမှာ password ကို hardcode လုပ်ထားပါတယ်။

```python
password="paswrd"
```

ဒါက မသင့်တော်ပါဘူး။ ဘာကြောင့်လဲဆိုတော့ —

- source code ထဲ password ပါသွားနိုင်တယ်
- GitHub / GitLab repo ထဲ leak ဖြစ်နိုင်တယ်
- developer အများကြီးမြင်နိုင်တယ်
- password rotate လုပ်ရခက်တယ်
- production security risk ဖြစ်တယ်

---

## 3. ConfigMap vs Secret

| Resource    | Use Case                                | Sensitive Data သင့်/မသင့် |
| ----------- | --------------------------------------- | ------------------------- |
| `ConfigMap` | app config, hostname, color, env config | မသင့်                     |
| `Secret`    | password, token, key, cert              | သင့်                      |

ဥပမာ —

| Data                 | Resource            |
| -------------------- | ------------------- |
| DB host = mysql      | ConfigMap or Secret |
| DB user = root       | ConfigMap or Secret |
| DB password = paswrd | Secret              |

Password ကို ConfigMap ထဲမထည့်သင့်ပါဘူး။

---

## 4. Kubernetes Secret ဆိုတာဘာလဲ?

`Secret` က Kubernetes object တစ်မျိုးဖြစ်ပြီး sensitive data တွေကို store လုပ်ဖို့သုံးပါတယ်။

Secret ထဲက data တွေက default အနေနဲ့ **Base64 encoded** ဖြစ်ပါတယ်။

Important:

```text
Base64 encoding is NOT encryption.
```

Base64 က encode လုပ်တာပဲဖြစ်ပြီး encryption မဟုတ်ပါဘူး။  
Access ရှိတဲ့သူတိုင်း decode လုပ်နိုင်ပါတယ်။

---

## 5. Encoded and Decoded Example

### Plain Text Values

```text
DB_Host=mysql
DB_User=root
DB_Password=paswrd
```

### Base64 Encoded Values

```text
DB_Host=bXlzcWw=
DB_User=cm9vdA==
DB_Password=cGFzd3Jk
```

Decoded ပြန်လုပ်ရင် —

```text
bXlzcWw=   -> mysql
cm9vdA==   -> root
cGFzd3Jk   -> paswrd
```

---

## 6. Secret Workflow

Kubernetes Secret ကိုသုံးတဲ့ flow က အဓိက ၂ ဆင့်ပါ။

1. Secret create လုပ်မယ်
2. Secret ကို Pod ထဲ inject လုပ်မယ်

Inject လုပ်တဲ့နည်း ၂ မျိုးရှိပါတယ်။

- Environment variables အဖြစ် inject လုပ်ခြင်း
- Volume အဖြစ် mount လုပ်ခြင်း

---

## 7. Create Secret — Imperative Method

Imperative method က command line ကနေတိုက်ရိုက် Secret create လုပ်တာပါ။

### One key-value Secret

```bash
kubectl create secret generic app-secret --from-literal=DB_Host=mysql
```

### Multiple key-value Secret

```bash
kubectl create secret generic app-secret \
  --from-literal=DB_Host=mysql \
  --from-literal=DB_User=root \
  --from-literal=DB_Password=paswrd
```

### Secret from file

ဥပမာ `app_secret.properties` file ရှိတယ်ဆိုပါစို့။

```text
DB_Host=mysql
DB_User=root
DB_Password=paswrd
```

Secret create လုပ်ရန်:

```bash
kubectl create secret generic app-secret --from-file=app_secret.properties
```

---

## 8. Create Secret — Declarative Method

Declarative method မှာ YAML file ရေးပြီး Secret create လုပ်ပါတယ်။

Secret YAML ထဲမှာ `data` field ကိုသုံးရင် value တွေကို Base64 encoded form နဲ့ထည့်ရပါတယ်။

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
data:
  DB_Host: bXlzcWw=
  DB_User: cm9vdA==
  DB_Password: cGFzd3Jk
```

Create လုပ်ရန်:

```bash
kubectl create -f secret-data.yaml
```

or

```bash
kubectl apply -f secret-data.yaml
```

---

## 9. Plain Text to Base64 Convert

Linux / macOS မှာ Base64 encode လုပ်ရန်:

```bash
echo -n 'mysql' | base64
echo -n 'root' | base64
echo -n 'paswrd' | base64
```

Output:

```text
bXlzcWw=
cm9vdA==
cGFzd3Jk
```

### `echo -n` သုံးရတဲ့အကြောင်း

`-n` မထည့်ရင် newline character ပါ encode ဖြစ်သွားနိုင်ပါတယ်။  
ဒါကြောင့် Secret encode လုပ်တဲ့အခါ `echo -n` သုံးတာပိုမှန်ပါတယ်။

---

## 10. Base64 Decode

Encoded value ကို decode ပြန်လုပ်ရန်:

```bash
echo -n 'bXlzcWw=' | base64 --decode
```

Output:

```text
mysql
```

Other examples:

```bash
echo -n 'cm9vdA==' | base64 --decode
echo -n 'cGFzd3Jk' | base64 --decode
```

Output:

```text
root
paswrd
```

---

## 11. View Secrets

### List Secrets

```bash
kubectl get secrets
```

Output example:

```text
NAME          TYPE     DATA   AGE
app-secret    Opaque   3      10m
```

Short form:

```bash
kubectl get secret
```

---

### Describe Secret

```bash
kubectl describe secret app-secret
```

`describe` command က sensitive values တွေကို မပြပါဘူး။  
Key name နဲ့ data count လောက်ပဲ ပြပါတယ်။

---

### View Secret YAML

```bash
kubectl get secret app-secret -o yaml
```

ဒီ command က Base64 encoded data တွေကိုပြပါတယ်။

---

## 12. Decode Secret Directly from Kubernetes

Secret ထဲက value ကို jsonpath နဲ့ယူပြီး decode လုပ်နိုင်ပါတယ်။

```bash
kubectl get secret app-secret -o jsonpath='{.data.DB_Password}' | base64 --decode
```

Output:

```text
paswrd
```

---

## 13. Inject Secret into Pod as Environment Variables

Secret ကို Pod ထဲ environment variables အဖြစ် inject လုပ်နိုင်ပါတယ်။

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
  labels:
    name: simple-webapp-color
spec:
  containers:
    - name: simple-webapp-color
      image: simple-webapp-color
      ports:
        - containerPort: 8080
      envFrom:
        - secretRef:
            name: app-secret
```

ဒီလိုရေးရင် `app-secret` ထဲက key တွေက container ထဲမှာ environment variables အဖြစ်ဝင်သွားပါတယ်။

ဥပမာ —

```text
DB_Host
DB_User
DB_Password
```

---

## 14. Inject Specific Secret Key as Environment Variable

Secret ထဲက key တစ်ခုချင်းစီကိုပဲ env variable အဖြစ်ထည့်ချင်ရင် `secretKeyRef` သုံးနိုင်ပါတယ်။

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
spec:
  containers:
    - name: simple-webapp-color
      image: simple-webapp-color
      env:
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: app-secret
              key: DB_Password
```

ဒီမှာ container ထဲမှာ `DB_PASSWORD` ဆိုတဲ့ env variable တစ်ခုရပါမယ်။

---

## 15. Mount Secret as Volume

Secret ကို file အဖြစ် mount လုပ်ချင်ရင် volume သုံးနိုင်ပါတယ်။

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
spec:
  containers:
    - name: simple-webapp-color
      image: simple-webapp-color
      volumeMounts:
        - name: app-secret-volume
          mountPath: /opt/app-secret-volumes
          readOnly: true
  volumes:
    - name: app-secret-volume
      secret:
        secretName: app-secret
```

ဒီလို mount လုပ်ရင် Secret ထဲက key တစ်ခုစီက file တစ်ခုစီအဖြစ် ဖြစ်လာပါတယ်။

```bash
ls /opt/app-secret-volumes
```

Output:

```text
DB_Host  DB_Password  DB_User
```

File content ကြည့်ရန်:

```bash
cat /opt/app-secret-volumes/DB_Password
```

Output:

```text
paswrd
```

---

## 16. Environment Variable vs Volume Mount

| Method         | Usage                                             | Advantage                        |
| -------------- | ------------------------------------------------- | -------------------------------- |
| `envFrom`      | Secret keys အားလုံးကို env variables အဖြစ်ထည့်    | Simple and easy                  |
| `secretKeyRef` | Secret key တစ်ခုချင်းစီကို env variable အဖြစ်ထည့် | More controlled                  |
| Volume mount   | Secret keys တွေကို files အဖြစ် mount              | Certificates/keys အတွက်အသုံးများ |

---

## 17. Secret Types

Default Secret type က `Opaque` ဖြစ်ပါတယ်။

```yaml
type: Opaque
```

Common Secret types:

| Type                                  | Usage                        |
| ------------------------------------- | ---------------------------- |
| `Opaque`                              | Generic key-value secrets    |
| `kubernetes.io/dockerconfigjson`      | Private registry credentials |
| `kubernetes.io/tls`                   | TLS certificate and key      |
| `kubernetes.io/service-account-token` | ServiceAccount token         |

CKA မှာ mostly `Opaque` generic secret ကိုတွေ့ရများပါတယ်။

---

## 18. Create TLS Secret

TLS certificate နဲ့ key ကို Secret အဖြစ် create လုပ်နိုင်ပါတယ်။

```bash
kubectl create secret tls my-tls-secret \
  --cert=tls.crt \
  --key=tls.key
```

---

## 19. Create Docker Registry Secret

Private Docker registry image pull လုပ်ဖို့ Secret create လုပ်နိုင်ပါတယ်။

```bash
kubectl create secret docker-registry regcred \
  --docker-server=<registry-server> \
  --docker-username=<username> \
  --docker-password=<password> \
  --docker-email=<email>
```

Pod ထဲမှာအသုံးပြုရန်:

```yaml
imagePullSecrets:
  - name: regcred
```

---

## 20. Security Considerations

Kubernetes Secret တွေက default အနေနဲ့ Base64 encoded ပဲဖြစ်ပါတယ်။  
Encryption မဟုတ်ပါဘူး။

![Secret Security Guidelines](https://kodekloud.com/kk-media/image/upload/v1752869672/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-Secrets/frame_470.jpg)

### Important Security Tips

- Secret YAML files တွေကို public GitHub repo ထဲမတင်ပါနဲ့။
- RBAC နဲ့ Secret access ကို limit လုပ်ပါ။
- etcd encryption at rest enable လုပ်ပါ။
- Secret ကိုလိုအပ်တဲ့ Pod/ServiceAccount တွေကိုပဲ access ပေးပါ။
- Production မှာ external secret manager တွေသုံးဖို့စဉ်းစားပါ။

---

## 21. External Secret Providers

Advanced production environment တွေမှာ Kubernetes Secret အစား third-party secret manager တွေသုံးနိုင်ပါတယ်။

Examples:

- AWS Secrets Manager
- Azure Key Vault
- GCP Secret Manager
- HashiCorp Vault
- External Secrets Operator

ဒီ tools တွေက —

- encryption ပိုကောင်းတယ်
- audit log ရနိုင်တယ်
- access control ပို granular ဖြစ်တယ်
- secret rotation လုပ်ရလွယ်တယ်

---

## 22. CKA Exam Tips

### Tip 1 — Secret create command ကိုမှတ်ထားပါ

```bash
kubectl create secret generic app-secret \
  --from-literal=DB_Host=mysql \
  --from-literal=DB_User=root \
  --from-literal=DB_Password=paswrd
```

---

### Tip 2 — Base64 encode/decode commands

Encode:

```bash
echo -n 'paswrd' | base64
```

Decode:

```bash
echo -n 'cGFzd3Jk' | base64 --decode
```

---

### Tip 3 — Secret ကို envFrom နဲ့ inject လုပ်နည်း

```yaml
envFrom:
  - secretRef:
      name: app-secret
```

---

### Tip 4 — Secret ကို volume အဖြစ် mount လုပ်နည်း

```yaml
volumes:
  - name: app-secret-volume
    secret:
      secretName: app-secret
```

```yaml
volumeMounts:
  - name: app-secret-volume
    mountPath: /opt/app-secret-volumes
    readOnly: true
```

---

### Tip 5 — Secret value ကိုတိုက်ရိုက် decode ကြည့်ရန်

```bash
kubectl get secret app-secret -o jsonpath='{.data.DB_Password}' | base64 --decode
```

---

## 23. Common Mistakes

### Mistake 1 — Plain text value ကို `data` ထဲထည့်ခြင်း

မမှန်:

```yaml
data:
  DB_Password: paswrd
```

မှန်:

```yaml
data:
  DB_Password: cGFzd3Jk
```

---

### Mistake 2 — Base64 ကို encryption လို့ထင်ခြင်း

Base64 က encryption မဟုတ်ပါဘူး။ Decode ပြန်လုပ်လို့ရပါတယ်။

---

### Mistake 3 — Secret file ကို public repo ထဲတင်ခြင်း

Secret YAML file တွေကို GitHub public repo ထဲမတင်သင့်ပါဘူး။

---

### Mistake 4 — Wrong key name သုံးခြင်း

Secret ထဲက key name နဲ့ Pod YAML ထဲက key name တူရပါမယ်။

```yaml
key: DB_Password
```

Secret ထဲမှာ `DB_PASSWORD` လို့ရေးထားပြီး Pod ထဲမှာ `DB_Password` လို့ရေးရင် error ဖြစ်နိုင်ပါတယ်။

---

## 24. Practice Example

### Task

`app-secret` ဆိုတဲ့ Secret တစ်ခု create လုပ်ပါ။

Values:

```text
DB_Host=mysql
DB_User=root
DB_Password=paswrd
```

ပြီးရင် `simple-webapp-color` Pod ထဲကို env variables အဖြစ် inject လုပ်ပါ။

---

### Solution 1 — Imperative Secret Create

```bash
kubectl create secret generic app-secret \
  --from-literal=DB_Host=mysql \
  --from-literal=DB_User=root \
  --from-literal=DB_Password=paswrd
```

Check:

```bash
kubectl get secret app-secret
kubectl describe secret app-secret
```

---

### Solution 2 — Pod YAML

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
spec:
  containers:
    - name: simple-webapp-color
      image: simple-webapp-color
      envFrom:
        - secretRef:
            name: app-secret
```

Apply:

```bash
kubectl apply -f pod.yaml
```

Check:

```bash
kubectl get pods
kubectl describe pod simple-webapp-color
```

---

## 25. Quick Reference

| Task                    | Command                                                                              |
| ----------------------- | ------------------------------------------------------------------------------------ |
| Create generic secret   | `kubectl create secret generic app-secret --from-literal=key=value`                  |
| Create secret from file | `kubectl create secret generic app-secret --from-file=app_secret.properties`         |
| Get secrets             | `kubectl get secrets`                                                                |
| Describe secret         | `kubectl describe secret app-secret`                                                 |
| View secret YAML        | `kubectl get secret app-secret -o yaml`                                              |
| Encode value            | `echo -n 'value' \| base64`                                                          |
| Decode value            | `echo -n 'encoded' \| base64 --decode`                                               |
| Decode from cluster     | `kubectl get secret app-secret -o jsonpath='{.data.DB_Password}' \| base64 --decode` |

---

## 26. Key Takeaways

- Sensitive data တွေအတွက် `Secret` ကိုသုံးပါ။
- Non-sensitive data တွေအတွက် `ConfigMap` ကိုသုံးပါ။
- Secret values တွေက Base64 encoded ဖြစ်ပြီး encryption မဟုတ်ပါဘူး။
- `kubectl create secret generic` နဲ့ Secret ကို command line ကနေ create လုပ်နိုင်ပါတယ်။
- YAML Secret `data` field ထဲ value ထည့်ရင် Base64 encoded ဖြစ်ရပါမယ်။
- Secret ကို Pod ထဲ env variable သို့မဟုတ် volume file အဖြစ် inject လုပ်နိုင်ပါတယ်။
- RBAC, etcd encryption, external secret managers တွေက production security အတွက်အရေးကြီးပါတယ်။
