# Kubernetes Scheduling Notes

This folder contains beginner-friendly Kubernetes scheduling notes for CKA preparation.  
The notes are written in a simple, readable style and include YAML examples, useful commands, and exam-focused reminders.

## 📌 About This Section

Kubernetes scheduling decides **where Pods should run** inside a cluster.  
This section covers the core scheduling topics required to understand Pod placement, node selection, workload priority, admission control, and scheduler customization.

Topics covered include:

- Manual Scheduling
- Labels, Selectors, and Annotations
- Taints and Tolerations
- Node Selectors
- Node Affinity
- Taints/Tolerations vs Node Affinity
- DaemonSets
- Static Pods
- Priority Classes
- Multiple Schedulers
- Scheduler Profiles
- Admission Controllers
- Validating and Mutating Admission Controllers

---

## 📂 Notes Index

| No. | Topic | File |
|---:|---|---|
| 1 | Manual Scheduling | [kubernetes_manual_scheduling_note.md](./kubernetes_manual_scheduling_note.md) |
| 2 | Labels, Selectors and Annotations | [kubernetes_labels_selectors_annotations_note.md](./kubernetes_labels_selectors_annotations_note.md) |
| 3 | Taints and Tolerations | [kubernetes_taints_tolerations_note.md](./kubernetes_taints_tolerations_note.md) |
| 4 | Node Selectors | [kubernetes_node_selectors_note.md](./kubernetes_node_selectors_note.md) |
| 5 | Node Affinity | [kubernetes_node_affinity_note.md](./kubernetes_node_affinity_note.md) |
| 6 | Taints and Tolerations vs Node Affinity | [kubernetes_taints_tolerations_vs_node_affinity_note.md](./kubernetes_taints_tolerations_vs_node_affinity_note.md) |
| 7 | DaemonSets | [kubernetes_daemonsets_note.md](./kubernetes_daemonsets_note.md) |
| 8 | Static Pods | [kubernetes_static_pods_note.md](./kubernetes_static_pods_note.md) |
| 9 | Priority Classes | [kubernetes_priority_classes_note.md](./kubernetes_priority_classes_note.md) |
| 10 | Multiple Schedulers | [kubernetes_multiple_schedulers_note.md](./kubernetes_multiple_schedulers_note.md) |
| 11 | Configuring Scheduler Profiles | [kubernetes_configuring_scheduler_profiles_note.md](./kubernetes_configuring_scheduler_profiles_note.md) |
| 12 | Admission Controllers | [kubernetes_admission_controllers_note.md](./kubernetes_admission_controllers_note.md) |
| 13 | Validating and Mutating Admission Controllers | [kubernetes_validating_mutating_admission_controllers_note.md](./kubernetes_validating_mutating_admission_controllers_note.md) |

---

## 🧠 Suggested Study Order

For better understanding, study the topics in this order:

1. **Manual Scheduling**  
   Learn how Pods can be assigned to specific nodes manually using `nodeName`.

2. **Labels, Selectors and Annotations**  
   Understand how Kubernetes objects are grouped and selected.

3. **Taints and Tolerations**  
   Learn how nodes can repel Pods unless the Pods tolerate specific taints.

4. **Node Selectors**  
   Learn simple node-based Pod placement using node labels.

5. **Node Affinity**  
   Learn advanced scheduling rules using operators such as `In`, `NotIn`, and `Exists`.

6. **Taints/Tolerations vs Node Affinity**  
   Understand when to use each and why both are often combined for dedicated nodes.

7. **DaemonSets**  
   Learn how to run one Pod on every node.

8. **Static Pods**  
   Learn how kubelet directly manages Pods from local manifest files.

9. **Priority Classes**  
   Understand how Kubernetes schedules important workloads before lower-priority ones.

10. **Multiple Schedulers and Scheduler Profiles**  
    Learn how custom scheduling behavior can be configured.

11. **Admission Controllers**  
    Learn how Kubernetes validates or modifies requests before saving objects to etcd.

---

## ⚙️ Useful Commands

### Check where Pods are running

```bash
kubectl get pods -o wide
```

### Show node labels

```bash
kubectl get nodes --show-labels
```

### Label a node

```bash
kubectl label nodes node-1 size=Large
```

### Filter Pods by label

```bash
kubectl get pods -l app=nginx
```

### Add a taint to a node

```bash
kubectl taint nodes node1 app=blue:NoSchedule
```

### Remove a taint from a node

```bash
kubectl taint nodes node1 app=blue:NoSchedule-
```

### Check taints on a node

```bash
kubectl describe node node1 | grep Taint
```

### Check DaemonSets

```bash
kubectl get ds
```

### Check PriorityClasses

```bash
kubectl get priorityclass
```

### Check scheduler events

```bash
kubectl get events -o wide
```

### Check kube-apiserver admission plugin flags

```bash
grep admission /etc/kubernetes/manifests/kube-apiserver.yaml
```

---

## 📝 Key Concepts Summary

### Manual Scheduling

Use `nodeName` when you want to place a Pod directly on a specific node.

```yaml
spec:
  nodeName: node02
```

### Node Selector

Use `nodeSelector` for simple key-value based node selection.

```yaml
nodeSelector:
  size: Large
```

### Node Affinity

Use node affinity for advanced node selection rules.

```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
        - matchExpressions:
            - key: size
              operator: In
              values:
                - Large
```

### Taints and Tolerations

Taints are applied to nodes. Tolerations are added to Pods.

```bash
kubectl taint nodes node1 app=blue:NoSchedule
```

```yaml
tolerations:
  - key: "app"
    operator: "Equal"
    value: "blue"
    effect: "NoSchedule"
```

### DaemonSet

A DaemonSet ensures that one copy of a Pod runs on every eligible node.

```yaml
kind: DaemonSet
apiVersion: apps/v1
```

### Static Pod

A Static Pod is managed directly by the kubelet using local manifest files.

Common path:

```bash
/etc/kubernetes/manifests
```

### PriorityClass

PriorityClass assigns scheduling importance to Pods.

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000000000
```

### Admission Controllers

Admission controllers validate or mutate API requests before objects are saved to etcd.

```text
Authentication → Authorization → Admission Control → etcd
```

---

## ✅ CKA Exam Focus

For the CKA exam, pay special attention to:

- `kubectl get pods -o wide`
- `kubectl get nodes --show-labels`
- `kubectl label nodes`
- `kubectl taint nodes`
- `nodeSelector`
- `nodeAffinity`
- `tolerations`
- `DaemonSet` YAML structure
- Static Pod manifest path
- `priorityClassName`
- `schedulerName`
- Admission controller flags in `kube-apiserver.yaml`

---

## 📚 Source

These notes are based on the Kubernetes scheduling lessons from KodeKloud CKA learning materials and organized for personal CKA exam preparation.

---

## 👤 Author

Created and maintained by [Aye Chan Htun Naing](https://github.com/AyeChanHtunNaing).

---

## ⭐ Goal

The goal of this folder is to provide clear and practical Kubernetes scheduling notes for CKA preparation, with examples that are easy to review before practice labs and the exam.
