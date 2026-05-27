cat > ~/OneDrive/Desktop/kubernetes-learning/phase-1-core-building-blocks/day-07-rolling-updates/notes-claude.md << 'EOF'
# Day 7: Rolling Updates & Rollbacks

## Two Update Strategies
| Strategy | How | Downtime? |
|---|---|---|
| Recreate | Kill ALL → start new | ❌ Yes |
| RollingUpdate | Replace one at a time | ✅ Zero |

RollingUpdate is the DEFAULT in Kubernetes.

## Key Settings
```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 1  # max Pods DOWN during update
    maxSurge: 1        # max EXTRA Pods during update
```

## With replicas: 3, maxUnavailable: 1, maxSurge: 1
- Minimum Pods always running = 2 (3-1)
- Maximum Pods at peak = 4 (3+1)
- Zero downtime guaranteed ✅

## The Rolling Update Flow
Start:  v1 v1 v1
Step 1: v1 v1 v1 v2  (surge)
Step 2: v1 v1 v2     (old killed)
Step 3: v1 v1 v2 v2  (surge)
Step 4: v1 v2 v2     (old killed)
Step 5: v1 v2 v2 v2  (surge)
Done:   v2 v2 v2     ✅

## Key Commands
# Trigger rolling update
kubectl set image deployment/<name> <container>=<image>:<tag>

# Watch rollout progress
kubectl rollout status deployment/<name>

# View rollout history
kubectl rollout history deployment/<name>

# Rollback to previous version
kubectl rollout undo deployment/<name>

# Rollback to specific revision
kubectl rollout undo deployment/<name> --to-revision=1

# Check which image is running
kubectl describe pod <name> | grep Image

## Important Facts
- Each update creates a NEW ReplicaSet
- Old ReplicaSets kept for rollback (with 0 Pods)
- Different Pods = Different IPs (containers INSIDE same Pod share IP)
- Rollback is instant — old ReplicaSet already exists, no rebuild needed

## Production Tips
- Always use RollingUpdate (never Recreate for production)
- Set maxUnavailable based on minimum capacity you can handle
- Always annotate changes: kubectl annotate deployment <name> kubernetes.io/change-cause="reason"
- Monitor rollout with: kubectl rollout status deployment/<name>
EOF