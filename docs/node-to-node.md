# 🌐 Node-to-Node Communication in Kubernetes

In Kubernetes, pods often need to talk across **different nodes**. This requires more than just local bridges — it involves **routing**, **kube-proxy**, and **CNI plugins** capable of multi-host networking.

---

## 🔁 What Happens Across Nodes?

When **Pod A (on Node 1)** sends traffic to **Pod B (on Node 2)**:

- The source pod sends a packet to the destination pod’s IP
- The node’s network stack checks if it owns that IP
- If not, it routes it through the correct path (via VPC, overlay, etc.)
- The destination node receives and delivers it to Pod B via its bridge

---

### 🖼️ Concept Diagram

```mermaid
flowchart TB
  subgraph Node1
    A[Pod A<br>192.168.1.2]
    v1[veth0]
    br1[bridge cni0]
  end

  subgraph Node2
    B[Pod B<br>192.168.1.3]
    v2[veth1]
    br2[bridge cni0]
  end

  A --> v1 --> br1
  B --> v2 --> br2

  br1 -- via routing --> br2
```
