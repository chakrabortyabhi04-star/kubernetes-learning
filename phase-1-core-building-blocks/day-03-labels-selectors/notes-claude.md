# Day 3: Labels & Selectors

## What Are Labels?
Labels are key-value pairs attached to any Kubernetes object.
They are the primary way Kubernetes objects find and connect to each other.

## Syntax
```yaml
labels:
  app: nginx-deployment
  env: dev
  team: backend
```

## Three Places Labels Appear in a Deployment
1. `metadata.labels` → tags the Deployment itself
2. `spec.selector.matchLabels` → how Deployment finds its Pods
3. `spec.template.metadata.labels` → tags the Pods that get created

## The Golden Rule
selector.matchLabels MUST equal template.metadata.labels
If they don't match → Deployment can't find its Pods → broken ❌

## Real-World Label Set (Production Standard)
```yaml
labels:
  app: web-app        # which service/app
  env: production     # which environment
  team: backend       # which team owns it
```

## Filtering with kubectl
```bash
# Single label filter
kubectl get pods -l env=dev

# Multiple label filter (AND condition)
kubectl get pods -l team=backend,env=dev

# Filter any object type
kubectl get deployments -l app=nginx-deployment
```

## Key Insights
- Labels are completely flexible — put anything you want
- `No resources found` is NOT an error — just no match
- Labels work on ALL Kubernetes objects (Pods, Deployments, Services)
- Services use labels to find which Pods to send traffic to (Day 4!)
- The `-l` flag means "filter by label"

## What's Coming Next
Services use label selectors to route traffic to the right Pods.
Without labels, Services wouldn't know which Pods to send requests to.