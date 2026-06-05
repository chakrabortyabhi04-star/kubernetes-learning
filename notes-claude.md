# Day 11 — Services: LoadBalancer

## The Problem LoadBalancer Solves
- NodePort exposes a raw Node IP + weird port (e.g. 30405) — not production-friendly
- Node IP can change if the VM is replaced
- LoadBalancer gives a stable, dedicated public IP managed by the cloud provider
- Users just call a normal IP on port 80/443 — they never see NodePort

## How LoadBalancer Works
- You declare type: LoadBalancer in Service YAML
- Cloud provider (Azure/AWS/GCP) automatically provisions a real load balancer
- A stable public IP is assigned (LoadBalancer Ingress field)
- Traffic flow: User → Cloud LB public IP → Node:NodePort (auto-assigned) → Service → Pod

## LoadBalancer builds on top of NodePort
- LoadBalancer does NOT bypass NodePort — it uses it internally
- Kubernetes auto-assigns a NodePort (e.g. 30405) — you don't specify it
- The cloud LB forwards to that NodePort on the Node
- User never sees or dials the NodePort directly

## What Changes in YAML
- Only one field changes from NodePort: type: LoadBalancer
- No nodePort field needed — Kubernetes assigns it automatically

## Key Fields in kubectl describe service
- LoadBalancer Ingress: stable public IP assigned by cloud provider
- NodePort: auto-assigned internal port (e.g. 30405)
- Endpoints: Pod IPs registered behind the Service

## What We Built
- nginx Deployment with label app: web (2 replicas)
- LoadBalancer Service selecting app: web, port 80
- Docker Desktop assigned EXTERNAL-IP: 172.18.0.5
- Auto-assigned NodePort: 30405

## Break → Debug → Fix
### The Break
- Scaled Deployment to 0 replicas
- LoadBalancer Ingress IP stayed — public IP never disappears
- Endpoints went empty — no Pods to receive traffic
- Result: traffic accepted by LB, dropped at Service layer

### Key Insight
- LoadBalancer failure is almost never at the LB layer
- Root cause is almost always at the Pod layer (crash, OOM, wrong selector)
- Always check Endpoints first when LoadBalancer stops serving traffic

### The Fix
- Scaled back to 2 replicas
- Waited for Pods to reach Running state
- Endpoints repopulated automatically

## Production Reality (AKS)
- AKS automatically creates Azure Public Load Balancer + frontend IP + backend pool + LB rules
- All from a single kubectl apply — no Azure portal needed
- LoadBalancer is the standard for production external exposure in cloud clusters
- For multiple services: use Ingress instead (Day 13) — one LB for all services

## Service Type Summary
- ClusterIP: internal only, Pod-to-Pod
- NodePort: Node IP + port, basic external access
- LoadBalancer: stable public IP, production external access, builds on NodePort

## Kubernetes Resource Naming Rule
- Resource names must be lowercase only
- Capital letters cause: metadata.name Invalid value error
- Always use kebab-case: my-service, nginx-deployment, web-loadbalancer

## Commands Used Today
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl get service loadbalancer
kubectl describe service loadbalancer
kubectl scale deployment loadbalancer --replicas=0
kubectl scale deployment loadbalancer --replicas=2

## Interview Answer: ClusterIP vs NodePort vs LoadBalancer
- ClusterIP: stable internal IP, Pod-to-Pod only
- NodePort: opens port on Node (30000-32767), basic external access
- LoadBalancer: cloud-provisioned public IP, production external access
- LoadBalancer internally uses NodePort — it builds on top of it
