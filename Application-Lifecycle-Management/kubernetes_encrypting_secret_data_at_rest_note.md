# Demo: Encrypting Secret Data at Rest

> Topic: Kubernetes Secret data ကို etcd ထဲမှာ encryption at rest enable လုပ်ပြီး secure ဖြစ်အောင်ပြုလုပ်ခြင်း

---

## 1. Overview

Kubernetes `Secret` တွေက default အနေနဲ့ **Base64 encoded** ပဲဖြစ်ပါတယ်။  
Encryption မဟုတ်ပါဘူး။

ဆိုလိုတာက —

- `kubectl get secret -o yaml` နဲ့ကြည့်ရင် encoded value တွေမြင်ရမယ်
- `base64 --decode` နဲ့ decode ပြန်လုပ်လို့ရမယ်
- etcd access ရှိတဲ့သူက secret value ကို decode လုပ်နိုင်နိုင်တယ်

ဒါကြောင့် production cluster တွေမှာ Secret data ကို **encryption at rest** enable လုပ်ပြီး etcd ထဲမှာ encrypted form နဲ့သိမ်းသင့်ပါတယ်။

---

## 2. Secret Data Flow in Kubernetes

Kubernetes Secret data flow ကိုအလွယ်မှတ်ရင် —

```text
kubectl create secret
        ↓
Kubernetes API Server
        ↓
etcd
```

Secret object တွေကို Kubernetes က etcd ထဲမှာ persist လုပ်ပါတယ်။

Encryption at rest မဖွင့်ထားရင် etcd ထဲမှာ Secret data တွေက Base64 encoded form နဲ့ပဲရှိနိုင်ပါတယ်။

---

## 3. Create a Secret Object

Secret ကို literal value နဲ့ create လုပ်နိုင်ပါတယ်။

```bash
kubectl create secret generic my-secret --from-literal=key1=supersecret
```

Verify:

```bash
kubectl get secret my-secret
```

Output example:

```text
NAME        TYPE     DATA   AGE
my-secret   Opaque   1      10s
```

Describe:

```bash
kubectl describe secret my-secret
```

`describe` command က actual secret value ကိုမပြပါဘူး။ Metadata နဲ့ key count လောက်ပဲပြပါတယ်။

---

## 4. Other Ways to Create Secrets

### Create Secret from all files in a directory

```bash
kubectl create secret generic my-secret --from-file=path/to/bar
```

### Create Secret using specific keys from files

```bash
kubectl create secret generic my-secret \
  --from-file=ssh-privatekey=path/to/id_rsa \
  --from-file=ssh-publickey=path/to/id_rsa.pub
```

### Create Secret from literals

```bash
kubectl create secret generic my-secret \
  --from-literal=key1=supersecret \
  --from-literal=key2=topsecret
```

### Create Secret from file and literal together

```bash
kubectl create secret generic my-secret \
  --from-file=ssh-privatekey=path/to/id_rsa \
  --from-literal=passphrase=topsecret
```

### Create Secret from env files

```bash
kubectl create secret generic my-secret \
  --from-env-file=path/to/foo.env \
  --from-env-file=path/to/bar.env
```

---

## 5. View Encoded Secret

Secret ကို YAML format နဲ့ကြည့်ပါ။

```bash
kubectl get secret my-secret -o yaml
```

Example output:

```yaml
apiVersion: v1
data:
  key1: c3VwZXJzZWNyZXQ=
kind: Secret
metadata:
  name: my-secret
  namespace: default
type: Opaque
```

ဒီမှာ `key1` value က Base64 encoded ဖြစ်ပါတယ်။

---

## 6. Decode Secret Value

Base64 encoded value ကို decode လုပ်ရန် —

```bash
echo "c3VwZXJzZWNyZXQ=" | base64 --decode
```

Output:

```text
supersecret
```

### Important

```text
Base64 encoding is NOT encryption.
```

Base64 က data ကို hide လုပ်ထားသလိုပဲ ဖြစ်ပြီး security အတွက် encryption မဟုတ်ပါဘူး။

---

## 7. Inspect Secret Data in etcd

Kubernetes cluster data တွေကို etcd ထဲမှာသိမ်းပါတယ်။  
Secret objects တွေလည်း etcd ထဲမှာပါသိမ်းပါတယ်။

Encryption at rest မဖွင့်ထားရင် etcd ထဲက Secret value တွေကို decode လုပ်နိုင်ပါတယ်။

---

## 8. Install etcdctl

Control plane node မှာ `etcdctl` မရှိသေးရင် install လုပ်ပါ။

```bash
apt-get install etcd-client
```

Verify:

```bash
etcdctl
```

---

## 9. Check etcd Certificates

