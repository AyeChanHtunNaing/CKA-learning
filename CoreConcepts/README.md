# CKA Learning – Core Concepts

This folder contains my study notes for the **Certified Kubernetes Administrator (CKA)** exam, focused on the **Core Concepts** domain.  
The notes are written to make Kubernetes fundamentals easier to understand with simple explanations, practical commands, YAML examples, and visual diagrams.

## 📚 Topics Covered

| No. | Topic | Description |
|---|---|---|
| 1 | [Kubernetes Cluster Architecture](./kubernetes_cluster_architecture_note.md) | Explains the relationship between the Control Plane and Worker Nodes |
| 2 | [Control Plane Components](./control_plane_components_note.md) | Covers `kube-apiserver`, `etcd`, `kube-scheduler`, and `kube-controller-manager` |
| 3 | [Worker Node Components](./worker_node_components_note.md) | Explains `kubelet`, `kube-proxy`, container runtime, Pods, and Services |
| 4 | [Pods](./kubernetes_pods_note.md) | Introduces Pods as the smallest deployable unit in Kubernetes |
| 5 | [ReplicaSets](./kubernetes_replicaset_note.md) | Explains how ReplicaSets maintain the desired number of Pod replicas |
| 6 | [Deployments](./kubernetes_deployments_note.md) | Covers Deployment, ReplicaSet, Pod relationship, rolling updates, and rollbacks |
| 7 | [Services](./kubernetes_services_note.md) | Explains ClusterIP, NodePort, LoadBalancer, selectors, and service traffic flow |
| 8 | [Namespaces](./kubernetes_namespaces_note.md) | Covers resource isolation, namespace commands, DNS format, and ResourceQuota |
| 9 | [Imperative vs Declarative](./imperative_vs_declarative_note.md) | Compares command-based and YAML-based Kubernetes management |
| 10 | [`kubectl apply` Command](./kubectl_apply_command_note.md) | Explains declarative apply, three-way merge, and last-applied configuration |
| 11 | [Certification Tips](./kubernetes_certification_tips_imperative_commands.md) | Provides useful imperative commands for faster CKA exam practice |

## 🗂️ Folder Structure

```text
Core Concepts/
├── images/
├── control_plane_components_note.md
├── imperative_vs_declarative_note.md
├── kubectl_apply_command_note.md
├── kubernetes_certification_tips_imperative_commands.md
├── kubernetes_cluster_architecture_note.md
├── kubernetes_deployments_note.md
├── kubernetes_namespaces_note.md
├── kubernetes_pods_note.md
├── kubernetes_replicaset_note.md
├── kubernetes_services_note.md
└── worker_node_components_note.md
```

## 🎯 Purpose of This Folder

The goal of this folder is to help me build a strong foundation in Kubernetes Core Concepts before practicing advanced CKA tasks.

These notes are useful for:

- Understanding Kubernetes architecture clearly
- Reviewing CKA Core Concepts quickly
- Practicing important `kubectl` commands
- Learning YAML structure for Kubernetes resources
- Preparing for hands-on Kubernetes labs
- Revising before the CKA exam

## 🧠 Key Learning Flow

A good learning order for this folder is:

```text
Cluster Architecture
        ↓
Control Plane Components
        ↓
Worker Node Components
        ↓
Pods
        ↓
ReplicaSets
        ↓
Deployments
        ↓
Services
        ↓
Namespaces
        ↓
Imperative vs Declarative
        ↓
kubectl apply
        ↓
Certification Tips
```

## ⚙️ Common Commands Practiced

```bash
kubectl get nodes
kubectl get pods
kubectl get pods -A
kubectl run nginx --image=nginx
kubectl create deployment nginx --image=nginx
kubectl get deployments
kubectl get replicasets
kubectl get svc
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl apply -f <file-name>.yaml
kubectl delete -f <file-name>.yaml
```

## 📝 Notes Style

The notes are designed to be:

- Beginner-friendly
- Practical and command-focused
- Easy to revise
- Suitable for CKA exam preparation
- Supported with visual diagrams where useful

## 🚀 CKA Exam Reminder

For the CKA exam, speed and accuracy are important.  
Use imperative commands for quick object creation and YAML files for more complex configurations.

Useful pattern:

```bash
kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > deployment.yaml
kubectl expose pod nginx --port=80 --dry-run=client -o yaml > service.yaml
```

## 📌 References

- Kubernetes Documentation: https://kubernetes.io/docs/
- kubectl Reference: https://kubernetes.io/docs/reference/kubectl/
- CKA Learning Practice Notes

## ✅ Status

This folder is part of my ongoing CKA learning journey.  
More notes, examples, and diagrams may be added as I continue studying Kubernetes.
