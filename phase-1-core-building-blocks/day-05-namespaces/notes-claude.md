# Day 5: Namespaces

## What is a Namespace?
A virtual cluster inside your real cluster. Provides isolation between
teams, projects, or environments sharing the same Kubernetes cluster.

## Default Namespaces
| Namespace | Purpose |
|---|---|
| default | Where your resources go when no namespace specified |
| kube-system | Kubernetes' own brain — never touch this |
| kube-public | Publicly readable cluster info |
| kube-node-lease | Node heartbeat tracking |

## Key Commands
kubectl get namespaces                          # list all namespaces
kubectl create namespace <name>                 # create namespace
kubectl get pods -n <namespace>                 # pods in specific namespace
kubectl get pods --all-namespaces               # pods in ALL namespaces
kubectl get pods -A                             # same as above (shortcut)
kubectl config set-context --current --namespace=<name>  # switch default namespace

## The Isolation Rule
kubectl delete all --all  → only affects CURRENT namespace
kube-system is NOT auto-protected — namespaces provide the protection

## Production Use Cases
- dev / staging / prod namespaces in same cluster
- team-based isolation (frontend, backend, data-science)
- Each team can only see and manage their own namespace (via RBAC)

## Senior DevOps Tips
- Always specify -n in scripts — never rely on default context
- Use -A to get full cluster picture during debugging
- Never deploy apps into kube-system
- Use RBAC (Day 31) to restrict teams to their own namespaces