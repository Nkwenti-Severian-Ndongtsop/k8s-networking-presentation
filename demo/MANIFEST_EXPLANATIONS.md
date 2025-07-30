# Kubernetes Networking Demo - Manifest Explanations

This document provides detailed explanations of each manifest file in the demo, what it does, and what networking concepts it demonstrates.

## 📋 Table of Contents

1. [Namespace Setup](#namespace-setup)
2. [CNI and Bridge Networking](#cni-and-bridge-networking)
3. [ClusterIP Service](#clusterip-service)
4. [NodePort Service](#nodeport-service)
5. [LoadBalancer Service](#loadbalancer-service)
6. [Ingress Resources](#ingress-resources)
7. [Network Policies](#network-policies)
8. [Scalable Deployment](#scalable-deployment)
9. [Test Client](#test-client)
10. [DNS and Service Discovery](#dns-and-service-discovery)

---

## 1. Namespace Setup

**File:** `01-namespace.yaml`

### What it does:
Creates a dedicated namespace called `networking-demo` to organize all demo resources and prevent conflicts with other workloads.

### Key Components:
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: networking-demo
  labels:
    name: networking-demo
    purpose: demonstration
```

### What it demonstrates:
- **Resource isolation**: Namespaces provide logical separation of resources
- **Labeling strategy**: Using labels for organization and filtering
- **Demo organization**: Keeping all demo resources in one place

### Networking Concepts:
- **Namespace isolation**: Network policies can be applied at namespace level
- **DNS resolution**: Services are accessible as `<service>.<namespace>.svc.cluster.local`
- **Resource management**: Clean separation of demo resources from other workloads

---

## 2. CNI and Bridge Networking

**File:** `02-cni-demo-pods.yaml`

### What it does:
Creates three pods that demonstrate CNI functionality, bridge networking, and shared network namespaces.

### Key Components:

#### Pod 1: `cni-demo-pod-1`
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: cni-demo-pod-1
  namespace: networking-demo
  labels:
    app: cni-demo
    role: frontend
spec:
  containers:
  - name: nginx
    image: nginx:alpine
    ports:
    - containerPort: 80
    env:
    - name: POD_NAME
      value: "cni-demo-pod-1"
    - name: POD_IP
      valueFrom:
        fieldRef:
          fieldPath: status.podIP
```

#### Pod 2: `cni-demo-pod-2`
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: cni-demo-pod-2
  namespace: networking-demo
  labels:
    app: cni-demo
    role: backend
spec:
  containers:
  - name: busybox
    image: busybox:1.35
    command: ["/bin/sh", "-c", "while true; do echo 'CNI Demo Pod 2 - $(date)'; sleep 30; done"]
```

#### Pod 3: `shared-namespace-pod`
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: shared-namespace-pod
  namespace: networking-demo
  labels:
    app: shared-namespace-demo
spec:
  containers:
  - name: nginx
    image: nginx:alpine
    ports:
    - containerPort: 80
  - name: sidecar
    image: busybox:1.35
    command: ["/bin/sh", "-c", "while true; do echo 'Sidecar container - $(date)'; sleep 60; done"]
```

### What it demonstrates:

#### CNI Functionality:
- **IP Assignment**: Each pod gets a unique IP address from the CNI plugin
- **veth Pair Creation**: CNI creates virtual ethernet pairs connecting pods to the host bridge
- **Network Interface Setup**: Pods get an `eth0` interface with assigned IP
- **Bridge Connection**: Pods are connected to the host's Linux bridge

#### Bridge Networking:
- **Same-node communication**: Pods on the same node communicate via the local bridge
- **Cross-node routing**: Pods on different nodes communicate via routing tables or overlay networks
- **Traffic flow**: Packets flow through veth → bridge → routing → destination

#### Shared Network Namespace:
- **Multi-container pods**: All containers in a pod share the same network namespace
- **Localhost communication**: Containers can communicate via `127.0.0.1`
- **Single IP per pod**: All containers in a pod share the same IP address

### Networking Concepts Demonstrated:
- **CNI Plugin Operation**: How CNI assigns IPs and creates network interfaces
- **veth Pairs**: Virtual ethernet links between host and pod
- **Linux Bridge**: How pods connect to the host network
- **Network Namespaces**: Pod-level network isolation
- **Pod-to-Pod Communication**: Direct IP-based communication

---

## 3. ClusterIP Service

**File:** `03-clusterip-service.yaml`

### What it does:
Creates ClusterIP services that provide stable internal access points for pods, enabling load balancing and service discovery.

### Key Components:

#### Service 1: `clusterip-demo`
```yaml
apiVersion: v1
kind: Service
metadata:
  name: clusterip-demo
  namespace: networking-demo
  labels:
    app: cni-demo
    service-type: clusterip
spec:
  type: ClusterIP
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
    name: http
  selector:
    app: cni-demo
```

#### Service 2: `shared-namespace-service`
```yaml
apiVersion: v1
kind: Service
metadata:
  name: shared-namespace-service
  namespace: networking-demo
  labels:
    app: shared-namespace-demo
    service-type: clusterip
spec:
  type: ClusterIP
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
    name: http
  selector:
    app: shared-namespace-demo
```

### What it demonstrates:

#### Service Abstraction:
- **Stable IP**: Service provides a consistent IP regardless of pod lifecycle
- **Load Balancing**: Traffic is distributed across multiple backend pods
- **Service Discovery**: Pods can find each other via service names

#### kube-proxy Operation:
- **iptables/IPVS Rules**: kube-proxy creates routing rules for service traffic
- **Virtual IP**: Service gets a virtual IP in the cluster's service CIDR
- **Endpoint Management**: kube-proxy tracks pod endpoints and updates routing

#### Internal Communication:
- **Pod-to-Service**: Pods communicate via service names instead of pod IPs
- **DNS Resolution**: Services are resolvable via Kubernetes DNS
- **Port Mapping**: Service ports are mapped to pod target ports

### Networking Concepts Demonstrated:
- **Service Discovery**: How pods find services via DNS
- **Load Balancing**: Traffic distribution across multiple endpoints
- **Virtual IP**: Service abstraction layer
- **kube-proxy**: How service routing works behind the scenes
- **Internal Communication**: Cluster-only access patterns

---

## 4. NodePort Service

**File:** `04-nodeport-service.yaml`

### What it does:
Creates a NodePort service that exposes the application on a static port on every node, enabling external access to the cluster.

### Key Components:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nodeport-demo
  namespace: networking-demo
  labels:
    app: cni-demo
    service-type: nodeport
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30001
    protocol: TCP
    name: http
  selector:
    app: cni-demo
```

### What it demonstrates:

#### External Access Pattern:
- **Node-level exposure**: Service is accessible on every node's IP
- **Static port**: Uses port 30001 on all nodes
- **Port mapping**: NodePort → ServicePort → TargetPort

#### Traffic Flow:
```
External Client → Node IP:30001 → kube-proxy → Service → Pod
```

#### Use Cases:
- **Development/Testing**: Quick external access for development
- **Bare Metal Clusters**: When cloud load balancers aren't available
- **Debugging**: Direct access to services for troubleshooting

### Networking Concepts Demonstrated:
- **External Access**: How to expose services outside the cluster
- **Port Mapping**: Three-level port mapping (NodePort → ServicePort → TargetPort)
- **Node-level Routing**: Traffic routing through node IPs
- **kube-proxy Integration**: How NodePort integrates with kube-proxy

---

## 5. LoadBalancer Service

**File:** `05-loadbalancer-service.yaml`

### What it does:
Creates a LoadBalancer service that integrates with cloud provider load balancers for production-grade external access.

### Key Components:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: loadbalancer-demo
  namespace: networking-demo
  labels:
    app: cni-demo
    service-type: loadbalancer
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-type: "nlb"
    service.beta.kubernetes.io/aws-load-balancer-scheme: "internet-facing"
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
    name: http
  selector:
    app: cni-demo
```

### What it demonstrates:

#### Cloud Integration:
- **Provider Integration**: Automatically provisions cloud load balancer
- **External IP**: Gets a public IP from the cloud provider
- **Health Checks**: Cloud provider handles health checking

#### Traffic Flow:
```
Internet → Cloud Load Balancer → NodePort → Service → Pod
```

#### Production Features:
- **High Availability**: Cloud provider handles failover
- **SSL Termination**: Can handle SSL/TLS termination
- **Global Distribution**: Can distribute traffic globally

### Networking Concepts Demonstrated:
- **Cloud Integration**: How Kubernetes integrates with cloud providers
- **External Load Balancing**: Production-grade external access
- **Service Layering**: LoadBalancer → NodePort → ClusterIP hierarchy
- **Cloud-specific Annotations**: Provider-specific configuration

---

## 6. Ingress Resources

**File:** `06-ingress-demo.yaml`

### What it does:
Creates Ingress resources that provide HTTP/HTTPS routing and load balancing based on hostnames and paths.

### Key Components:

#### Ingress 1: Path-based routing
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-demo
  namespace: networking-demo
  labels:
    app: ingress-demo
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  ingressClassName: nginx
  rules:
  - host: networking-demo.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: clusterip-demo
            port:
              number: 80
      - path: /shared
        pathType: Prefix
        backend:
          service:
            name: shared-namespace-service
            port:
              number: 80
```

#### Ingress 2: Host-based routing
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-demo-multi-host
  namespace: networking-demo
  labels:
    app: ingress-demo-multi-host
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  ingressClassName: nginx
  rules:
  - host: frontend.networking-demo.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: clusterip-demo
            port:
              number: 80
  - host: backend.networking-demo.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: shared-namespace-service
            port:
              number: 80
```

### What it demonstrates:

#### HTTP Routing:
- **Path-based routing**: Different paths route to different services
- **Host-based routing**: Different hostnames route to different services
- **URL rewriting**: Path rewriting for backend services

#### Ingress Controller Integration:
- **Controller dependency**: Requires an Ingress controller (nginx, haproxy, etc.)
- **SSL/TLS termination**: Can handle HTTPS termination
- **Load balancing**: HTTP-level load balancing

#### Traffic Flow:
```
HTTP Request → Ingress Controller → Service → Pod
```

### Networking Concepts Demonstrated:
- **HTTP Routing**: Application-layer routing based on URLs
- **Ingress Controller**: How ingress controllers work
- **SSL Termination**: HTTPS handling at ingress level
- **URL Rewriting**: Path manipulation for backend services

---

## 7. Network Policies

**File:** `07-network-policy.yaml`

### What it does:
Creates NetworkPolicy resources that control traffic flow between pods, implementing network security and access control.

### Key Components:

#### NetworkPolicy 1: Pod-to-pod communication
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: network-policy-demo
  namespace: networking-demo
spec:
  podSelector:
    matchLabels:
      app: cni-demo
      role: frontend
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: cni-demo
          role: backend
    ports:
    - protocol: TCP
      port: 80
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: cni-demo
          role: backend
    ports:
    - protocol: TCP
      port: 80
  - to: []
    ports:
    - protocol: UDP
      port: 53
    - protocol: TCP
      port: 443
```

#### NetworkPolicy 2: Namespace-level access
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: shared-namespace-network-policy
  namespace: networking-demo
spec:
  podSelector:
    matchLabels:
      app: shared-namespace-demo
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: networking-demo
    ports:
    - protocol: TCP
      port: 80
```

### What it demonstrates:

#### Traffic Control:
- **Ingress rules**: Control incoming traffic to pods
- **Egress rules**: Control outgoing traffic from pods
- **Port restrictions**: Limit traffic to specific ports and protocols

#### Security Patterns:
- **Zero-trust networking**: Default deny, explicit allow
- **Micro-segmentation**: Fine-grained access control
- **Namespace isolation**: Control traffic between namespaces

#### CNI Integration:
- **CNI requirement**: Requires CNI that supports NetworkPolicy (Calico, Cilium, etc.)
- **Policy enforcement**: CNI enforces policies at the network level
- **Performance impact**: Policies can affect network performance

### Networking Concepts Demonstrated:
- **Network Security**: How to secure pod-to-pod communication
- **Traffic Control**: Ingress and egress policy enforcement
- **Micro-segmentation**: Fine-grained network access control
- **CNI Integration**: How NetworkPolicy works with CNI plugins

---

## 8. Scalable Deployment

**File:** `08-deployment-demo.yaml`

### What it does:
Creates a scalable deployment with multiple replicas to demonstrate load balancing and how services handle multiple backend pods.

### Key Components:

#### Deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: scalable-app
  namespace: networking-demo
  labels:
    app: scalable-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: scalable-app
  template:
    metadata:
      labels:
        app: scalable-app
    spec:
      containers:
      - name: nginx
        image: nginx:alpine
        ports:
        - containerPort: 80
        env:
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: POD_IP
          valueFrom:
            fieldRef:
              fieldPath: status.podIP
        - name: NODE_NAME
          valueFrom:
            fieldRef:
              fieldPath: spec.nodeName
```

#### Service for the deployment
```yaml
apiVersion: v1
kind: Service
metadata:
  name: scalable-app-service
  namespace: networking-demo
  labels:
    app: scalable-app
    service-type: clusterip
spec:
  type: ClusterIP
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
    name: http
  selector:
    app: scalable-app
```

### What it demonstrates:

#### Load Balancing:
- **Multiple endpoints**: Service distributes traffic across multiple pods
- **Health checking**: Unhealthy pods are removed from endpoints
- **Session affinity**: Can maintain session affinity if configured

#### Scaling Patterns:
- **Horizontal scaling**: Multiple identical pods
- **Rolling updates**: Zero-downtime deployments
- **Resource distribution**: Pods distributed across nodes

#### Service Integration:
- **Endpoint management**: Service automatically tracks pod endpoints
- **Load distribution**: Traffic distributed across healthy pods
- **Failover**: Automatic failover when pods become unhealthy

### Networking Concepts Demonstrated:
- **Load Balancing**: How services distribute traffic across multiple pods
- **Endpoint Management**: How services track pod endpoints
- **Health Checking**: How unhealthy pods are handled
- **Scaling**: How networking adapts to scaled deployments

---

## 9. Test Client

**File:** `09-test-client.yaml`

### What it does:
Creates a test client pod that continuously tests network connectivity, service communication, and demonstrates packet flow.

### Key Components:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-client
  namespace: networking-demo
  labels:
    app: test-client
spec:
  containers:
  - name: curl-client
    image: curlimages/curl:latest
    command: ["/bin/sh", "-c"]
    args:
    - |
      while true; do
        echo "=== Testing ClusterIP Service ==="
        curl -s http://clusterip-demo/ | head -5
        
        echo "=== Testing Shared Namespace Service ==="
        curl -s http://shared-namespace-service/ | head -5
        
        echo "=== Testing Scalable App Service ==="
        curl -s http://scalable-app-service/ | head -5
        
        echo "=== Testing Direct Pod Communication ==="
        POD1_IP=$(kubectl get pod cni-demo-pod-1 -n networking-demo -o jsonpath='{.status.podIP}')
        POD2_IP=$(kubectl get pod cni-demo-pod-2 -n networking-demo -o jsonpath='{.status.podIP}')
        
        if [ ! -z "$POD1_IP" ]; then
          echo "Direct access to cni-demo-pod-1: $POD1_IP"
          curl -s --connect-timeout 5 http://$POD1_IP/ | head -3 || echo "Direct pod access failed"
        fi
        
        echo "=== Network Information ==="
        echo "My Pod IP: $(hostname -i)"
        echo "My Node: $(kubectl get pod test-client -n networking-demo -o jsonpath='{.spec.nodeName}')"
        
        echo "=== Waiting 60 seconds before next test ==="
        sleep 60
      done
```

### What it demonstrates:

#### Connectivity Testing:
- **Service communication**: Tests service-based access
- **Direct pod access**: Tests direct IP-based communication
- **DNS resolution**: Tests service name resolution
- **Load balancing**: Tests traffic distribution

#### Network Diagnostics:
- **Pod IP discovery**: Shows how to get pod IPs
- **Node information**: Shows pod-to-node mapping
- **Connectivity verification**: Tests various access patterns

#### Continuous Monitoring:
- **Periodic testing**: Continuous connectivity verification
- **Error handling**: Graceful handling of connection failures
- **Logging**: Detailed logging of test results

### Networking Concepts Demonstrated:
- **Connectivity Testing**: How to verify network connectivity
- **Service Discovery**: How to find and access services
- **Direct Communication**: Pod-to-pod direct IP communication
- **Load Balancing Verification**: How to verify load balancing works

---

## 10. DNS and Service Discovery

**File:** `10-dns-demo.yaml`

### What it does:
Creates DNS-related resources to demonstrate service discovery, DNS resolution, and external service integration.

### Key Components:

#### DNS Test Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: dns-test-service
  namespace: networking-demo
  labels:
    app: dns-demo
spec:
  type: ClusterIP
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
    name: http
  selector:
    app: cni-demo
```

#### ExternalName Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: external-service
  namespace: networking-demo
  labels:
    app: dns-demo
spec:
  type: ExternalName
  externalName: httpbin.org
  ports:
  - port: 80
    protocol: TCP
    name: http
```

#### DNS Test Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: dns-test-pod
  namespace: networking-demo
  labels:
    app: dns-demo
spec:
  containers:
  - name: dns-test
    image: busybox:1.35
    command: ["/bin/sh", "-c"]
    args:
    - |
      while true; do
        echo "=== DNS Resolution Tests ==="
        
        echo "1. Resolving ClusterIP service:"
        nslookup clusterip-demo
        nslookup clusterip-demo.networking-demo.svc.cluster.local
        
        echo "2. Resolving ExternalName service:"
        nslookup external-service
        nslookup external-service.networking-demo.svc.cluster.local
        
        echo "3. Resolving Kubernetes DNS:"
        nslookup kubernetes.default.svc.cluster.local
        
        echo "4. Testing service connectivity:"
        wget -qO- --timeout=5 http://clusterip-demo/ | head -3 || echo "Service access failed"
        
        echo "5. Testing external service:"
        wget -qO- --timeout=5 http://external-service/ | head -3 || echo "External service access failed"
        
        echo "=== Waiting 60 seconds ==="
        sleep 60
      done
```

### What it demonstrates:

#### DNS Resolution:
- **Service DNS**: How services are resolved via DNS
- **FQDN resolution**: Full qualified domain name resolution
- **Namespace resolution**: Cross-namespace service resolution

#### Service Discovery:
- **Internal services**: How to discover internal cluster services
- **External services**: How to integrate external services
- **DNS patterns**: Standard DNS naming conventions

#### External Integration:
- **ExternalName service**: DNS-based external service integration
- **External connectivity**: How pods access external services
- **DNS delegation**: How DNS queries are handled

### Networking Concepts Demonstrated:
- **Service Discovery**: How services are discovered via DNS
- **DNS Resolution**: How Kubernetes DNS works
- **External Integration**: How to integrate external services
- **DNS Patterns**: Standard Kubernetes DNS naming conventions

---

## 🎯 Summary of Networking Concepts Demonstrated

### Core Concepts:
1. **CNI Operation**: IP assignment, veth pairs, bridge networking
2. **Pod Communication**: Direct IP-based and service-based communication
3. **Service Abstraction**: Load balancing, stable IPs, service discovery
4. **External Access**: NodePort, LoadBalancer, Ingress patterns
5. **Network Security**: NetworkPolicy implementation and traffic control
6. **DNS and Discovery**: Service discovery and external integration
7. **Scaling**: How networking adapts to scaled deployments

### Traffic Flow Patterns:
- **Pod-to-Pod**: Direct IP communication
- **Pod-to-Service**: Service-based communication with load balancing
- **External-to-Service**: NodePort, LoadBalancer, Ingress access
- **Service-to-External**: ExternalName and external service integration

### Security Patterns:
- **Network Policies**: Fine-grained traffic control
- **Namespace Isolation**: Resource and network isolation
- **Service Security**: Controlled access to services

This comprehensive demo provides hands-on experience with all major Kubernetes networking concepts, from basic CNI operation to advanced patterns like Ingress and NetworkPolicy. 