cat > ~/OneDrive/Desktop/kubernetes-learning/phase-1-core-building-blocks/day-08-review/notes-claude.md << 'EOF'
# Day 8: Phase 1 Review + Debug Lab

## Phase 1 Summary — What You Now Know

### The Kubernetes Object Hierarchy
Pod → managed by → ReplicaSet → managed by → Deployment

### Quick Reference: When to Use What
| Situation | Object |
|---|---|
| Run a container | Pod (inside a Deployment) |
| Ensure N replicas always run | Deployment |
| Isolate teams/environments | Namespace |
| Control resource usage | Requests + Limits |
| Zero-downtime updates | RollingUpdate strategy |
| Instant rollback | kubectl rollout undo |
| Find specific Pods | Labels + Selectors |

---

## Debug Scenarios Covered

### Scenario 1: ImagePullBackOff / ErrImagePull
**Cause:** Wrong image name or tag — Docker Hub can't find it
**Debug:**
kubectl describe pod <name>  → check Events for "failed to pull image"
**Fix:**
kubectl set image deployment/<name> <container>=<correct-image>:<tag>

### Scenario 2: Label Mismatch
**Cause:** selector.matchLabels != template.metadata.labels
**Error:** "selector does not match template labels"
**Fix:** Make both labels identical — the Golden Rule

### Scenario 3: OOMKilled / Memory Issues
**Cause 1:** requests > limits → Kubernetes rejects immediately
**Cause 2:** Container uses more RAM than limit → OOMKilled → CrashLoopBackOff
**Fix:** Set requests < limits, and size limits appropriately

---

## Production-Ready Deployment Checklist
- [ ] Pinned image version (never latest)
- [ ] Meaningful labels (app: payment-service, not app: nginx)
- [ ] selector.matchLabels matches template.metadata.labels
- [ ] Resource requests AND limits set
- [ ] CPU limit > CPU request (allows bursting)
- [ ] Memory limit >= memory request
- [ ] Proper naming (lowercase, hyphens only)
- [ ] RollingUpdate strategy defined

---

## Interview Answers

### Pod vs Deployment
"A Pod is the smallest execution unit. A Deployment is a controller
ensuring replicas always run with self-healing, scaling, and rolling
updates. Always use Deployments in production — standalone Pods are
only for quick debugging."

### Debugging CrashLoopBackOff
1. kubectl get pods                  → spot the problem
2. kubectl describe pod <name>       → read Events section
3. kubectl logs <name>               → current container logs
4. kubectl logs <name> --previous    → logs from crashed container ← KEY
5. Fix → kubectl apply or kubectl set image

### Requests vs Limits
"Requests = minimum guaranteed (Scheduler uses for placement).
Limits = hard ceiling (RAM exceeded = OOMKilled, CPU exceeded = throttled).
Without them in production: Noisy Neighbor problem — one greedy app
starves others, Scheduler can't make smart decisions, cluster-wide failures."

---

## Key kubectl Commands — Phase 1 Master List
kubectl get pods                              # list pods
kubectl get pods -A                           # all namespaces
kubectl get pods -l app=payment-service       # filter by label
kubectl get pods -o wide                      # with IP and Node
kubectl describe pod <name>                   # full details + events
kubectl logs <name>                           # container logs
kubectl logs <name> --previous                # crashed container logs
kubectl apply -f <file>                       # create/update resource
kubectl delete pod <name>                     # delete pod
kubectl set image deployment/<n> <c>=<image>  # update image
kubectl rollout undo deployment/<name>        # rollback
kubectl rollout history deployment/<name>     # revision history
kubectl get namespaces                        # list namespaces
kubectl get replicasets                       # list replicasets
kubectl config set-context --current --namespace=<n>  # switch namespace
EOF