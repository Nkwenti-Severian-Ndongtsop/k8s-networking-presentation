4️⃣ External DNS + Domain Name
To map a domain like app.example.com to your cluster:

Use External DNS (a Kubernetes controller)

Automatically updates DNS provider (e.g., Route53)

Maps domain → Ingress IP → Service

☁️ Cloud Provider Support Matrix
Feature	Bare Metal	AWS / GCP / Azure
NodePort	✅	✅
LoadBalancer	❌	✅
Ingress (with IP)	✅ (with MetalLB)	✅
External DNS	❌ (manual)	✅ (automated)

🔐 HTTPS & TLS
Ingress can:

Terminate TLS using certificates (e.g., via Let’s Encrypt)

Offload SSL from backend services

Route traffic securely

✅ Summary
Kubernetes provides flexible ways to expose internal services to the outside world:

Option	Use When
NodePort	Local/dev clusters
LoadBalancer	Cloud-native production deployments
Ingress	HTTP routing, path/domain-based rules
External DNS	Automated DNS + Ingress integration