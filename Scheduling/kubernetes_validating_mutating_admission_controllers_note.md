# Kubernetes Validating and Mutating Admission Controllers Note

## 1. Overview

ဒီ lesson မှာ Kubernetes **Admission Controllers** ထဲက အရေးကြီးတဲ့ အမျိုးအစား ၂ မျိုးဖြစ်တဲ့ -

- **Mutating Admission Controllers**
- **Validating Admission Controllers**

အကြောင်းကို လေ့လာပါမယ်။

Admission Controller ဆိုတာ API Server က request တစ်ခုကို etcd ထဲမသိမ်းခင် စစ်ဆေး၊ ပြင်ဆင်၊ allow/reject လုပ်နိုင်တဲ့ mechanism ပါ။

Request flow အလွယ်မှတ်ရန် -

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

Admission Controllers က Kubernetes cluster security, policy enforcement, default configuration injection တွေအတွက် အရေးကြီးပါတယ်။

---

## 2. Validating Admission Controller ဆိုတာဘာလဲ

**Validating Admission Controller** ဆိုတာ request object က သတ်မှတ်ထားတဲ့ policy/rule တွေကိုက်ညီလား စစ်ပြီး allow သို့မဟုတ် reject လုပ်တဲ့ controller ပါ။

ဥပမာ -

- Namespace ရှိ/မရှိ စစ်တယ်။
- Pod security policy ကိုက်/မကိုက် စစ်တယ်။
- Image registry policy ကိုက်/မကိုက် စစ်တယ်။
- Required labels ပါ/မပါ စစ်တယ်။

အလွယ်မှတ်ရန် -

```text
Validating = စစ်ပြီး allow/reject လုပ်တာ
```

ဥပမာ namespace မရှိဘဲ Pod create လုပ်ရင် NamespaceLifecycle admission controller က reject လုပ်နိုင်ပါတယ်။

```bash
kubectl run nginx --image nginx --namespace blue
```

`blue` namespace မရှိရင် request ကို reject လုပ်ပါမယ်။

---

## 3. Mutating Admission Controller ဆိုတာဘာလဲ

**Mutating Admission Controller** ဆိုတာ request object ကို etcd ထဲမသိမ်းခင် modify/change လုပ်နိုင်တဲ့ controller ပါ။

ဥပမာ -

- PVC မှာ storageClassName မပါရင် default storage class ထည့်ပေးတယ်။
- Pod မှာ default service account ထည့်ပေးတယ်။
- Required label ကို auto add လုပ်ပေးတယ်။
- Security defaults တွေ inject လုပ်ပေးတယ်။

အလွယ်မှတ်ရန် -

```text
Mutating = object ကို ပြင်ပြီးမှ သိမ်းတာ
```

---

## 4. PVC Default StorageClass Example

PersistentVolumeClaim တစ်ခု create လုပ်တဲ့အခါ storage class မထည့်ထားဘူးဆိုပါစို့။

Initial PVC request -

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
```

Admission Controller ဖြတ်ပြီးနောက် PVC မှာ default storage class ထည့်ပြီးသား ဖြစ်နိုင်ပါတယ်။

Modified PVC -

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
  storageClassName: default
```

ဒီလို object ကို modify လုပ်တဲ့ controller ကို Mutating Admission Controller အဖြစ် သတ်မှတ်နိုင်ပါတယ်။

---

## 5. Mutating vs Validating Admission Controllers

| Type | အလုပ် | Example |
|---|---|---|
| Mutating Admission Controller | Request object ကို modify လုပ်တယ် | Default storage class ထည့်ခြင်း |
| Validating Admission Controller | Request object ကို စစ်ပြီး allow/reject လုပ်တယ် | Namespace မရှိရင် reject လုပ်ခြင်း |

အရေးကြီးဆုံးက -

```text
Mutating controllers usually run before validating controllers.
```

ဘာကြောင့်လဲဆိုတော့ validating controller က final object ကို စစ်သင့်လို့ပါ။

ဥပမာ -

1. Mutating controller က missing namespace ကို auto create လုပ်တယ်။
2. Validating controller က namespace ရှိမရှိ စစ်တယ်။
3. Namespace ရှိသွားပြီဖြစ်လို့ request proceed ဖြစ်နိုင်တယ်။

