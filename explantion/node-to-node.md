This explains how pods on different nodes communicate in a Kubernetes cluster, covering:

Cross-node pod communication

Role of kube-proxy and IP routing

Use of bridges and veth pairs

Why CNI alone is not enough

⚙️ What Enables This Routing?
CNI plugin support
Must support multi-host networking (e.g., Flannel, Calico, Cilium)

Kube-proxy

Sets up iptables or IPVS rules

Allows services (ClusterIP) to forward traffic between pods

Overlay or routed networks

Most CNI plugins create an overlay network (VXLAN, IP-in-IP)

Makes remote pods appear local

Traffic uses node-level routing

Each pod is still addressable by IP

🔌 Role of the Bridge (cni0)
On each node:

A Linux bridge connects all local pods

Bridge knows pod MAC addresses via ARP

Non-local packets are routed outside the bridge

🧪 Example IP Addresses
Node	Pod IP	Host IP
node1	192.168.1.2	10.244.0.1
node2	192.168.1.3	10.244.1.1

Even though the pods are on different nodes, they appear on the same subnet thanks to overlay networking.

🔍 kube-proxy and Services
If Pod A connects to a ClusterIP service, kube-proxy:

Uses iptables or IPVS to map service IP to a pod IP

Routes traffic to a backend (possibly on a remote node)

This allows:

bash
Copy
Edit
curl http://my-service
# Behind the scenes:
# Service IP → Pod IP on another node
💡 Key Concepts
Concept	Description
Overlay Network	Encapsulates pod traffic (e.g., VXLAN) to span nodes
Node Routing	Routes traffic to pods across nodes via host routing table
kube-proxy	Adds rules to forward service traffic
ARP	Resolves pod IP to MAC (on local bridge only)
CNI Plugins	Provide networking logic beyond the node

✅ Summary
Pods on different nodes communicate via routing

Overlay networks make this seamless

Bridges handle local traffic, routing handles cross-node traffic

kube-proxy is essential for service-to-pod routing

