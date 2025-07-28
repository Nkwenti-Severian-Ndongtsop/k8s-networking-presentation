🔁 How Services Work Behind the Scenes
kube-proxy sets up iptables or IPVS rules

Traffic to service IP is redirected to backend pods

Load balancing occurs across endpoints

Pods don’t need to know pod IPs — they talk to services.

🧠 Quick Recap
Concept	Role
ClusterIP	Default service for internal-only traffic
NodePort	Exposes service on a node’s external IP
LoadBalancer	Cloud-integrated external access
ExternalName	DNS-based service redirection
kube-proxy	Handles routing rules behind the scenes

✅ Summary
Kubernetes Services abstract away pod lifecycles and provide stable access points:

Use ClusterIP for internal communication

Use NodePort or LoadBalancer for external access

Use ExternalName to integrate external services