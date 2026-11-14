# Course map: each day → what you build in Phoenix

| Day | Topic | Phoenix files | Key commands / actions | Deliverable |
|---|---|---|---|---|
| 1 | GitOps fundamentals & project setup | `phoenix-app/`, `phoenix-infra/kind/` | `kind create cluster`, `docker build`, `kubectl apply -k phoenix-gitops/apps/...` | Running app on local K8s |
| 2 | K8s manifests & Git workflow | `phoenix-gitops/apps/{frontend,backend,database}` | Feature branch → PR → review → merge ([how we work](how-we-work.md)) | Version-controlled manifests |
| 3 | Argo CD install & first deployment | `argocd/project.yaml`, `argocd/applications/day3-first-app.yaml` | `02-bootstrap-platform.sh`, Argo CD UI → **SYNC** | App deployed from Git |
| 4 | Auto-sync, drift, rollback | uncomment `automated` in day3 app; `argocd/kustomize-apps/*` | `kubectl scale deploy phoenix-backend --replicas=5` → watch revert; delete a Service → recreated; `git revert` | Self-healing deployment |
| 5 | Multi-env with Kustomize | `environments/base`, `environments/{dev,qa,prod}` | `kubectl kustomize environments/qa`, `TRACK=kustomize ./05-deploy-phoenix.sh`, promote by editing `images:` in a PR | 3 independent envs |
| 6 | Helm & ApplicationSet | `helm/phoenix/`, `applicationsets/phoenix-helm.yaml`, `argocd/root-app.yaml` | `helm template phoenix helm/phoenix -f helm/phoenix/values-prod.yaml`, `./05-deploy-phoenix.sh` | dev/qa/prod generated from one definition |
| 7 | GitHub Actions CI + GitOps | `phoenix-app/.github/workflows/ci.yml` | Push to `main` → tests → Trivy → GHCR → PR on phoenix-gitops → merge → Argo syncs | Code-to-cluster pipeline |
| 8 | Security & governance | `argocd/project.yaml`, `argocd/argocd-rbac-cm.yaml`, `.github/CODEOWNERS`, `helm/phoenix/templates/database.yaml` | `03-argocd-access.sh`, `04-seal-db-secrets.sh`, branch protection, log in as `student`, then try to sync prod (denied) | Locked-down platform |
| 9 | Progressive delivery & observability | `templates/backend.yaml` (Rollout), `templates/analysis.yaml`, `templates/monitoring.yaml`, `monitoring/grafana-dashboard.yaml` | Set `backend.errorRate: "0.5"` in values-dev → canary aborts. `kubectl argo rollouts get rollout phoenix-backend -n phoenix-dev -w` | Failed release auto-stopped |
| 10 | Capstone | everything | [demo script](demo-script.md), [capstone checklist](capstone-checklist.md) | Live demo + runbook |

## Learning outcomes → where they're covered

| # | Outcome | Where |
|---|---|---|
| 1 | GitOps vs traditional CI/CD | Day 1 talk. Note that `ci.yml` never runs `kubectl` |
| 2 | Declarative deployment | `apps/`, `environments/`, `helm/` |
| 3 | Operate Argo CD | Days 3–4, `argocd/` |
| 4 | Auto-sync, drift, self-heal, prune | `syncPolicy.automated` in Applications and the ApplicationSet |
| 5 | dev/qa/prod with Kustomize and Helm | `environments/*`, `values-*.yaml` |
| 6 | CI: build, test, scan, publish | `ci.yml` |
| 7 | Secure config, RBAC, approvals | AppProject, RBAC ConfigMap, SealedSecret, CODEOWNERS, manual prod sync |
| 8 | Canary / blue-green with Rollouts | `Rollout` + `AnalysisTemplate` (blue-green is a Day 9 stretch exercise: swap `canary:` for `blueGreen:`) |
| 9 | Prometheus and Grafana | ServiceMonitor, dashboard, analysis queries |
| 10 | Design and document | `project_docs/` |