ဒါပေမယ့် validating controller က အရင် run ရင် namespace မရှိသေးလို့ request reject ဖြစ်နိုင်ပါတယ်။

---

## 6. Admission Controller Reject ဖြစ်ရင်

Admission Controller တစ်ခုခုက request ကို reject လုပ်လိုက်ရင် request တစ်ခုလုံး fail ဖြစ်ပါတယ်။

အရေးကြီးသော rule -

```text
Any admission controller rejects = Whole request denied
```

User က error message ပြန်ရပါမယ်။

![Mutating and Validating Admission Controllers](https://kodekloud.com/kk-media/image/upload/v1752869883/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-2025-Updates-Validating-and-Mutating-Admission-Controllers/frame_140.jpg)

---

## 7. Built-in vs External Admission Controllers

Kubernetes မှာ built-in admission controllers တွေက Kubernetes source code ထဲမှာ ပါပြီးသား ဖြစ်ပါတယ်။

ဥပမာ -

- NamespaceLifecycle
- ServiceAccount
- DefaultStorageClass
- LimitRanger
- AlwaysPullImages
- NodeRestriction

ဒါပေမယ့် custom validation/mutation logic လိုရင် external admission controllers ကို webhook mechanism နဲ့ implement လုပ်နိုင်ပါတယ်။

External webhook types ၂ မျိုးရှိပါတယ်။

- **Mutating Admission Webhook**
- **Validating Admission Webhook**

---

## 8. Admission Webhook ဆိုတာဘာလဲ

**Admission Webhook** ဆိုတာ Kubernetes API Server က admission request ကို external server တစ်ခုဆီ ပို့ပြီး allow/reject/mutate ဆုံးဖြတ်ခိုင်းတဲ့ mechanism ပါ။

Webhook server က cluster ထဲမှာလည်း run နိုင်သလို cluster အပြင်ဘက်မှာလည်း run နိုင်ပါတယ်။

Flow -

```text
API Server
   ↓
Built-in Admission Controllers
   ↓
External Admission Webhook Server
   ↓
Webhook response
   ↓
Allow / Reject / Mutate
```

---

## 9. AdmissionReview Object

API Server က webhook server ကို request ပို့တဲ့အခါ **AdmissionReview** JSON object ပို့ပါတယ်။

AdmissionReview ထဲမှာ request details တွေပါပါတယ်။

ဥပမာ -

- User information
- Operation type
- Resource type
- Object metadata
- UID
- Namespace
- Request object

AdmissionReview request example -

```json
{
  "apiVersion": "admission.k8s.io/v1",
  "kind": "AdmissionReview",
  "request": {
    "uid": "705ab415-6393-11e7-b7cc-4201a8000002",
    "kind": {"group": "autoscaling", "version": "v1", "kind": "Scale"},
    "resource": {"group": "apps", "version": "v1", "resource": "deployments"},
    "subresource": "scale",
    "requestKind": {"group": "autoscaling", "version": "v1", "kind": "Scale"},
    "requestResource": {"group": "apps", "version": "v1", "resource": "deployments"}
  }
}
```

---

## 10. AdmissionReview Response

Webhook server က AdmissionReview response ပြန်ပေးရပါတယ်။

Allow response example -

```json
{
  "apiVersion": "admission.k8s.io/v1",
  "kind": "AdmissionReview",
  "response": {
    "uid": "<value_from_request.uid>",
    "allowed": true
  }
}
```

Reject လုပ်ချင်ရင် -

```json
{
  "response": {
    "uid": "<value_from_request.uid>",
    "allowed": false,
    "status": {
      "message": "Request rejected by admission webhook"
    }
  }
}
```

အရေးကြီးဆုံး field က -

```json
"allowed": true
```

သို့မဟုတ်

```json
"allowed": false
```

ဖြစ်ပါတယ်။

`allowed: false` ဆိုရင် API Server က request ကို reject လုပ်ပါမယ်။

---

## 11. Webhook Server Implement လုပ်နည်း

Custom admission webhook server တစ်ခုရေးဖို့ webhook server က HTTP endpoint တွေ expose လုပ်ရပါမယ်။

ဥပမာ -

```text
/validate
/mutate
```

- `/validate` endpoint က request ကို စစ်ပြီး allow/reject response ပေးမယ်။
- `/mutate` endpoint က request object ကို patch ပြန်ပေးပြီး modify လုပ်မယ်။

Webhook server က AdmissionReview JSON request ကို receive လုပ်ပြီး AdmissionReview JSON response ပြန်ပေးရပါမယ်။

---

## 12. Go Webhook Server Structure

Go နဲ့ webhook server ရေးတဲ့အခါ request body ကို read လုပ်ပြီး AdmissionReview object အဖြစ် parse လုပ်ရပါတယ်။

Example structure -

```go
package main

import (
    "encoding/json"
    "flag"
    "fmt"
    "io/ioutil"
    "net/http"
    "k8s.io/api/admission/v1beta1"
    metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
    "k8s.io/klog"
)

// toAdmissionResponse creates an AdmissionResponse with an error message.
func toAdmissionResponse(err error) v1beta1.AdmissionResponse {
    return v1beta1.AdmissionResponse{
        Result: &metav1.Status{
            Message: err.Error(),
        },
    }
}

// admitFunc defines the signature for validators and mutators.
type admitFunc func(v1beta1.AdmissionReview) v1beta1.AdmissionResponse

// serve processes the HTTP request before calling the admitFunc.
func serve(w http.ResponseWriter, r *http.Request, admit admitFunc) {
    var data []byte
    if r.Body == nil {
        return
    }
    data, err := ioutil.ReadAll(r.Body)
    if err != nil {
        return
    }
    // Additional processing logic goes here...
}
```

CKA exam မှာ ဒီ code ကိုရေးဖို့ မလိုနိုင်ပေမယ့် AdmissionReview flow ကို နားလည်ထားသင့်ပါတယ်။

---

## 13. Python Webhook Server Example

အောက်က Python Flask example မှာ endpoints ၂ ခုရှိပါတယ်။

- `/validate`
- `/mutate`

### Validate Endpoint

`/validate` endpoint က object name နဲ့ user name တူနေရင် request ကို reject လုပ်ပါတယ်။

```python
from flask import Flask, request, jsonify
import base64

app = Flask(__name__)

@app.route("/validate", methods=["POST"])
def validate():
    object_name = request.json["request"]["object"]["metadata"]["name"]
    user_name = request.json["request"]["userInfo"]["name"]
    status = True
    message = ""
    if object_name == user_name:
        message = "You can't create objects with your own name"
        status = False
    return jsonify(
        {
            "response": {
                "allowed": status,
                "uid": request.json["request"]["uid"],
                "status": {"message": message},
            }
        }
    )
```

ဒီ logic ရဲ့ အဓိပ္ပါယ် -

```text
If object name == user name, reject the request.
```

---

## 14. Mutate Endpoint

`/mutate` endpoint က object metadata ထဲမှာ user name ကို label အဖြစ် ထည့်ပေးပါတယ်။

```python
@app.route("/mutate", methods=["POST"])
def mutate():
    user_name = request.json["request"]["userInfo"]["name"]
    patch = [{"op": "add", "path": "/metadata/labels/users", "value": user_name}]
    encoded_patch = base64.b64encode(str(patch).encode()).decode()
    return jsonify(
        {
            "response": {
                "allowed": True,
                "uid": request.json["request"]["uid"],
                "patch": encoded_patch,
                "patchType": "JSONPatch",
            }
        }
    )
```

ဒီ response မှာ -

```json
"patchType": "JSONPatch"
```

နဲ့

```json
"patch": "<base64_encoded_patch>"
```

ပါရပါတယ်။

အဓိပ္ပါယ်က Kubernetes API Server က object ကို JSONPatch နဲ့ mutate လုပ်မယ်ဆိုတာပါ။

---

## 15. Webhook Server ကို ဘယ်မှာ Run မလဲ

Webhook server ကို ၂ နည်းနဲ့ host လုပ်နိုင်ပါတယ်။

1. Cluster အပြင်ဘက် external server အဖြစ် run ခြင်း
2. Kubernetes cluster ထဲမှာ Pod/Deployment + Service အဖြစ် run ခြင်း

Cluster ထဲမှာ run မယ်ဆိုရင် API Server က webhook server ကို call လုပ်နိုင်ဖို့ Kubernetes Service တစ်ခုရှိရပါမယ်။

Flow -

```text
API Server -> Kubernetes Service -> Webhook Pod
```

---

## 16. ValidatingWebhookConfiguration ဆိုတာဘာလဲ

API Server ကို external validating webhook သုံးခိုင်းဖို့ `ValidatingWebhookConfiguration` object create လုပ်ရပါတယ်။

Example -

```yaml
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingWebhookConfiguration
metadata:
  name: "pod-policy.example.com"
webhooks:
- name: "pod-policy.example.com"
  clientConfig:
    service:
      namespace: "webhook-namespace"
      name: "webhook-service"
    caBundle: "Ci0tLS0tQk......tLS0K"
  rules:
  - apiGroups: [""]
    apiVersions: ["v1"]
    operations: ["CREATE"]
    resources: ["pods"]
    scope: "Namespaced"
```

ဒီ config ရဲ့ အဓိပ္ပါယ် -

```text
Pod CREATE request ဖြစ်လာရင် webhook-service ကို call လုပ်ပါ
```

---

## 17. ValidatingWebhookConfiguration Fields

| Field | Description |
|---|---|
| `apiVersion` | `admissionregistration.k8s.io/v1` |
| `kind` | `ValidatingWebhookConfiguration` |
| `metadata.name` | Webhook configuration name |
| `webhooks.name` | Webhook name |
| `clientConfig.service.namespace` | Webhook Service ရှိတဲ့ namespace |
| `clientConfig.service.name` | Webhook Service name |
| `clientConfig.caBundle` | TLS certificate CA bundle |
| `rules.operations` | Trigger operation, e.g. `CREATE` |
| `rules.resources` | Target resource, e.g. `pods` |
| `rules.scope` | `Namespaced` or `Cluster` |

---

## 18. MutatingWebhookConfiguration ဆိုတာဘာလဲ

Object ကို modify/mutate လုပ်ချင်ရင် `MutatingWebhookConfiguration` ကို သုံးပါတယ်။

Structure က ValidatingWebhookConfiguration နဲ့ ဆင်တူပါတယ်။

```yaml
apiVersion: admissionregistration.k8s.io/v1
kind: MutatingWebhookConfiguration
metadata:
  name: "pod-mutator.example.com"
webhooks:
- name: "pod-mutator.example.com"
  clientConfig:
    service:
      namespace: "webhook-namespace"
      name: "webhook-service"
    caBundle: "Ci0tLS0tQk......tLS0K"
  rules:
  - apiGroups: [""]
    apiVersions: ["v1"]
    operations: ["CREATE"]
    resources: ["pods"]
    scope: "Namespaced"
```

Mutating webhook က response ထဲမှာ patch ပြန်ပေးနိုင်ပါတယ်။

---

## 19. Webhook Configuration Important Points

Webhook configuration မှာ အရေးကြီးတာတွေက -

- API Server က webhook service ကို reach လုပ်နိုင်ရမယ်။
- TLS certificate/CA bundle မှန်ရမယ်။
- rules section မှာ correct resource/operation သတ်မှတ်ရမယ်။
- Webhook server က AdmissionReview JSON response မှန်မှန်ပြန်ပေးရမယ်။
- `allowed` field ပါရမယ်။
- Mutating webhook ဖြစ်ရင် patch ကို base64 encoded JSONPatch အဖြစ်ပြန်ပေးရမယ်။

---

## 20. Webhook Request Trigger Example

အောက်က rule ကို ကြည့်ပါ။

```yaml
rules:
- apiGroups: [""]
  apiVersions: ["v1"]
  operations: ["CREATE"]
  resources: ["pods"]
  scope: "Namespaced"
```

အဓိပ္ပါယ် -

```text
Namespaced Pod CREATE request ဖြစ်လာတိုင်း ဒီ webhook ကို call လုပ်ပါ
```

ဥပမာ -

```bash
kubectl run nginx --image nginx
```

လုပ်လိုက်ရင် API Server က webhook server ကို AdmissionReview request ပို့ပါမယ်။

---

## 21. Admission Webhook Flow

```text
User runs kubectl apply/create
      ↓
API Server receives request
      ↓
Authentication
      ↓
Authorization
      ↓
Mutating Admission Controllers
      ↓
Validating Admission Controllers
      ↓
Mutating/Validating Webhooks
      ↓
Allowed? 
   ┌───────┴────────┐
   │                │
 Yes               No
   │                │
Persist to etcd    Reject request
```

Admission Controller တစ်ခုခုက reject လုပ်ရင် request တစ်ခုလုံး fail ဖြစ်ပါတယ်။

---

## 22. Built-in Admission Controllers vs Webhooks

| Feature | Built-in Admission Controllers | Admission Webhooks |
|---|---|---|
| Location | Kubernetes source code ထဲမှာ ပါပြီးသား | External server/Service |
| Custom logic | Limited | ကိုယ်ရေးထားတဲ့ logic သုံးနိုင် |
| Examples | DefaultStorageClass, NamespaceLifecycle | Custom pod policy webhook |
| Configure | kube-apiserver flags | WebhookConfiguration object |
| Use case | Standard Kubernetes policies | Organization-specific policies |

---

## 23. CKA Exam အတွက် မှတ်ရန်

CKA exam မှာ Admission Controllers/Webhooks နဲ့ပတ်သက်ပြီး အောက်ပါအချက်တွေကို သိထားသင့်ပါတယ်။

- Admission Controllers က request ကို etcd ထဲမသိမ်းခင် စစ်တယ်။
- Mutating Admission Controller က object ကို modify လုပ်နိုင်တယ်။
- Validating Admission Controller က object ကို allow/reject လုပ်နိုင်တယ်။
- Mutating controllers တွေက typically validating controllers မတိုင်ခင် run တယ်။
- Admission Controller တစ်ခုခု reject လုပ်ရင် request တစ်ခုလုံး denied ဖြစ်တယ်။
- Built-in admission controllers တွေက Kubernetes source code ထဲမှာပါပြီးသား ဖြစ်တယ်။
- Custom validation/mutation လိုရင် admission webhooks သုံးနိုင်တယ်။
- Webhook request/response format က AdmissionReview JSON ဖြစ်တယ်။
- Validating webhook အတွက် `ValidatingWebhookConfiguration` သုံးတယ်။
- Mutating webhook အတွက် `MutatingWebhookConfiguration` သုံးတယ်။
- Webhook server ကို cluster ထဲမှာ run မယ်ဆိုရင် Service နဲ့ expose လုပ်ရမယ်။
- `caBundle` TLS config မှန်ရမယ်။
- Mutating webhook response မှာ `patch` နဲ့ `patchType: JSONPatch` ပါနိုင်တယ်။

---

## 24. အသုံးများတဲ့ Commands

### Webhook configurations ကြည့်ရန်

```bash
kubectl get validatingwebhookconfigurations
```

```bash
kubectl get mutatingwebhookconfigurations
```

### Specific webhook detail ကြည့်ရန်

```bash
kubectl describe validatingwebhookconfiguration <name>
```

```bash
kubectl describe mutatingwebhookconfiguration <name>
```

### Webhook server pods ကြည့်ရန်

```bash
kubectl get pods -n webhook-namespace
```

### Webhook service ကြည့်ရန်

```bash
kubectl get svc -n webhook-namespace
```

### Webhook server logs ကြည့်ရန်

```bash
kubectl logs <webhook-pod-name> -n webhook-namespace
```

### Test pod create လုပ်ရန်

```bash
kubectl run nginx --image nginx
```

---

## 25. Simple Summary

Validating and Mutating Admission Controllers က Kubernetes API Server ရဲ့ admission phase မှာ request object တွေကို policy အရ control လုပ်တဲ့ mechanism ပါ။

**Mutating Admission Controller** -

```text
Object ကို modify လုပ်တယ်
```

ဥပမာ -

```text
storageClassName မပါရင် default value ထည့်ပေးတယ်
```

**Validating Admission Controller** -

```text
Object ကို စစ်ပြီး allow/reject လုပ်တယ်
```

ဥပမာ -

```text
Namespace မရှိရင် reject လုပ်တယ်
```

Custom policy လိုရင် webhook သုံးနိုင်ပါတယ်။

Webhook types -

```text
MutatingWebhookConfiguration
ValidatingWebhookConfiguration
```

Webhook server က API Server ကပို့တဲ့ AdmissionReview JSON ကို receive လုပ်ပြီး allowed true/false response ပြန်ပေးရပါတယ်။

အရေးကြီးဆုံး မှတ်ရန် -

```text
Mutating = Change object
Validating = Allow or deny object
Webhook = External custom admission logic
AdmissionReview = API Server and webhook communication format
```
