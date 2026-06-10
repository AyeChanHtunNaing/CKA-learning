# Application Lifecycle Management

This folder contains Kubernetes **Application Lifecycle Management** notes for CKA preparation.  
The notes focus on how applications are deployed, updated, configured, secured, and scaled in Kubernetes.

---

## 📌 Topics Covered

| No. | Topic | Note File |
|---|---|---|
| 1 | Rolling Updates and Rollbacks | [`kubernetes_rolling_updates_rollbacks_note.md`](./kubernetes_rolling_updates_rollbacks_note.md) |
| 2 | Commands and Arguments in Docker | [`docker_commands_arguments_note.md`](./docker_commands_arguments_note.md) |
| 3 | Commands and Arguments in Kubernetes | [`kubernetes_commands_arguments_note.md`](./kubernetes_commands_arguments_note.md) |
| 4 | Kubernetes Secrets | [`kubernetes_secrets_note.md`](./kubernetes_secrets_note.md) |
| 5 | Encrypting Secret Data at Rest | [`kubernetes_encrypting_secret_data_at_rest_note.md`](./kubernetes_encrypting_secret_data_at_rest_note.md) |
| 6 | Multi Container Pods | [`kubernetes_multi_container_pods_note.md`](./kubernetes_multi_container_pods_note.md) |
| 7 | Introduction to Autoscaling | [`kubernetes_autoscaling_intro_note.md`](./kubernetes_autoscaling_intro_note.md) |
| 8 | Horizontal Pod Autoscaler — HPA | [`kubernetes_hpa_note.md`](./kubernetes_hpa_note.md) |
| 9 | Vertical Pod Autoscaler — VPA | [`kubernetes_vpa_note.md`](./kubernetes_vpa_note.md) |
| 10 | In-place Resize of Pods | [`kubernetes_in_place_resize_pods_note.md`](./kubernetes_in_place_resize_pods_note.md) |

---

## 🎯 Learning Goals

After studying this section, you should understand how to:

- Manage application updates using **rollouts and rollbacks**
- Override container behavior using **command** and **args**
- Store sensitive data securely using **Secrets**
- Understand why Secrets are **Base64 encoded, not encrypted by default**
- Enable **encryption at rest** for Secrets stored in etcd
- Run tightly coupled containers using **multi-container Pods**
- Understand **horizontal vs vertical scaling**
- Use **HPA** to scale Pod replicas automatically
- Understand **VPA** for automatic resource recommendation and adjustment
- Understand the purpose of **in-place Pod resizing**

---

## 🧭 Recommended Study Order

For better understanding, study the notes in this order:

1. **Rolling Updates and Rollbacks**
2. **Commands and Arguments in Docker**
3. **Commands and Arguments in Kubernetes**
4. **Kubernetes Secrets**
5. **Encrypting Secret Data at Rest**
6. **Multi Container Pods**
7. **Introduction to Autoscaling**
8. **Horizontal Pod Autoscaler — HPA**
9. **Vertical Pod Autoscaler — VPA**
10. **In-place Resize of Pods**

---

## ⚡ CKA Quick Commands

### Rollout and Rollback

```bash
kubectl rollout status deployment/myapp-deployment
kubectl rollout history deployment/myapp-deployment
kubectl rollout undo deployment/myapp-deployment
kubectl get rs
```

Short form:

```bash
kubectl rollout status deploy/myapp-deployment
kubectl rollout undo deploy/myapp-deployment
kubectl get rs
```

---

### Update Deployment Image

```bash
kubectl set image deployment/myapp-deployment nginx-container=nginx:1.9.1
```

Short form:

```bash
kubectl set image deploy/myapp-deployment nginx-container=nginx:1.9.1
```

---

### Docker CMD and ENTRYPOINT Mapping

| Dockerfile | Kubernetes Pod YAML |
|---|---|
| `ENTRYPOINT` | `command` |
| `CMD` | `args` |

Example:

```yaml
command: ["sleep"]
args: ["3600"]
```

Imperative command:

```bash
kubectl run sleeper --image=ubuntu -- sleep 3600
```

---

### Create Secret

```bash
kubectl create secret generic app-secret \
  --from-literal=DB_Host=mysql \
  --from-literal=DB_User=root \
  --from-literal=DB_Password=paswrd
```

