# 🌐 Kubernetes Service Types

Kubernetes Services provide a stable way to access **pods**, which are ephemeral and can be recreated anytime. A Service defines how clients reach those pods, both inside and outside the cluster.

---

## 🎯 Why Services?

Pods:

- Come and go (killed/restarted)
- Get dynamic IPs
- May run in multiple replicas (behind a deployment)

Services:

- Provide a **stable IP/DNS name**
- Enable **load balancing** across pods
- Act as a **single access point**

---

## 🧭 Overview of Service Types

| Type          | Reachable From        | Use Case                       |
|---------------|------------------------|--------------------------------|
| ClusterIP     | Inside the cluster     | Default; pod-to-pod communication |
| NodePort      | External via node IP   | Access from outside the cluster (dev/test) |
| LoadBalancer  | External via cloud LB  | Production-grade external access |

---

## 1️⃣ ClusterIP (default)

- Exposes a service **internally** within the cluster
- Assigns a **virtual IP (VIP)** only accessible from other pods

## 2️⃣ NodePort
Exposes the service on a static port on each node

Useful for local testing or bare metal clusters

## 3️⃣ LoadBalancer
Creates an external load balancer (via cloud provider)

Maps traffic to a ClusterIP service underneath
