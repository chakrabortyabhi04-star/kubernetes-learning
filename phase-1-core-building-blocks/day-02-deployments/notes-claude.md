# Day 2: Deployments

## What is a Deployment?
A Deployment manages Pods automatically. It ensures your desired number
of Pod replicas are always running. If a Pod dies, the Deployment
creates a new one automatically (self-healing).

## Key Concepts
- **Desired state**: You declare "I want 3 replicas" — Kubernetes
  makes it happen and keeps it that way forever
- **Self-healing**: Pod dies → Deployment detects it → creates new Pod
- **ReplicaSet**: The intermediate object Deployment uses to manage Pods
- **Rollback**: Old ReplicaSets are kept so you can roll back to
  previous versions

## The Hierarchy
Deployment → ReplicaSet → Pods
- Delete a Pod → Deployment recreates it ✅
- Delete the Deployment → everything dies 💀

## Pod Naming Convention
nginx-deployment - 87b5595c9 - 4b8nt
      ↑                ↑          ↑
 Deployment name  ReplicaSet ID  Pod ID

## Deployment YAML Structure
apiVersion: apps/v1        # apps group (not core v1)
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx-deployment
spec:
  replicas: 3              # desired Pod count
  selector:
    matchLabels:
      app: nginx-deployment  # must match template labels
  template:                # Pod blueprint starts here
    metadata:
      labels:
        app: nginx-deployment  # must match selector
    spec:
      containers:
        - name: nginx-container
          image: nginx
          ports:
            - containerPort: 80

## The Golden Rule
selector.matchLabels MUST match template.metadata.labels
If they don't match → Deployment can't find its Pods → broken

## Key kubectl Commands (Day 2)
kubectl apply -f deployment.yaml     # create/update Deployment
kubectl get deployments              # list Deployments
kubectl get pods                     # see all Pods
kubectl delete pod <name>            # delete one Pod (gets recreated!)
kubectl delete deployment <name>     # delete everything

## Reading kubectl get deployments
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           2m

- READY 3/3 = 3 desired, 3 running
- UP-TO-DATE = Pods running latest template version
- AVAILABLE = Pods ready to serve traffic

## Why apiVersion: apps/v1?
Deployments belong to the "apps" API group — added after core Kubernetes.
Pods use just "v1" (core group). Deployments use "apps/v1".

## Production Tips
- Always use Deployments, never standalone Pods in production
- Pin image versions: image: nginx:1.25.3 (not just nginx)
- Set replicas based on traffic needs + node capacity
- Labels are how Kubernetes connects objects — never skip them