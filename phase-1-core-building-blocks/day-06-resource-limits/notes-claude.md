
# Day 6: Resource Limits & Requests

## The Problem (Noisy Neighbor)
Without resource limits, one greedy app can consume all CPU/RAM
on a Node — starving other apps. This is called the "Noisy Neighbor" problem.

## Two Tools Kubernetes Gives You
| Tool | Purpose |
|---|---|
| requests | Minimum guaranteed resources — used by Scheduler for placement |
| limits | Maximum resources — container gets killed/throttled if exceeded |

## Analogy
- Request = Reserved parking spot (guaranteed, always yours)
- Limit = Speed limit (you cannot exceed this, ever)

## What Happens When Limits Are Exceeded
| Resource | Exceeds Limit |
|---|---|
| Memory (RAM) | Container gets KILLED → OOMKilled → CrashLoopBackOff |
| CPU | Container gets THROTTLED (slowed down, not killed) |

## Units
CPU:
- 1000m = 1 full CPU core
- 500m = half a CPU core
- 250m = quarter of a CPU core

Memory:
- Mi = Mebibytes (~megabytes)
- Gi = Gibibytes (~gigabytes)
- 64Mi = ~64MB RAM
- 128Mi = ~128MB RAM

## YAML Structure
```yaml
containers:
  - name: nginx-container
    image: nginx
    resources:
      requests:
        cpu: "250m"
        memory: "64Mi"
      limits:
        cpu: "500m"
        memory: "128Mi"
```

## How Scheduler Uses Requests
Scheduler checks each Node's available resources against Pod requests:
- Node has 64Mi free + Pod requests 64Mi → Pod gets scheduled ✅
- Node has 32Mi free + Pod requests 64Mi → Pod stays Pending ❌

## Works Everywhere
Same YAML works on:
- Docker Desktop (learning)
- EKS (AWS)
- AKS (Azure)
- GKE (Google)
Any Kubernetes cluster!

## Key Commands
kubectl describe pod <name> | grep -A 5 "Limits\|Requests"  # verify resources
kubectl top pods                                               # see live usage
kubectl top nodes                                             # see node usage

## Production Tips
- ALWAYS set requests and limits in production
- Request = what you need to START
- Limit = maximum you'll EVER need
- Never set limits lower than requests (Kubernetes will reject it)
- Start conservative, tune based on actual usage with kubectl top
