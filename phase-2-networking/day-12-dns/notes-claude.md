# Day 12 — DNS in Kubernetes

## Why DNS Matters
- Pods get new IPs every restart — hardcoding IPs breaks communication
- Every Service gets an automatic DNS name inside the cluster
- Apps call Service by name — DNS resolves to ClusterIP — no IP hardcoding needed

## Full DNS Format
<service-name>.<namespace>.svc.cluster.local

Example: nginx-service.default.svc.cluster.local → 10.96.49.9

## DNS Name Breakdown
- service-name: the Service metadata.name
- namespace: the namespace the Service lives in
- svc: indicates this is a Service resource
- cluster.local: the cluster's internal domain name

## DNS Shortcut Rules
| Caller location     | Target location     | Minimum DNS name                          |
|---------------------|---------------------|-------------------------------------------|
| same namespace      | same namespace      | nginx-service                             |
| different namespace | backend namespace   | nginx-service.backend                     |
| anywhere            | anywhere            | nginx-service.backend.svc.cluster.local   |

## CoreDNS
- Internal DNS server for every Kubernetes cluster
- Runs as a Deployment in kube-system namespace (2 replicas for HA)
- CoreDNS Service IP: 10.96.0.10 (all DNS queries go here first)
- Every Pod is automatically configured to use CoreDNS for DNS resolution

## What We Proved
- nslookup nginx-service → resolved to 10.96.49.9 (ClusterIP)
- wget -qO- http://nginx-service → got nginx response using name only
- Full FQDN also works: nginx-service.default.svc.cluster.local

## Break → Debug → Fix
### The Break
- Scaled CoreDNS to 0 replicas
- nslookup nginx-service → Connection refused, no servers could be reached
- Service and Pods were perfectly healthy — DNS was the only failure

### Key Insight
- DNS failure = app failure even when everything else is healthy
- CoreDNS is a critical cluster component — treat it like a core service
- Always check CoreDNS Pods when apps can't find each other by name

### The Fix
- Scaled CoreDNS back to 2 replicas
- DNS resolution restored immediately

## Production Debug Checklist for DNS Issues
1. kubectl get pods -n kube-system → check CoreDNS is Running
2. kubectl run dns-test --image=busybox --rm -it --restart=Never -- sh
3. nslookup <service-name> → verify DNS resolves
4. Check Service exists in correct namespace
5. Verify caller and target namespaces — use FQDN for cross-namespace

## Control Plane Components (visible in kube-system)
- coredns: internal DNS server
- kube-apiserver: cluster API — all kubectl commands go here
- kube-controller-manager: manages Deployments, ReplicaSets
- kube-scheduler: decides which Node a Pod runs on
- kube-proxy: manages iptables rules for Services
- etcd: cluster database — stores all cluster state

## Commands Used Today
kubectl get pods -n kube-system
kubectl run dns-test --image=busybox --rm -it --restart=Never -- sh
nslookup nginx-service
nslookup nginx-service.default.svc.cluster.local
wget -qO- http://nginx-service
kubectl scale deployment coredns -n kube-system --replicas=0
kubectl scale deployment coredns -n kube-system --replicas=2

## Interview Answer: How does DNS work in Kubernetes?
- Every Service gets an automatic DNS name: <name>.<namespace>.svc.cluster.local
- CoreDNS runs in kube-system and resolves all internal DNS queries
- Short name works within same namespace, FQDN works everywhere
- Pods never need to know Service IPs — just the Service name