etcd ကို query လုပ်ဖို့ certificate files တွေလိုပါတယ်။

Check:

```bash
ls /etc/kubernetes/pki/etcd/ca.crt
ls /etc/kubernetes/pki/etcd/server.crt
ls /etc/kubernetes/pki/etcd/server.key
```

---

## 10. Read Secret Directly from etcd

Secret path format:

```text
/registry/secrets/<namespace>/<secret-name>
```

Example:

```bash
ETCDCTL_API=3 etcdctl \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  get /registry/secrets/default/my-secret | hexdump -C
```

Encryption မရှိသေးရင် hexdump output ထဲမှာ secret value ကို readable form နဲ့တွေ့နိုင်ပါတယ်။

---

## 11. Check Whether Encryption at Rest is Enabled

kube-apiserver process ထဲမှာ `encryption-provider-config` flag ရှိ/မရှိစစ်ပါ။

```bash
ps -aux | grep kube-api | grep "encryption-provider-config"
```

Flag မတွေ့ရင် encryption at rest မဖွင့်ထားတာဖြစ်နိုင်ပါတယ်။

---

## 12. Enable Encryption at Rest

Encryption at rest enable လုပ်ဖို့ steps —

1. Encryption key generate လုပ်မယ်
2. Encryption configuration file create လုပ်မယ်
3. kube-apiserver manifest ထဲ flag ထည့်မယ်
4. config file ကို kube-apiserver container ထဲ mount လုပ်မယ်
5. kube-apiserver restart ဖြစ်ပြီး encryption enable ဖြစ်မယ်
6. New secrets တွေ encrypted ဖြစ်မဖြစ် verify လုပ်မယ်
7. Existing secrets တွေကို re-encrypt လုပ်မယ်

---

## 13. Generate Encryption Key

32-byte random key generate လုပ်ရန် —

```bash
head -c 32 /dev/urandom | base64
```

Example output:

```text
y0xTt+U6xgRdNxe4nDYYsijOGgRDoUYC+wAwOKeNfPs=
```

ဒီ key ကို encryption configuration file ထဲမှာထည့်သုံးပါမယ်။

---

## 14. Create Encryption Configuration File

`enc.yaml` file create လုပ်ပါ။

```yaml
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: y0xTt+U6xgRdNxe4nDYYsijOGgRDoUYC+wAwOKeNfPs=
      - identity: {}
```

Check file:

```bash
cat enc.yaml
```

---

## 15. Encryption Providers Explanation

ဒီ config ထဲမှာ provider ၂ ခုရှိပါတယ်။

```yaml
providers:
  - aescbc:
      keys:
        - name: key1
          secret: <base64-key>
  - identity: {}
```

| Provider   | Meaning                                                |
| ---------- | ------------------------------------------------------ |
| `aescbc`   | Secret data ကို AES-CBC နဲ့ encrypt လုပ်မယ်            |
| `identity` | Encryption မလုပ်ဘဲ raw/plain storage ကို allow လုပ်မယ် |

### Important

Provider order အရေးကြီးပါတယ်။

```yaml
- aescbc
- identity
```

ဆိုရင် new writes တွေကို `aescbc` နဲ့ encrypt လုပ်ပါတယ်။  
`identity` က old unencrypted data တွေကို still read လုပ်နိုင်အောင် fallback အဖြစ်ထားတာပါ။

---

## 16. Move Encryption Config to Secure Directory

Directory create:

```bash
mkdir -p /etc/kubernetes/enc
```

Move file:

```bash
mv enc.yaml /etc/kubernetes/enc/
```

Final path:

```text
/etc/kubernetes/enc/enc.yaml
```

---

## 17. Modify kube-apiserver Manifest

kubeadm cluster မှာ kube-apiserver static pod manifest က ဒီ path မှာရှိပါတယ်။

```bash
/etc/kubernetes/manifests/kube-apiserver.yaml
```

Edit:

```bash
vi /etc/kubernetes/manifests/kube-apiserver.yaml
```

---

## 18. Add Encryption Provider Config Flag

`kube-apiserver` command section ထဲမှာ flag ထည့်ပါ။

```yaml
- --encryption-provider-config=/etc/kubernetes/enc/enc.yaml
```

Example:

```yaml
spec:
  containers:
    - command:
        - kube-apiserver
        - --advertise-address=192.168.1.10
        - --encryption-provider-config=/etc/kubernetes/enc/enc.yaml
```

---

## 19. Add Volume Mount

kube-apiserver container ထဲကို `/etc/kubernetes/enc` directory mount လုပ်ရပါမယ်။

`volumeMounts` ထဲမှာထည့်ပါ။

```yaml
volumeMounts:
  - name: enc
    mountPath: /etc/kubernetes/enc
    readOnly: true
```

