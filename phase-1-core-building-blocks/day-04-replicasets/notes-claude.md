# Day 4: ReplicaSets

## What is a ReplicaSet?
A ReplicaSet ensures a specified number of identical Pod replicas
are running at all times. It's the worker behind a Deployment.

## The Hierarchy
You → Deployment → ReplicaSet → Pods
- Deployment = manager (handles updates, rollbacks, strategy)
- ReplicaSet = worker (keeps X identical Pods running)
- Pod = actual running container

## Why ReplicaSets Exist as Separate Objects
- Each Deployment update creates a NEW ReplicaSet
- Old ReplicaSets are kept with 0 Pods (for rollback)
- Rollback = scale old RS up, scale new RS down → instant!

## Key Commands
kubectl get replicasets                              # list all ReplicaSets
kubectl describe replicaset <name>                   # full details
kubectl rollout history deployment <name>            # see revision history
kubectl rollout undo deployment <name>               # rollback one version
kubectl rollout undo deployment <name> --to-revision=1  # rollback to specific version
kubectl annotate deployment <name> kubernetes.io/change-cause="reason"  # document change

## Production Rules
- NEVER create standalone ReplicaSets — always use Deployments
- ReplicaSets are for debugging and rollback understanding
- Always annotate changes with change-cause for team visibility

## Debugging Path
kubectl describe deployment → kubectl describe replicaset → kubectl describe pod

## Reading kubectl get replicasets
NAME                         DESIRED   CURRENT   READY   AGE
nginx-deployment-8544bcb4bd    3         3         3      26h  ← active
nginx-deployment-87b5595c9     0         0         0      2d   ← kept for rollback

- DESIRED = how many Pods should exist
- CURRENT = how many exist right now
- READY = how many are healthy