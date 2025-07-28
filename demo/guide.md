# 🧪 Kubernetes Networking Hands-On Guide

## 1️⃣ Create Pod and Inspect Network

```bash
kubectl apply -f 01-pod-demo.yaml
kubectl get pod net-demo -o wide
kubectl exec -it net-demo -- sh
ip a       # See eth0 with Pod IP
ip route   # See default route to bridge (e.g., cni0)## 🧪 Kubernetes Networking Demo Guide for K3s + Docker

### 1️⃣ Apply the Demo Pod

```bash
bash
CopyEdit
kubectl apply -f 01-pod-demo.yaml
kubectl get pod net-demo -o wide

```

### ✅ Inside the Pod

```bash
bash
CopyEdit
kubectl exec -it net-demo -- sh
ip a        # View eth0 (Pod IP)
ip route    # Default route to cni0

```

---

### 2️⃣ On the Node: Inspect Pause Container, Veth & Bridge

### Find Pod Container

```bash
bash
CopyEdit
sudo ctr -n k8s.io containers list | grep net-demo

```

### Get PID of pause container

```bash
bash
CopyEdit
sudo ctr -n k8s.io tasks ls | grep net-demo
# Use the PID shown

```

### Explore Network Namespace

```bash
bash
CopyEdit
sudo nsenter -t <PID> -n ip a
sudo nsenter -t <PID> -n ip link

```

### View veth pairs and bridge

```bash
bash
CopyEdit
ip link               # Shows vethXY and vethYZ
brctl show            # Shows bridge (usually cni0)

```

---

### 3️⃣ ClusterIP Service

```bash
bash
CopyEdit
kubectl apply -f 02-service-demo.yaml
kubectl get svc clusterip-demo

```

### Test DNS inside another pod

```bash
bash
CopyEdit
kubectl run curl --image=curlimages/curl -it --rm -- sh
nslookup clusterip-demo
curl clusterip-demo:80

```

---

### 4️⃣ NodePort Service

```bash
bash
CopyEdit
kubectl apply -f 03-nodeport-demo.yaml
kubectl get svc nodeport-demo

```

### Access via external IP:

```bash
bash
CopyEdit
curl http://<NODE_IP>:30080

```

Use `hostname -I` or `ip a` to get your node’s IP.

---

### 5️⃣ LoadBalancer (Optional for K3s)

K3s does **not support LoadBalancer natively** unless you install **MetalLB**:

### a. Install MetalLB:

```bash
bash
CopyEdit
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.10/config/manifests/metallb-native.yaml

```

### b. Configure IP pool (replace subnet with yours):

```yaml
yaml
CopyEdit
# metallb-config.yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: default-pool
  namespace: metallb-system
spec:
  addresses:
  - 192.168.1.240-192.168.1.250

---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: example
  namespace: metallb-system

```

```bash
bash
CopyEdit
kubectl apply -f metallb-config.yaml

```

Then apply:

```bash
bash
CopyEdit
kubectl apply -f 04-loadbalancer-demo.yaml
kubectl get svc lb-demo

```

---

### 6️⃣ Ingress

K3s comes with **Traefik** as the default ingress controller.

### a. Check that Traefik is running

```bash
bash
CopyEdit
kubectl get pods -n kube-system | grep traefik

```

### b. Apply ingress + service

```bash
bash
CopyEdit
kubectl apply -f 02-service-demo.yaml
kubectl apply -f 05-ingress-demo.yaml

```

### c. Add to /etc/hosts

```bash
bash
CopyEdit
sudo nano /etc/hosts
# Add:
127.0.0.1 net-demo.local

```

### d. Access in browser:

```bash
bash
CopyEdit
http://net-demo.local

```

Or test with curl:

```bash
bash
CopyEdit
curl -H "Host: net-demo.local" http://localhost

```