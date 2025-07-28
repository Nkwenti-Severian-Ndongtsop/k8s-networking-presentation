📦 Pod Network Namespace
All containers in a pod share the same network namespace

They communicate using localhost (127.0.0.1)

A pause container is created first to hold the network namespace

🧠 All other containers (like busybox, nginx) join the pause container’s network space.

🔌 CNI in Action
CNI plugin creates a veth pair:

One end in the pod (eth0)

One end on the host (vethXYZ)

Assigns IP (e.g., 192.168.1.4) to eth0 inside pod

veth connects pod to the bridge on the host

Traffic enters pod via eth0

🧪 Output Example

kubectl get pods -o wide
# OUTPUT:
# NAME                IP              NODE
# shared-namespace    192.168.1.4     node01

✅ Summary
In a single pod:

All containers share the same IP

They use localhost to talk

CNI ensures the pod is attached to the cluster network