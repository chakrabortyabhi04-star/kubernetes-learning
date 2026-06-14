# Day 14 — Network Policies

## The Problem Network Policies Solve
- By default all Pods can talk to all other Pods — no restrictions
- If attacker compromises frontend, they can reach database directly
- Network Policies = Kubernetes firewall rules for Pods
- Think: VLANs and firewall rules from IT Ops, but for Pods

## Key Concept: Whitelist Model
- No NetworkPolicy on a Pod = allow ALL traffic (default open)
- Any NetworkPolicy targeting a Pod = DENY ALL except what's explicitly allowed
- It's a whitelist, not a blacklist — this is critical

## Two Policy Types
- Ingress: controls incoming traffic TO the Pod
- Egress: controls outgoing traffic FROM the Pod
- Same as firewall inbound/outbound rules

## Key Fields in NetworkPolicy YAML
- podSelector: which Pods this policy applies to (uses labels)
- policyTypes: Ingress, Egress, or both
- ingress.from.podSelector: which Pods are allowed to send traffic in
- egress.to.podSelector: which Pods this Pod is allowed to send traffic to
- ports: which port/protocol is allowed

## Labels vs Namespaces in Network Policies
- Labels: identify specific Pods ("app=database", "role=backend")
- Namespace: identify environment/team boundary ("production", "staging")
- Both can be combined: "allow from Pods labeled app=frontend in namespace=production"

## What We Built
- Three Pods: frontend-pod, backend-pod, database-pod
- NetworkPolicy targeting database Pod (app=database)
- Only backend Pod (app=backend) allowed to reach database on port 80
- Frontend Pod is blocked from reaching database

## Docker Desktop Limitation
- kindnet (Docker Desktop CNI) has limited Network Policy enforcement
- In production: use Calico, Cilium, or Azure NPM for full enforcement
- AKS uses Azure NPM which fully enforces Network Policies
- The YAML and concept are correct — enforcement is a CNI concern

## Production Use Cases
- Database isolation: only app Pods can reach DB, not other services
- Namespace isolation: production Pods can't talk to staging Pods
- Egress control: prevent Pods from making outbound internet calls
- Zero trust: deny all by default, explicitly allow needed paths

## Debug Checklist for Network Policy Issues
1. kubectl describe networkpolicy <name> — verify rules are correct
2. Check podSelector matches target Pod labels exactly
3. Check ingress.from.podSelector matches source Pod labels
4. Verify CNI plugin supports Network Policies (kindnet has limits)
5. kubectl get networkpolicy — list all policies in namespace

## Commands Used Today
kubectl apply -f pod.yaml
kubectl apply -f network-policy.yaml
kubectl describe networkpolicy db-network-policy
kubectl get pod database-pod -o wide
kubectl exec -it backend-pod -- sh

## Interview Answer: What are Network Policies?
- Kubernetes firewall rules for Pod-to-Pod communication
- Use label selectors to target Pods and define allowed traffic
- Whitelist model: once a policy targets a Pod, all other traffic is denied
- Require a CNI plugin that supports enforcement (Calico, Cilium, Azure NPM)
- Two types: Ingress (incoming) and Egress (outgoing)
