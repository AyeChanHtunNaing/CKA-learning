# Container Orchestration

This folder contains notes for the **Container Orchestration** section from the *Introduction to Kubernetes* course.

Container orchestration is the process of automatically deploying, managing, scaling, networking, and recovering containers across multiple hosts or clusters. It is especially important when running containerized applications in QA and production environments.

---

## Notes

1. [Container Deployment](./container_deployment_notes.md)

2. [What Is Container Orchestration?](./container_orchestration_notes.md)

3. [Container Orchestrators](./container_orchestrators_notes.md)

4. [Why Use Container Orchestrators?](./why_use_container_orchestrators_notes.md)

5. [Where to Deploy Container Orchestrators?](./where_to_deploy_container_orchestrators_notes.md)

---

## Learning Flow

```text
Container Deployment
        ↓
What Is Container Orchestration?
        ↓
Container Orchestrators
        ↓
Why Use Container Orchestrators?
        ↓
Where to Deploy Container Orchestrators?
```

---

## Folder Structure

```text
Linux Foundation Courses/Introduction to Kubernetes/Container Orchestration/
├── README.md
├── container_deployment_notes.md
├── container_orchestration_notes.md
├── container_orchestrators_notes.md
├── why_use_container_orchestrators_notes.md
└── where_to_deploy_container_orchestrators_notes.md
```

---

## Quick Summary

### 1. Container Deployment

Containers package an application together with its runtime, libraries, and dependencies. They provide isolated and portable environments for running microservices across different infrastructures.

### 2. What Is Container Orchestration?

Container orchestration manages containers automatically across multiple servers. It provides fault tolerance, scalability, service discovery, external access, and seamless updates or rollbacks.

### 3. Container Orchestrators

Container orchestrators are tools or services that manage containerized applications at scale. Examples include Kubernetes, Amazon ECS, Azure Container Instances, Azure Service Fabric, Nomad, and Docker Swarm.

### 4. Why Use Container Orchestrators?

Manual container management is possible for a few containers, but not for hundreds or thousands. Orchestrators automate cluster management, container scheduling, networking, storage binding, load balancing, resource optimization, and security policies.

### 5. Where to Deploy Container Orchestrators?

Container orchestrators can run on bare metal, virtual machines, on-premises data centers, public cloud, hybrid cloud, local workstations, and managed Kubernetes services such as Amazon EKS, AKS, GKE, DigitalOcean Kubernetes, IBM Cloud Kubernetes Service, Oracle Container Engine for Kubernetes, and VMware Tanzu Kubernetes Grid.

---

## Key Concepts

| Concept | Meaning |
|---|---|
| Container | Isolated runtime environment for an application |
| Container Image | Packaged application template used to create containers |
| Container Runtime | Software that runs and manages containers |
| Container Orchestrator | Tool that manages containers at scale |
| Cluster | Group of machines working together |
| Node | A machine inside a cluster |
| Scheduling | Choosing where containers should run |
| Service Discovery | Helping services find and communicate with each other |
| Load Balancing | Distributing traffic across multiple containers |
| Rolling Update | Updating applications gradually without downtime |
| Rollback | Returning to a previous working version |

---

## Simple Explanation in Burmese

**Container Orchestration** ဆိုတာ container အများကြီးကို server အများကြီးပေါ်မှာ automatically manage လုပ်ပေးတဲ့ process ပါ။

Development မှာ container နည်းနည်းကို manually run လုပ်လို့ရပေမယ့် Production မှာတော့ container ရာချီ၊ ထောင်ချီ ဖြစ်လာနိုင်ပါတယ်။ အဲဒီအချိန်မှာ container တွေကို ကိုယ်တိုင် deploy, restart, scale, network ချိတ်, update, rollback လုပ်ဖို့ မလွယ်တော့ပါဘူး။

ဒါကြောင့် **Kubernetes** လို container orchestrator တွေကိုသုံးပြီး containerized applications တွေကို reliable, scalable, secure, and automated ဖြစ်အောင် manage လုပ်ပါတယ်။

---

## Recommended Reading Order

```text
1. container_deployment_notes.md
2. container_orchestration_notes.md
3. container_orchestrators_notes.md
4. why_use_container_orchestrators_notes.md
5. where_to_deploy_container_orchestrators_notes.md
```

---

## Main Takeaway

Container orchestration is essential for running containerized applications at production scale. It helps teams manage containers across clusters with automation, reliability, scalability, networking, storage, security, and zero-downtime updates.
