# Capstone acceptance criteria → how to prove each one

Tick each box live during the final presentation.

## Application delivery
- [ ] **Source change triggers CI.** Edit `backend/app.py` (for example, add a product to `SEED_PRODUCTS`), push to `main`, and show the run under phoenix-app → Actions.
- [ ] **CI tests and builds.** Show the `test` and `build-scan-push` jobs.
- [ ] **Security scan before promotion.** Show the Trivy step. The push step only runs after it.
- [ ] **Immutable, traceable reference.** Show the job summary with `sha-xxxxxxx@sha256:…`.
- [ ] **GitOps repo records the reference.** Show the auto-PR diff on `values-dev.yaml`.
- [ ] **Argo CD reconciles.** Merge the PR and watch `phoenix-dev` sync. The footer of the storefront shows the new version.

## Environment management
- [ ] **Independent dev/qa/prod config.** `diff helm/phoenix/values-dev.yaml helm/phoenix/values-prod.yaml` (replicas, resources, loadgen).
- [ ] **Promotion via reviewed Git change.** Run the `promote` workflow, show the PR, then merge.
- [ ] **Prod needs approval.** The PR shows "Review required — CODEOWNERS". `phoenix-prod` is manual sync.
- [ ] **Declarative resources.** Everything lives under `phoenix-gitops`, and nothing was `kubectl apply`'d by hand except the bootstrap.

## Drift and recovery
- [ ] **Manual change → drift detected.** `kubectl -n phoenix-dev scale rollout phoenix-backend --replicas=3`. On prod, show the OutOfSync diff.
- [ ] **Reconciliation corrects it.** dev returns to 1 replica by itself (self-heal).
- [ ] **Unwanted Git change reverted safely.** Merge a bad change, then `git revert`, and show the history in Argo CD.
- [ ] **Failed canary detected and rolled back.** Set `backend.errorRate: "0.5"` in values-dev, then watch the AnalysisRun fail and the Rollout abort.

## Security and governance
- [ ] **Least-privilege Argo CD.** Log in as `student`, sync `phoenix-dev` (works), then try `phoenix-prod` (denied).
- [ ] **Prod deployment controls.** CODEOWNERS, branch protection and manual sync.
- [ ] **Encrypted secrets.** Show `database.sealedPassword` (ciphertext) in Git, and the decrypted Secret only inside the cluster.
- [ ] **Container scanning in CI.** Trivy step.
- [ ] **Traceable changes.** `git log` on phoenix-gitops; each deploy PR links back to its CI run and source commit.

## Observability and documentation
- [ ] **Dashboards.** Grafana → "Phoenix — Application & Deployment Health".
- [ ] **Architecture diagram.** [architecture.md](architecture.md)
- [ ] **README for setup, deployment and promotion.** [setup-guide.md](setup-guide.md), phoenix-gitops README
- [ ] **Recovery runbook.** [runbook.md](runbook.md)
- [ ] **Full workflow demonstrated live.** [demo-script.md](demo-script.md)
