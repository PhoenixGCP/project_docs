# Demo script: Project Phoenix, end to end (3 hours)

**Before students join (06:45):** run `./show-urls.sh` and open these tabs: Argo CD (admin), Grafana dashboard, the three storefronts, and GitHub (phoenix-app Actions, phoenix-gitops PRs). In one terminal, keep `kubectl argo rollouts get rollout phoenix-backend -n phoenix-dev -w` running.

| Time | Segment | What you show | Talking point |
|---|---|---|---|
| 07:00 | **The story** (10 min) | project_docs README, architecture diagram | DataQuaint won Phoenix Electronics. We ship their store the GitOps way. CI builds; Git decides; Argo CD delivers |
| 07:10 | **The repos** (20 min) | phoenix-app tree → phoenix-gitops tree → course map | Two repos, two responsibilities. Walk through `apps/` → `environments/` → `helm/` → `argocd/` |
| 07:30 | **The platform live** (15 min) | Argo CD: `phoenix-root` → ApplicationSet → dev/qa/prod; app tree view | App of Apps. One ApplicationSet produces three environments. Sync status vs health status |
| 07:45 | **Storefront** (10 min) | Open dev/qa/prod URLs, add to cart, footer versions | Same chart, different values: replicas 1/2/4 |
| 07:55 | **Code to cluster** (25 min) | Edit `SEED_PRODUCTS` in `backend/app.py` → commit → CI run (test, Trivy, push) → auto-PR on phoenix-gitops → merge → dev syncs → new product visible | Immutable `sha@digest`. CI never touches the cluster |
| 08:20 | *Break* (10 min) | | |
| 08:30 | **Promotion** (20 min) | `promote` dev→qa (auto sync), qa→prod (CODEOWNERS, manual SYNC) | Promotion = PR. Prod needs a human twice: approve, then sync |
| 08:50 | **Drift and self-heal** (15 min) | `kubectl -n phoenix-dev scale rollout phoenix-backend --replicas=3` → reverted. Same on prod → OutOfSync diff, then SYNC | Git is the source of truth. Self-heal on dev/qa, deliberate on prod |
| 09:05 | **Bad canary** (20 min) | PR: `backend.errorRate: "0.5"` in values-dev → merge → watch Rollout step 25%, AnalysisRun fails, abort; Grafana shows the canary hash's 5xx spike → `git revert` → Healthy | Only the canary's own pods are judged. Users barely noticed |
| 09:25 | **Security** (15 min) | Log in as `student`: sync prod → denied. AppProject destinations. SealedSecret ciphertext in Git. Trivy step. Branch protection | Least privilege everywhere |
| 09:40 | **Runbook and Q&A** (15 min) | runbook.md; capstone checklist | How to recover anything, cluster included, from Git |
| 09:55 | **Homework** (5 min) | Fork repos, run Days 1–5 locally on kind | Next session: each team owns a feature |

## Copy-paste helpers

```bash
# watch canary
kubectl argo rollouts get rollout phoenix-backend -n phoenix-dev -w
# drift
kubectl -n phoenix-dev scale rollout phoenix-backend --replicas=3
kubectl -n phoenix-prod scale rollout phoenix-backend --replicas=1
# break the canary (or edit on GitHub)
cd /tmp/phoenix-gitops && git checkout main && git pull
yq -i '.backend.errorRate = "0.5"' helm/phoenix/values-dev.yaml
git checkout -b demo/bad-canary && git commit -am "chore: tweak api (demo)" && git push -u origin HEAD
# recover
git checkout main && git pull && git revert --no-edit HEAD && git push
```

> **Canary tip:** a config-only change (errorRate) creates a new pod template, which triggers a new canary even though the image is the same. Steps take about 30 s each, and the analysis fails within about a minute.

## If something breaks live
- Argo CD slow to notice a merge → click **Refresh** (it polls every 3 minutes).
- CI slower than expected → while it runs, show a previous run's summary and the PR it created.
- LoadBalancer IP missing → `kubectl -n phoenix-dev port-forward svc/phoenix-frontend 8080:80` and use Cloud Shell Web Preview.
