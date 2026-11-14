# Recovery runbook

Golden rule: **fix it in Git, not in the cluster.** A manual `kubectl` fix will be reverted by self-heal, and it leaves no audit trail.

Useful aliases:
```bash
export NS=phoenix-dev
alias k=kubectl
k -n argocd get applications                        # sync + health of everything
argocd app get phoenix-dev --grpc-web               # details and last sync result
k -n $NS get pods,rollouts,svc
kubectl argo rollouts get rollout phoenix-backend -n $NS
```

## 1. App shows OutOfSync and won't sync
| Check | Fix |
|---|---|
| `argocd app get <app>` shows "permission denied / not permitted in project" | The destination namespace or resource kind isn't allowed by the `phoenix` AppProject. Fix the manifest or the project, in Git |
| `ComparisonError` / `Unable to generate manifests` | Bad Helm or Kustomize. Reproduce with `helm template ...` or `kubectl kustomize ...`. The `validate` workflow should have caught it on the PR |
| prod stays OutOfSync | Expected: prod is manual. A release manager clicks **SYNC** |
| Repo unreachable | Argo CD → Settings → Repositories. A public repo needs no credentials |

## 2. Pods in ImagePullBackOff
1. `k -n $NS describe pod <pod>` → look for `denied` / `not found`.
2. The GHCR package is private → make it **Public** (see the setup guide) or add an imagePullSecret.
3. The tag doesn't exist → compare `values-<env>.yaml` with the tags in GHCR. Revert the bad PR.

## 3. Product API CrashLoop / not Ready
- `k -n $NS logs deploy/phoenix-backend` or `k -n $NS logs -l app.kubernetes.io/name=phoenix-backend`
- `readyz` fails = the database can't be reached. Check `k -n $NS get pods -l app.kubernetes.io/name=phoenix-db` and the secret: `k -n $NS get secret phoenix-db`.
- SealedSecret not decrypting (`k -n $NS get sealedsecret phoenix-db -o yaml` → status). The ciphertext was made for a different cluster or namespace. Re-run `04-seal-db-secrets.sh`, open a PR and merge it.
- If you changed the DB password but Postgres already initialised its volume with the old one, delete the PVC in that env, **dev/qa only**: `k -n $NS delete pvc data-phoenix-db-0 && k -n $NS delete pod phoenix-db-0`.

## 4. Drift detected (someone used kubectl)
- With self-heal on (dev/qa), Argo CD reverts within seconds. Check **History and rollback** and the app events.
- On prod (manual sync), the app shows OutOfSync and the diff. Decide whether the change is wanted:
  - **Wanted** → raise a PR to put it in Git, then sync.
  - **Not wanted** → click **SYNC** to restore the Git state.

## 5. A bad change was merged
```bash
cd phoenix-gitops && git log --oneline -5
git revert <sha> && git push            # or revert the PR on GitHub (needs review on prod)
```
Argo CD rolls forward to the previous good state. Don't use the "Rollback" button in the Argo CD UI on auto-sync apps: auto-sync is disabled while you do, and Git still holds the bad state.

## 6. Canary failed (Rollout Degraded / Aborted)
This is the safety net working. Users stayed on the stable version.
1. `kubectl argo rollouts get rollout phoenix-backend -n $NS` shows step, analysis result and which pods are stable vs canary.
2. `k -n $NS get analysisrun` then `k -n $NS describe analysisrun <name>` shows the measured error rate.
3. Fix the desired state in Git (revert the bad image or config). The Rollout returns to Healthy on the next sync.
4. Abort or retry by hand (dev): `kubectl argo rollouts abort phoenix-backend -n $NS` / `kubectl argo rollouts retry rollout phoenix-backend -n $NS`.

## 7. Analysis always "Inconclusive" or errors
- Is Prometheus scraping? Open Grafana → Explore: `phoenix_http_requests_total`.
- Missing `rollouts_pod_template_hash` label = ServiceMonitor `podTargetLabels` not applied. Check `k -n $NS get servicemonitor phoenix-backend -o yaml`.
- No traffic → result is NaN, which counts as a pass. Make sure the loadgen is running in that env.

## 8. Argo CD itself is broken
```bash
k -n argocd get pods
k -n argocd rollout restart deploy argocd-server argocd-repo-server statefulset argocd-application-controller
```
Last resort: re-run `02-bootstrap-platform.sh` and `03-argocd-access.sh`, then `05-deploy-phoenix.sh`. All app state comes back from Git. **That is the point of GitOps.**

## 9. Whole cluster lost
`01-create-gke.sh` → `02` → `03` → `04` (new sealed secrets, PR, merge) → `05`. Expect about 30 minutes to full recovery. Production data in Postgres is not backed up in this training setup. Mention this in your design review.
