# ArgoCD GitOps Demo 🚀

A realistic ArgoCD demo with two microservices deployed via GitOps.

## Repo Structure

```
apps/
├── payments-api/
│   ├── deployment.yaml     # 2 replicas, nginx:1.25-alpine
│   ├── service.yaml        # ClusterIP on port 80
│   └── configmap.yaml      # DB config, feature flags
└── orders-api/
    ├── deployment.yaml     # 2 replicas, nginx:1.25-alpine
    ├── service.yaml        # ClusterIP on port 80
    └── configmap.yaml      # DB config, order limits
```

## Demo Walkthroughs

### Demo 1: OutOfSync (manual kubectl edit causes drift)
```bash
# 1. Verify app is Synced
kubectl get applications -n argocd

# 2. Someone manually changes the deployment (bypasses Git!)
kubectl set image deployment/payments-api api=nginx:latest -n practice

# 3. ArgoCD detects drift → OutOfSync
kubectl get applications -n argocd
```

### Demo 2: Self-Heal (ArgoCD auto-reverts drift)
```bash
# Enable self-heal on the payments app
kubectl patch application payments-app -n argocd --type merge \
  -p '{"spec":{"syncPolicy":{"automated":{"selfHeal":true}}}}'

# Now try to drift again — ArgoCD will revert it automatically!
kubectl set image deployment/payments-api api=nginx:latest -n practice
kubectl get pods -w  # Watch ArgoCD revert in real-time
```

### Demo 3: Degraded (bad image in Git)
```bash
# 1. Change the deployment image to a nonexistent image in Git
#    Edit apps/payments-api/deployment.yaml:
#    image: nginx:1.25-alpine → image: nginx:99.99.99
# 2. git add, commit, push
# 3. ArgoCD syncs → pods fail → Synced but Degraded
kubectl get applications -n argocd
```

### Demo 4: Rollback
```bash
# Undo the bad image change in Git
#    image: nginx:99.99.99 → image: nginx:1.25-alpine
# git add, commit, push
# ArgoCD auto-syncs → Healthy again
```
