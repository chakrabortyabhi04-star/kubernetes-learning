# Day 1: Pods

## What is a Pod?
The smallest deployable unit in Kubernetes. A Pod runs one or more
containers that share the same network (IP address) and storage (volumes).
Every Pod runs on a Node (worker machine).

## Key Facts
- One IP address per Pod — containers inside share it
- Containers inside same Pod must use different ports
- Most Pods have just ONE container (multi-container is the exception)
- Pods are mortal — they can die and be recreated

## The 4 Top-Level YAML Fields (Every K8s Object)
apiVersion: v1        # which API version
kind: Pod             # what object type
metadata:             # info about the object (name, labels)
  name: my-first-pod
spec:                 # what the object should do
  containers:
    - name: nginx-container
      image: nginx
      ports:
        - containerPort: 80

## YAML Rules
- Use 2 spaces for indentation (NEVER tabs)
- Field names are camelCase (containerPort, apiVersion)
- kind values are PascalCase (Pod, Deployment, Service)
- Plural fields (containers, ports) = lists → use dash (-)
- Wrong capitalization = error (pod ≠ Pod)

## Essential kubectl Commands (Day 1)
kubectl apply -f pod.yaml          # create/update Pod from YAML
kubectl get pods                   # list all Pods (summary)
kubectl describe pod <name>        # full details + events
kubectl logs <pod-name>            # container logs
kubectl delete pod <name>          # delete a Pod

## Pod Lifecycle (What Happens When You Apply)
1. Scheduler → picks which Node to place Pod on
2. kubelet → pulls container image from Docker Hub
3. kubelet → creates the container
4. kubelet → starts the container
5. Pod status → Running ✅

## Common Pod Errors
| Error | Cause | Fix |
|---|---|---|
| InvalidImageName | Image name badly formatted | Fix image name in YAML |
| ImagePullBackOff | Image not found / wrong tag | Check image name + tag |
| CrashLoopBackOff | Container keeps cras