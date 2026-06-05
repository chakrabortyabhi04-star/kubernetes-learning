# Day 10 — Services: NodePort

## The Problem NodePort Solves
- ClusterIP is internal only — unreachable from your laptop or browser
- NodePort opens a port on the Node itself for external access
- Think: opening a port on your firewall to reach an internal server

## Traffic Flow
Your Browser → Node IP : nodePort → Service port → targetPort on Pod

## Three Port Fields
- nodePort: port opened on the Node (your machine) — what browser dials
- port: port the Service listens on internally inside the cluster
- targetPort: port on the Pod container where traffic finally lands

## NodePort Range
- Valid range: 30000–32767
- Kubernetes enforces this at API level — hard rejection if outside range
- Can be assigned manually or auto-assigned by Kubernetes

## What We Built
- nginx Deployment with label app=frontend (2 replicas)
- NodePort Service selecting app=frontend
- nodePort: 30008, port: 80, targetPort: 80
- Verified Endpoints populated: both Pod IPs registered

## Key Difference from ClusterIP
- ClusterIP: internal only, no external access
- NodePort: opens external access via Node IP + port
- NodePort also gets a ClusterIP automatically — internal access still works

## Break → Debug → Fix
### The Break
- Tried to patch nodePort to 999
- Result: immediate API rejection — invalid port range

### What This Means in Production
- Kubernetes validates port range at API level
- No silent failures — hard rejection with clear error message
- Always use 30000–32767 for nodePort values

## Production Reality
- NodePort is rarely used in production cloud environments
- In cloud (AKS, EKS, GKE): use LoadBalancer or Ingress instead
- NodePort is useful for: local development, bare metal clusters, quick testing
- Docker Desktop on Windows: NodePort to localhost may not work directly
  use kubectl port-forward as workaround

## Commands Used Today
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl get pods --show-labels
kubectl get service my-nodeport-service
kubectl describe service my-nodeport-service
kubectl patch service my-nodeport-service -p '{"spec":{"ports":[{"port":80,"targetPort":80,"nodePort":999}]}}'

## Interview Answer: ClusterIP vs NodePort
- ClusterIP: stable internal IP, Pod-to-Pod communication only
- NodePort: exposes Service on Node IP + port, allows external access
- NodePort range: 30000–32767, enforced by Kubernetes API
