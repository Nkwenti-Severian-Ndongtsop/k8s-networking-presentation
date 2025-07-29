# 🔌 CNI – Container Network Interface

In Kubernetes, networking is handled through a powerful standard called **CNI** (Container Network Interface). It’s the component responsible for **assigning IP addresses and connecting pods to the network**.

---

## 🧱 What is CNI?

CNI is a specification and set of plugins used by container runtimes (like `containerd`, `crun`, or `Kata`) to configure network interfaces for Linux containers.

> 📌 Think of CNI as the tool that **plugs the pod into the network**.

---

## 🔌 CNI – Container Network Interface

CNI (Container Network Interface) is a standard interface and a set of plugins used to configure networking for containers. It's triggered by the container runtime when a pod is created.

It handles:

- 📡 Assigning unique IP addresses to pods
- 🔗 Connecting pods to the node's network
- 🧩 Working with container runtimes like `containerd`, `crun`, `Kata`

## 🧠 How It Works

```mermaid
graph LR
  A[Pod Creation Request] --> B[Kubelet]
  B --> C["Container Runtime (e.g. containerd)"]
  C --> D[CNI Plugin]
  D --> E[Configure veth pair]
  E --> F[Assign Pod IP + Add route]
  F --> G[Connect Pod to Node's bridge or overlay network]
```
