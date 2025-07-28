Kubelet receives a pod spec

It asks the container runtime to create the pod

The runtime calls the CNI plugin

CNI plugin:

Creates a virtual network interface (veth)

Assigns an IP

Connects it to the host/bridge network

🔌 CNI Plugin Types
Popular plugins include:

bridge: Creates a local Linux bridge

ipvlan: Gives direct access to the host’s interface

loopback: For internal container communication

Third-party: Calico, Flannel, Cilium, etc.

📎 You can chain plugins to apply policies (e.g., IPAM + bridge)

🧪 Example: What CNI Does
Let’s say a pod with 2 containers is created:

CNI assigns a shared network namespace

A virtual interface eth0 is created inside the pod

A unique IP is assigned (e.g., 192.168.1.4)

Every pod acts like a tiny VM on the network.

🖼️ Sketch Reference (from your diagram)
CNI: Adds IP

Pod: Gets unique IP

Plugins: bridge, ipvlan, loopback

Runtime: containerd, crun, kata

📚 Key Terms
Term	Description
CNI	Standard for container networking
Plugin	Executable that sets up network (bridge, etc.)
veth pair	Virtual Ethernet link between host and pod
IPAM	IP Address Management
CRI	Container Runtime Interface (calls CNI)
OCI	Open Container Initiative (runtime standard)

✅ Summary
CNI is what connects containers to the network in Kubernetes.
It works behind the scenes when a pod is created, assigning IPs and setting up connectivity.