View Secret:

```bash
kubectl get secrets
kubectl describe secret app-secret
kubectl get secret app-secret -o yaml
```

Decode Secret:

```bash
kubectl get secret app-secret -o jsonpath='{.data.DB_Password}' | base64 --decode
```

---

### Secret as Environment Variables

```yaml
envFrom:
- secretRef:
    name: app-secret
```

---

### Secret as Volume

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

### Encrypt Secrets at Rest

Check API server encryption config:

```bash
ps -aux | grep kube-api | grep "encryption-provider-config"
```

Generate encryption key:

```bash
head -c 32 /dev/urandom | base64
```

Re-encrypt existing Secrets:

```bash
kubectl get secret --all-namespaces -o json | kubectl replace -f -
```

---

### Multi Container Pod Logs

```bash
kubectl logs <pod-name> -c <container-name>
```

Exec into a specific container:

```bash
kubectl exec -it <pod-name> -c <container-name> -- sh
```

---

### Manual Scaling

```bash
kubectl scale deployment/my-app --replicas=3
```

Short form:

```bash
kubectl scale deploy/my-app --replicas=3
```

---

### HPA

Create HPA:

```bash
kubectl autoscale deployment my-app --cpu-percent=50 --min=1 --max=10
```

Check HPA:

```bash
kubectl get hpa
kubectl describe hpa my-app
```

Delete HPA:

```bash
kubectl delete hpa my-app
```

---

### VPA

Install VPA:

```bash
kubectl apply -f https://github.com/kubernetes/autoscaler/releases/latest/download/vertical-pod-autoscaler.yaml
```

Check VPA components:

```bash
kubectl get pods -n kube-system | grep vpa
```

Check VPA recommendation:

```bash
kubectl describe vpa my-app-vpa
```

---

## 🧠 Key Concepts

### Rolling Update vs Recreate

| Strategy | Description | Downtime |
|---|---|---|
| `RollingUpdate` | Gradually replaces old Pods with new Pods | Usually no downtime |
| `Recreate` | Deletes all old Pods first, then creates new Pods | Downtime possible |

---

### ConfigMap vs Secret

| Resource | Used For |
|---|---|
| `ConfigMap` | Non-sensitive configuration |
| `Secret` | Passwords, tokens, keys, certificates |

> Kubernetes Secrets are Base64 encoded by default. Base64 is not encryption.

---

### HPA vs VPA

| Feature | HPA | VPA |
|---|---|---|
| Full Name | Horizontal Pod Autoscaler | Vertical Pod Autoscaler |
| Scales | Number of Pods | CPU/Memory requests and limits |
| Best For | Stateless apps, web services, microservices | Stateful apps, JVM apps, databases, AI workloads |
| Pod Restart | Usually not required for existing Pods | May evict/recreate Pods |

---

### Multi Container Pod

Containers inside the same Pod share:

- Same lifecycle
- Same network namespace
- Same Pod IP
- Same storage volumes

Common pattern:

```text
Main container + Sidecar container
```

Example:

```text
web-app + log-agent
```

---


## 📁 Folder Structure

```text
Application-Lifecycle-Management/
├── docker_commands_arguments_note.md
├── kubernetes_autoscaling_intro_note.md
├── kubernetes_commands_arguments_note.md
├── kubernetes_encrypting_secret_data_at_rest_note.md
├── kubernetes_hpa_note.md
├── kubernetes_in_place_resize_pods_note.md
├── kubernetes_multi_container_pods_note.md
├── kubernetes_rolling_updates_rollbacks_note.md
├── kubernetes_secrets_note.md
└── kubernetes_vpa_note.md
```

---

## ✅ Exam Focus

For the CKA exam, focus especially on:

1. `kubectl rollout`
2. `kubectl set image`
3. `command` and `args`
4. Secret creation and injection
5. Multi-container Pod troubleshooting
6. `kubectl logs -c`
7. `kubectl exec -c`
8. HPA basic commands
9. Resource requests and limits
10. Autoscaling concepts

---

## 📚 References

- Kubernetes Documentation
- KodeKloud CKA Course Notes
- Kubernetes Autoscaling Concepts
