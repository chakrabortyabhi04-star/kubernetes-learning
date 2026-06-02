# Day 9 — Services: ClusterIP

## The Problem ClusterIP Solves
- Pod IPs are temporary — they change on every restart
- Hardcoding Pod IPs breaks communication
- Solution: A Service gives a stable, permanent internal endpoint
- ClusterIP = internal only (like a private corporate network)

## How ClusterIP Works
- Service gets a stable virtual IP (ClusterIP) from the cluster IP pool
- All internal traffic goes to the Service IP, never to Pod IPs directly
- Service routes traffic to healthy Pods behind it
- Pods come and go — Service IP never changes

## Key Fields in Service YAML
- selector: matches Pod labels — this is how Service finds its Pods
- port: the port the Service listens on (what callers use)
- targetPort: the port on the Pod where traffic is delivered
- These do NOT have to be the same number

## The Receptionist Analogy
- Callers dial office number (port: 80)
- Receptionist forwards to your desk extension (targetPort: 8080)

## What We Built
- nginx Deployment with label app=backend (3 replicas)
- ClusterIP Service selecting app=backend
- Tested connectivity from busybox test Pod using wget
- Service IP: 10.96.49.9 — stable, never changes

## Break → Debug → Fix
### The Break
- Patched Service selector to app=wrong-label
- Result: Connection refused from inside cluster

### How to Debug Service Issues
1. kubectl describe service <name>
2. Check Selector field — does it match Pod labels?
3. Check Endpoints field — empty means no Pods matched
4. kubectl get pods --show-labels — verify Pod labels

### The Fix
- Patch selector back to correct label
- Endpoints immediately repopulate with Pod IPs

## Production Gotcha
- Selector mismatch is the #1 cause of Service connectivity failures
- Empty Endpoints = Service has no Pods to route to
- Always verify: Service selector == Pod labels

## Commands Used Today
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl get pods
kubectl get pods -o wide
kubectl get service nginx-service
kubectl describe service nginx-service
kubectl patch service nginx-service -p '{"spec":{"selector":{"app":"backend"}}}'
kubectl run test-pod --image=busybox --rm -it --restart=Never -- sh
wget -qO- http://<ClusterIP>

## Interview Answer: port vs targetPort
- port: the port the Service listens on (what other Pods dial)
- targetPort: the port on the Pod container where traffic lands
- Example: port 80 → targetPort 8080 (app runs on 8080 internally)
