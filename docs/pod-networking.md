# 🚀 Kubernetes Networking: CNI & Pod Communication

### 🔄 Pod-to-Pod Communication

In Kubernetes, **every pod gets a unique IP**, and **pods can talk to each other directly** using these IPs, regardless of which node they're on.

### 🧠 How it works:

- 🟢 **Same Node:**
    
    Pods communicate via the **local Linux bridge** set up by CNI. Traffic stays on the host.
    
- 🔵 **Across Nodes:**
    
    The traffic is routed through the **node's network** using **routing tables** or **overlay networks** (e.g., Flannel, Calico).

  ```mermaid
  sequenceDiagram
    participant PodA
    participant NodeA
    participant CNI_Network
    participant NodeB
    participant PodB

    PodA->>NodeA: Sends packet to PodB IP
    NodeA->>CNI_Network: Forwards packet via routing rule/overlay
    CNI_Network->>NodeB: Delivers packet to target node
    NodeB->>PodB: Delivers packet to PodB via its veth
  ```