# 🚀 CKA (Certified Kubernetes Administrator) Roadmap

## 📌 Exam Overview

- **Type:** Performance-based (hands-on)
- **Duration:** 2 hours
- **Environment:** Real Kubernetes cluster
- **Key Skill:** `kubectl` command + troubleshooting

---

## 🧠 Prerequisites

### 🔹 Linux Basics

- File system commands: `ls`, `cd`, `cat`, `grep`, `find`
- Process checking: `ps`, `top`
- Networking basics: `curl`, `netstat`, `ss`
- Text editing: `vim` or `nano`

### 🔹 YAML Basics

- Indentation
- Key-value structure
- Kubernetes manifest writing

### 🔹 Container Basics

- Docker image
- Container lifecycle
- Container logs

---

## ⚙️ Kubernetes Core Concepts

- Pod
- Deployment
- ReplicaSet
- Service
- Namespace
- ConfigMap
- Secret
- Ingress

---

## 📚 CKA Main Topics

### 🏗️ 1. Cluster Architecture

Learn these topics:

- `kubeadm`
- Control Plane vs Worker Node
- API Server
- Scheduler
- Controller Manager
- kubelet
- kube-proxy
- etcd backup and restore

---

### 📦 2. Workloads & Scheduling

Practice these:

- Create Deployment
- Scale Deployment
- Rolling Update
- Rollback
- Job and CronJob
- nodeSelector
- taints and tolerations
- affinity and anti-affinity

Example:

```bash
kubectl create deployment web --image=nginx
kubectl scale deployment web --replicas=3
kubectl rollout status deployment web
kubectl rollout undo deployment web
```

---

### 🌐 3. Services & Networking

Important topics:

- ClusterIP
- NodePort
- LoadBalancer
- CoreDNS
- NetworkPolicy
- Ingress

Example:

```bash
kubectl expose deployment web --port=80 --target-port=80 --type=ClusterIP
kubectl get svc
```

---

### 💾 4. Storage

Learn these:

- Persistent Volume (PV)
- Persistent Volume Claim (PVC)
- StorageClass
- Volume mounts

Important idea:

> Pod is temporary, but storage can be persistent using PV and PVC.

---

### 🛠️ 5. Troubleshooting

This is the most important part for CKA.

Practice debugging:

- Pod `CrashLoopBackOff`
- Pod `Pending`
- Node `NotReady`
- Service not reachable
- DNS issue
- kubelet issue
- wrong image name
- wrong selector labels

Useful commands:

```bash
kubectl get pods -A
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl get events --sort-by=.metadata.creationTimestamp
kubectl get nodes
kubectl describe node <node-name>
```

---

## 📅 6 Weeks Study Plan

## Week 1 — Linux, YAML, kubectl Basics

Study:

- Linux commands
- YAML format
- kubectl basic commands

Practice:

```bash
kubectl get nodes
kubectl get pods
kubectl run nginx --image=nginx
kubectl describe pod nginx
kubectl delete pod nginx
```

---

## Week 2 — Pods, Deployments, Services

Study:

- Pod lifecycle
- Deployment
- ReplicaSet
- Service types

Practice:

```bash
kubectl create deployment web --image=nginx
kubectl get deploy
kubectl expose deployment web --port=80 --type=NodePort
kubectl get svc
```

---

## Week 3 — Scheduling, ConfigMap, Secret

Study:

- nodeSelector
- taints and tolerations
- ConfigMap
- Secret

Practice:

```bash
kubectl create configmap app-config --from-literal=APP_ENV=prod
kubectl create secret generic app-secret --from-literal=PASSWORD=123456
```

---

## Week 4 — Storage and Networking

Study:

- PV
- PVC
- StorageClass
- NetworkPolicy
- Ingress
- CoreDNS

Practice:

```bash
kubectl get pv
kubectl get pvc
kubectl get storageclass
kubectl get ingress
```

---

## Week 5 — Cluster Setup, Upgrade, Backup

Study:

- kubeadm cluster setup
- cluster upgrade
- etcd backup
- etcd restore

Important command examples:

```bash
kubectl drain <node-name> --ignore-daemonsets
kubectl uncordon <node-name>
```

---

## Week 6 — Troubleshooting and Mock Exams

Study:

- Pod troubleshooting
- Node troubleshooting
- Network troubleshooting
- Service troubleshooting

Practice:

- Do mock exam
- Practice with timer
- Repeat weak topics
- Use official Kubernetes docs quickly

---

## 💻 Daily Practice Routine

Run these commands daily:

```bash
kubectl run nginx --image=nginx
kubectl get pods -o wide
kubectl describe pod nginx
kubectl logs nginx
kubectl delete pod nginx
```

Generate YAML:

```bash
kubectl create deployment web --image=nginx --dry-run=client -o yaml > web.yaml
```

Apply YAML:

```bash
kubectl apply -f web.yaml
kubectl get all
```

Delete resources:

```bash
kubectl delete -f web.yaml
```

---

## 🔥 Best Resources

- KodeKloud CKA Course
- killer.sh Exam Simulator
- Kubernetes Official Documentation
- Play with Kubernetes
- kind or minikube for local practice

---

## 💡 Pro Tips

- Do not only watch videos. Practice commands.
- Learn `kubectl explain`.
- Learn how to generate YAML fast.
- Practice with timer.
- Focus heavily on troubleshooting.
- Know how to search Kubernetes official docs quickly.

Useful shortcuts:

```bash
alias k=kubectl
export do="--dry-run=client -o yaml"
export now="--force --grace-period=0"
```

Example:

```bash
k create deployment web --image=nginx $do > web.yaml
```

---

## 🎯 Final Goal

Before taking the CKA exam, you should be able to:

- Create Kubernetes resources quickly
- Debug broken pods and services
- Understand cluster components
- Use kubectl efficiently
- Read official documentation fast
- Finish mock exam tasks within time limit

---

## ✅ Final Motivation

CKA is not about memorizing theory.  
CKA is about solving Kubernetes problems quickly in a real terminal.

Practice every day, and you can pass it. 💪🔥
