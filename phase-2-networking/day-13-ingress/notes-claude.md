# Day 13 — Ingress

## The Problem Ingress Solves
- One LoadBalancer per Service = expensive in cloud (multiple public IPs)
- No path-based routing with raw Services
- Ingress = one entry point, one IP, route to many Services by URL path or hostname

## Ingress vs LoadBalancer
- LoadBalancer: one public IP per Service
- Ingress: one public IP, routes to multiple Services based on rules
- Ingress is like an Application Gateway / reverse proxy in front of all Services

## Two Parts of Ingress
1. Ingress resource — YAML routing rules (path → Service mapping)
2. Ingress Controller — the actual reverse proxy enforcing the rules (nginx, AGIC in AKS)

## Key Fields in Ingress YAML
- apiVersion: networking.k8s.io/v1
- ingressClassName: which controller handles this Ingress
- rules.host: optional — only route if Host header matches
- rules.http.paths.path: URL path to match
- pathType: Prefix (match path + anything below) or Exact (exact match only)
- backend.service.name: which Service to route to
- backend.service.port.number: which port on the Service

## pathType Values
- Prefix: /app matches /app, /app/anything, /app/nested/path
- Exact: /app matches ONLY /app — not /app/something

## host field
- Optional — filters by HTTP Host header
- host: example.com only routes requests with Host: example.com
- Remove host for local testing — requests come as Host: localhost
- In production: use host for virtual hosting (app.company.com vs api.company.com)

## What We Built
- Two Deployments: app-deployment (app: app) and api-deployment (app: api)
- Two ClusterIP Services: app-service and api-service
- One Ingress routing /app → app-service, /api → api-service
- ingressClassName: nginx

## Ingress Controller
- Must be installed separately — not included in Kubernetes by default
- We installed: ingress-nginx (official nginx Ingress Controller)
- In AKS: use AGIC (Application Gateway Ingress Controller)
- Runs in ingress-nginx namespace

## Docker Desktop Limitation
- ADDRESS field stays empty locally — normal Docker Desktop behavior
- In real cloud (AKS/EKS/GKE): ADDRESS populates with public IP automatically
- Test locally using: kubectl port-forward -n ingress-nginx service/ingress-nginx-controller 8080:80

## VS Code CRLF Fix
- Click CRLF in bottom right of VS Code → change to LF → save
- Fixes Windows line ending issue without needing cat heredoc
- Do this for every YAML file going forward

## Production Reality (AKS)
- Use AGIC (Application Gateway Ingress Controller) in AKS
- Azure Application Gateway automatically provisioned
- One public IP handles all routing rules
- TLS termination handled at Ingress level — Pods only see HTTP

## Commands Used Today
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f ingress.yaml
kubectl get pods
kubectl get ingress example-ingress
kubectl get service -n ingress-nginx
kubectl get pods -n ingress-nginx
kubectl port-forward -n ingress-nginx service/ingress-nginx-controller 8080:80

## Interview Answer: What is Ingress?
- Ingress is a Kubernetes resource that provides HTTP routing rules
- It sits in front of Services and routes traffic based on path or hostname
- Requires an Ingress Controller (nginx, AGIC) to enforce the rules
- One Ingress = one public IP serving multiple Services
- More cost-effective than one LoadBalancer per Service
