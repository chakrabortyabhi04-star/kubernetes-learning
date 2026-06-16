# Day 15 — Phase 2 Review + Debug Lab

## Phase 2 Complete — Networking Summary

### Service Types — When to Use What
- ClusterIP: internal Pod-to-Pod communication only
- NodePort: external access on bare metal or local dev (30000-32767)
- LoadBalancer: external access in cloud, single Service, stable public IP
- Ingress: external access in cloud, multiple Services, path/host based routing

### Decision Framework
Who needs access?
- Internal Pods only → ClusterIP
- External, local/bare metal → NodePort
- External, cloud, single Service → LoadBalancer
- External, cloud, multiple Services → Ingress

### DNS Format
<service-name>.<namespace>.svc.cluster.local
- Same namespace: just use service name
- Cross namespace: service-name.namespace
- CoreDNS runs in kube-system, 2 replicas for HA

### Network Policies
- Default: all Pods can talk to all Pods
- Any policy on a Pod = deny all except explicitly allowed (whitelist)
- Use label selectors to target Pods
- Requires CNI support (Calico, Cilium, Azure NPM)

## Debug Lab — 3 Scenarios

### Scenario 1: Connection Refused to ClusterIP Service
- Symptom: app can't connect to backend-service, connection refused
- Root cause: Service selector app=backend-api didn't match Pod labels app=backend
- Debug: kubectl describe service → check Endpoints (empty = selector mismatch)
- Fix: update Service selector to match Pod labels
- Lesson: selector mismatch = empty Endpoints = connection refused

### Scenario 2: Ingress Returns 404
- Symptom: Ingress exists, Pods running, but all requests return 404
- Root cause: ingressClassName: traefik but only nginx controller installed
- Debug: kubectl get pods -n ingress-nginx → check which controller is running
- Fix: change ingressClassName to match installed controller (nginx)
- Lesson: ingressClassName must match the installed Ingress Controller exactly

### Scenario 3: DNS Resolution Timing Out
- Symptom: nslookup times out, Pods and Services all Running
- Root cause: CoreDNS Pods down (CrashLoopBackOff or Pending)
- Debug: kubectl get pods -n kube-system → check CoreDNS status
- Fix: kubectl rollout restart deployment coredns -n kube-system
- Lesson: DNS failure = app failure even when everything else is healthy

## Production Rule — Never Patch, Always Fix YAML
- kubectl patch is for debugging only
- Always fix the YAML file and commit to Git
- Git is the source of truth — not the cluster (GitOps principle)
- Changes via CI/CD pipeline, reviewed via Pull Request

## Master Debug Checklist for Phase 2
1. kubectl get pods → are Pods Running?
2. kubectl describe service → check Selector and Endpoints
3. kubectl get pods -n kube-system → is CoreDNS Running?
4. kubectl describe ingress → check ingressClassName and rules
5. kubectl describe networkpolicy → check podSelector and from/to rules

## Phase 2 Commands Reference
kubectl get service <name>
kubectl describe service <name>
kubectl get ingress <name>
kubectl describe ingress <name>
kubectl get networkpolicy
kubectl describe networkpolicy <name>
kubectl get pods -n kube-system
kubectl get pods -n ingress-nginx
kubectl run dns-test --image=busybox --rm -it --restart=Never -- sh
nslookup <service-name>
nslookup <service-name>.<namespace>.svc.cluster.local
