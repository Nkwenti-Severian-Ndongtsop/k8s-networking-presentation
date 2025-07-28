# 🔌 CNI – Container Network Interface

In Kubernetes, networking is handled through a powerful standard called **CNI** (Container Network Interface). It’s the component responsible for **assigning IP addresses and connecting pods to the network**.

---

## 🧱 What is CNI?

CNI is a specification and set of plugins used by container runtimes (like `containerd`, `crun`, or `Kata`) to configure network interfaces for Linux containers.

> 📌 Think of CNI as the tool that **plugs the pod into the network**.

---

## ⚙️ CNI Responsibilities

- 📡 Assigns **unique IP** addresses to containers
- 🔗 Connects containers to the node’s network
- 🔌 Integrates with container runtimes via OCI standards

---

## 🧠 How It Works

```mermaid
graph LR
  A[Pod Creation Request] --> B[Kubelet]
  B --> C["Container Runtime (e.g. containerd)"]
  C --> D[CNI Plugin]
  D --> E[Configure Network + Add IP]
```