---

## 20. Add Volume

`volumes` section ထဲမှာ hostPath volume ထည့်ပါ။

```yaml
volumes:
  - name: enc
    hostPath:
      path: /etc/kubernetes/enc
      type: DirectoryOrCreate
```

---

## 21. Combined kube-apiserver Manifest Example

```yaml
spec:
  containers:
    - command:
        - kube-apiserver
        - --encryption-provider-config=/etc/kubernetes/enc/enc.yaml
      volumeMounts:
        - name: enc
          mountPath: /etc/kubernetes/enc
          readOnly: true
  volumes:
    - name: enc
      hostPath:
        path: /etc/kubernetes/enc
        type: DirectoryOrCreate
```

Save လုပ်ပြီးနောက် kubelet က kube-apiserver static pod ကို auto restart လုပ်ပါမယ်။

---

## 22. Verify kube-apiserver Restart

Check control plane pods:

```bash
kubectl get pods -n kube-system
```

Check kube-apiserver:

```bash
kubectl get pod -n kube-system | grep kube-apiserver
```

Check process flag:

```bash
ps -aux | grep kube-api | grep "encryption-provider-config"
```

---

## 23. Create New Secret After Encryption Enabled

Encryption enable ပြီးနောက် new Secret create လုပ်ပါ။

```bash
kubectl create secret generic my-secret-2 --from-literal=key2=topsecret
```

Verify:

```bash
kubectl get secret
```

---

## 24. Verify Secret is Encrypted in etcd

New secret ကို etcd ထဲမှာ inspect လုပ်ပါ။

```bash
ETCDCTL_API=3 etcdctl \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  get /registry/secrets/default/my-secret-2 | hexdump -C
```

Encrypted ဖြစ်နေပြီဆိုရင် output ထဲမှာ `topsecret` ကို readable text အဖြစ်မမြင်ရတော့ပါဘူး။

---

## 25. Existing Secrets Are Not Automatically Encrypted

Encryption at rest enable လုပ်ပြီးနောက် **new secrets only** encrypted ဖြစ်ပါတယ်။

Encryption မဖွင့်ခင် create လုပ်ထားတဲ့ old secrets တွေက automatic encrypted မဖြစ်ပါဘူး။

Existing secrets တွေကို re-encrypt လုပ်ဖို့ update/replace လုပ်ရပါမယ်။

---

## 26. Re-encrypt Existing Secrets

All namespaces ထဲက existing secrets တွေကို re-encrypt လုပ်ရန် —

```bash
kubectl get secret --all-namespaces -o json | kubectl replace -f -
```

ဒီ command က existing secrets တွေကို Kubernetes API server ကတစ်ဆင့် ပြန် write လုပ်စေပါတယ်။  
ပြန် write လုပ်တဲ့အချိန်မှာ encryption provider config အရ encrypted form နဲ့ etcd ထဲသိမ်းသွားပါမယ်။

---

## 27. Verify Re-encryption

Old secret ကို etcd မှာပြန် inspect လုပ်ပါ။

```bash
ETCDCTL_API=3 etcdctl \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  get /registry/secrets/default/my-secret | hexdump -C
```

Secret value readable text အဖြစ်မမြင်ရတော့ရင် encrypted at rest အောင်မြင်ပါပြီ။

---

## 28. Important File Paths

| Item                        | Path                                            |
| --------------------------- | ----------------------------------------------- |
| kube-apiserver manifest     | `/etc/kubernetes/manifests/kube-apiserver.yaml` |
| Encryption config directory | `/etc/kubernetes/enc`                           |
| Encryption config file      | `/etc/kubernetes/enc/enc.yaml`                  |
| etcd CA cert                | `/etc/kubernetes/pki/etcd/ca.crt`               |
| etcd server cert            | `/etc/kubernetes/pki/etcd/server.crt`           |
| etcd server key             | `/etc/kubernetes/pki/etcd/server.key`           |

---

## 29. Important Commands Summary

| Task                     | Command                                                                           |
| ------------------------ | --------------------------------------------------------------------------------- |
| Create Secret            | `kubectl create secret generic my-secret --from-literal=key1=supersecret`         |
| View Secret YAML         | `kubectl get secret my-secret -o yaml`                                            |
| Decode Base64            | `echo "encoded-value" \| base64 --decode`                                         |
| Install etcdctl          | `apt-get install etcd-client`                                                     |
| Read Secret from etcd    | `ETCDCTL_API=3 etcdctl ... get /registry/secrets/default/my-secret \| hexdump -C` |
| Check encryption flag    | `ps -aux \| grep kube-api \| grep "encryption-provider-config"`                   |
| Generate encryption key  | `head -c 32 /dev/urandom \| base64`                                               |
| Edit API server manifest | `vi /etc/kubernetes/manifests/kube-apiserver.yaml`                                |
| Re-encrypt all secrets   | `kubectl get secret --all-namespaces -o json \| kubectl replace -f -`             |

