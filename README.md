# ArgoCD + GitOps Home Assignment - Submission

**Student:** Itamar Davideliyahu  
**Date:** 22/02/2026  
**Repository:** https://github.com/itamardavideliyahu-beep/Helms.git

---

## Screenshots

### ArgoCD UI - Both Applications (Healthy & Synced)

Both `myapp-dev` and `myapp-prod` are running with status **Healthy** and **Synced**.

| App | Branch | Namespace | Status |
|-----|--------|-----------|--------|
| myapp-dev | dev | dev | Healthy / Synced |
| myapp-prod | main | prod | Healthy / Synced |

### Dev Namespace - 2 Pods Running

```
NAME                                    READY   STATUS    RESTARTS   AGE
myapp-dev-myapp-chart-d7d579858-hx5dk   1/1     Running   0          12s
myapp-dev-myapp-chart-d7d579858-mgtv8   1/1     Running   0          10m
```

### Prod Namespace - 3 Pods Running

```
NAME                                      READY   STATUS    RESTARTS   AGE
myapp-prod-myapp-chart-8467768bd8-klvr8   1/1     Running   0          13m
myapp-prod-myapp-chart-8467768bd8-qsdsg   1/1     Running   0          13m
myapp-prod-myapp-chart-8467768bd8-zsb5l   1/1     Running   0          13m
```

---

## Explanation

### 1. How ArgoCD Connects to GitHub

ArgoCD connects to GitHub through a component called **argocd-repo-server**.

When you create an ArgoCD `Application` resource, you specify:
- `repoURL` - the GitHub repository URL
- `targetRevision` - the branch to watch (e.g. `dev` or `main`)
- `path` - the folder inside the repo containing the Helm chart

ArgoCD then:
1. **Polls** the GitHub repository every 3 minutes (or on webhook)
2. Clones/fetches the repo using `argocd-repo-server`
3. Compares the latest commit on the target branch with the last synced commit
4. If there is a difference → triggers a sync

For **public repositories**, no credentials are needed.  
For **private repositories**, credentials (SSH key or GitHub token) must be added to ArgoCD.

In this assignment, the repo `https://github.com/itamardavideliyahu-beep/Helms.git` is public, so no credentials were needed.

---

### 2. How Helm Rendering Works

Helm is a **templating engine** for Kubernetes manifests.

The chart structure used in this assignment:

```
myapp-chart/
├── Chart.yaml          → Chart metadata (name, version)
├── values.yaml         → Default configuration values
├── values-dev.yaml     → Dev environment overrides
├── values-prod.yaml    → Prod environment overrides
└── templates/
    ├── deployment.yaml → Kubernetes Deployment template
    ├── service.yaml    → Kubernetes Service template
    └── ingress.yaml    → Kubernetes Ingress template
```

**How rendering works:**

1. ArgoCD calls `helm template` internally
2. Helm takes the templates (e.g. `deployment.yaml`) which contain Go template expressions like `{{ .Values.replicaCount }}`
3. Helm merges `values.yaml` with the environment-specific values file (`values-dev.yaml` or `values-prod.yaml`)
4. The result is plain Kubernetes YAML manifests
5. ArgoCD applies those manifests to the cluster with `kubectl apply`

**Example:**

`values-dev.yaml` has `replicaCount: 2` → Helm renders a Deployment with `replicas: 2`  
`values-prod.yaml` has `replicaCount: 3` → Helm renders a Deployment with `replicas: 3`

This is how the same chart produces different environments with different configurations.

---

### 3. How Reconciliation Works

**Reconciliation** is the core of GitOps. It is the process of continuously ensuring that what runs in the cluster matches what is defined in Git.

**The principle:**
- **Git** = Desired state (what SHOULD be running)
- **Kubernetes cluster** = Actual state (what IS running)
- **ArgoCD** = The reconciler (keeps them in sync)

**The loop:**

```
Every ~3 minutes:
  1. ArgoCD reads desired state from Git (Helm chart + values)
  2. ArgoCD reads actual state from Kubernetes API
  3. ArgoCD compares the two states (diff)
  4. If diff detected → ArgoCD applies the desired state to the cluster
```

**Live demo performed in this assignment:**

1. Started with `replicaCount: 1` in `values-dev.yaml` → 1 pod running in dev namespace
2. Changed `replicaCount: 2` in `values-dev.yaml`
3. Committed and pushed to `dev` branch on GitHub
4. ArgoCD detected the change automatically
5. ArgoCD applied the new desired state
6. Result: 2 pods running in dev namespace — **without any manual kubectl command!**

This demonstrates the power of GitOps: **Git is the single source of truth**. Any change to the cluster must go through Git. If someone manually changes the cluster (e.g. `kubectl scale`), ArgoCD will detect the drift and **self-heal** by reverting back to the Git state.

---

## Repository Structure

```
Helms/                          ← GitHub Repository
├── myapp-chart/                ← Helm Chart
│   ├── Chart.yaml
│   ├── values.yaml
│   ├── values-dev.yaml         ← replicaCount: 2, nginx:1.25
│   ├── values-prod.yaml        ← replicaCount: 3, nginx:1.25
│   └── templates/
│       ├── deployment.yaml
│       ├── service.yaml
│       └── ingress.yaml
└── argocd/
    ├── dev-app.yaml            ← ArgoCD Application (branch: dev, namespace: dev)
    └── prod-app.yaml           ← ArgoCD Application (branch: main, namespace: prod)
```

**Branches:**
- `main` → used by `myapp-prod`
- `dev` → used by `myapp-dev`

---

## Summary

This assignment demonstrated a complete GitOps workflow:

1. **ArgoCD** installed on Minikube and connected to a GitHub Helm repository
2. **Two environments** (dev and prod) deployed from the same Helm chart with different values
3. **Automated sync** — changes pushed to Git are automatically applied to the cluster
4. **Self-healing** — if the cluster state drifts from Git, ArgoCD corrects it automatically
5. **Full visibility** — the ArgoCD UI shows the health and sync status of all applications in real time