---

## 30. CKA / CKS Exam Tips

### Tip 1 — Base64 is not encryption

```text
Secret default = Base64 encoded
Encryption at rest = extra API server configuration needed
```

---

### Tip 2 — kube-apiserver flag မှတ်ထားပါ

```bash
--encryption-provider-config=/etc/kubernetes/enc/enc.yaml
```

---

### Tip 3 — Static pod manifest ပြင်ရမယ့် path

```bash
/etc/kubernetes/manifests/kube-apiserver.yaml
```

---

### Tip 4 — Existing secrets must be re-written

```bash
kubectl get secret --all-namespaces -o json | kubectl replace -f -
```

---

### Tip 5 — etcd path format မှတ်ထားပါ

```text
/registry/secrets/<namespace>/<secret-name>
```

Example:

```text
/registry/secrets/default/my-secret
```

---

## 31. Common Mistakes

### Mistake 1 — Secret ကို Base64 encoded ဖြစ်တာနဲ့ secure လို့ထင်ခြင်း

Base64 decode လုပ်လို့ရတာကြောင့် encryption မဟုတ်ပါဘူး။

---

### Mistake 2 — Encryption enable ပြီး old secrets encrypted ဖြစ်ပြီလို့ထင်ခြင်း

Encryption at rest enable လုပ်ပြီးနောက် **new writes only** encrypted ဖြစ်ပါတယ်။  
Old secrets တွေကို re-encrypt လုပ်ဖို့ `kubectl replace` command run ရပါမယ်။

---

### Mistake 3 — kube-apiserver manifest ထဲ volume mount မထည့်ခြင်း

`--encryption-provider-config` flag ပေးထားပေမယ့် file ကို container ထဲ mount မလုပ်ထားရင် kube-apiserver start မဖြစ်နိုင်ပါဘူး။

လိုအပ်တာ —

```yaml
volumeMounts:
  - name: enc
    mountPath: /etc/kubernetes/enc
    readOnly: true
```

```yaml
volumes:
  - name: enc
    hostPath:
      path: /etc/kubernetes/enc
      type: DirectoryOrCreate
```

---

### Mistake 4 — Wrong YAML indentation

Encryption config YAML indentation မှားရင် kube-apiserver error ဖြစ်နိုင်ပါတယ်။

---

## 32. Lab Flow Summary

```text
1. Create secret
2. View secret YAML
3. Decode base64 value
4. Inspect etcd and confirm secret is visible
5. Generate encryption key
6. Create enc.yaml
7. Move enc.yaml to /etc/kubernetes/enc/
8. Edit kube-apiserver manifest
9. Add encryption-provider-config flag
10. Add volumeMount and volume
11. Wait for kube-apiserver restart
12. Create new secret
13. Inspect etcd and confirm value is encrypted
14. Re-encrypt old secrets
15. Verify again
```

---

## 33. Practice Example

### Task

Enable encryption at rest for Kubernetes Secrets using `aescbc`.

### Solution Steps

Generate key:

```bash
head -c 32 /dev/urandom | base64
```

Create config:

```bash
mkdir -p /etc/kubernetes/enc
vi /etc/kubernetes/enc/enc.yaml
```

`enc.yaml`:

```yaml
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: <GENERATED_BASE64_KEY>
      - identity: {}
```

Edit kube-apiserver:

```bash
vi /etc/kubernetes/manifests/kube-apiserver.yaml
```

Add flag:

```yaml
- --encryption-provider-config=/etc/kubernetes/enc/enc.yaml
```

Add mount:

```yaml
volumeMounts:
  - name: enc
    mountPath: /etc/kubernetes/enc
    readOnly: true
```

Add volume:

```yaml
volumes:
  - name: enc
    hostPath:
      path: /etc/kubernetes/enc
      type: DirectoryOrCreate
```

Re-encrypt old secrets:

```bash
kubectl get secret --all-namespaces -o json | kubectl replace -f -
```

---

## 34. Key Takeaways

- Kubernetes Secrets are Base64 encoded by default, not encrypted.
- Secrets are stored in etcd.
- Anyone with etcd access may read secret data if encryption at rest is not enabled.
- Encryption at rest is configured through kube-apiserver.
- Use `EncryptionConfiguration` with providers like `aescbc`.
- kube-apiserver needs `--encryption-provider-config` flag.
- Encryption config file must be mounted into kube-apiserver static pod.
- New secrets are encrypted after enabling encryption.
- Existing secrets must be re-written to be encrypted.
