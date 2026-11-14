# Setup guide: zero to a running platform

Everything runs in **Google Cloud Shell**, which needs nothing installed on your laptop and works the same on Windows, Mac or Linux. Cloud Shell already has `gcloud`, `kubectl`, `helm`, `git` and `docker`.

**Time:** about 45 minutes, most of it waiting for the cluster and load balancers.

---

## 1. GitHub: tokens and settings (10 min)

**a) Create a personal access token** (classic) at https://github.com/settings/tokens. Give it the `repo`, `workflow` and `write:packages` scopes and call it `phoenix-gitops`. You'll use it for `git push` and as the CI secret.

> The `workflow` scope matters: without it GitHub rejects any push that contains `.github/workflows/`.

**b) phoenix-app:** go to Settings → Secrets and variables → Actions.
- Secret `GITOPS_TOKEN` = the token above.

**c) phoenix-gitops:** go to Settings → Actions → General → Workflow permissions.
- Choose **Read and write permissions**.
- Tick **Allow GitHub Actions to create and approve pull requests**. The `promote` workflow needs this.
- If either option is greyed out, enable it first at org level: `phoenix-skies` → Settings → Actions → General.

**d) Team for CODEOWNERS:** go to `phoenix-skies` → Teams → New team `release-managers` and add yourself.

**e) Branch protection on phoenix-gitops `main`:** go to Settings → Branches → Add rule.
- Require a pull request before merging, with 1 approval and **Require review from Code Owners**.
- Require status checks: `render-and-validate`.
- If you're the only person on the team, add yourself to **Allow specified actors to bypass**. Otherwise you can't merge your own PRs during a demo.

## 2. Push the code (10 min)

In Cloud Shell, click **⋮ → Upload** and upload `phoenix-project.zip`. Then run:

```bash
unzip -o phoenix-project.zip -d ~/phoenix && cd ~/phoenix
git config --global user.name  "Your Name"
git config --global user.email "you@example.com"
git config --global credential.helper store      # paste the token as the password once

for repo in phoenix-app phoenix-gitops phoenix-infra project_docs; do
  rm -rf /tmp/$repo && git clone https://github.com/phoenix-skies/$repo.git /tmp/$repo
  cp -r $repo/. /tmp/$repo/
  (cd /tmp/$repo && git add -A && git commit -m "feat: Project Phoenix GitOps platform" && git push origin HEAD:main)
done
```

Pushing `phoenix-app` starts the **ci** workflow straight away, so watch its Actions tab. When it finishes:
- `phoenix-skies` → **Packages** → `phoenix-backend` → Package settings → Change visibility → **Public**. Do the same for `phoenix-frontend`. GKE can then pull without credentials.
- `phoenix-gitops` now has a PR called **deploy(dev): phoenix sha-…**. Leave it open for now.

> If Trivy blocks the build on a brand-new CVE, set the repo variable `TRIVY_EXIT_CODE=0` (Settings → Variables). That makes the scan report-only. Re-run the job, and fix it properly later.

## 3. Cluster and platform (20 min; run while CI is running)

```bash
cd ~/phoenix/phoenix-infra/scripts
sed -i 's/your-gcp-project-id/<YOUR_PROJECT_ID>/' env.sh     # gcloud projects list
./00-install-tools.sh && source ~/.bashrc
./01-create-gke.sh            # ~6 min
./02-bootstrap-platform.sh    # ~8 min — Argo CD, Rollouts, Sealed Secrets, Prometheus, Grafana
./03-argocd-access.sh         # RBAC + users student / release-manager + AppProject
```

## 4. Secrets, then deploy (10 min)

```bash
cd ~/phoenix/phoenix-infra/scripts
GITOPS_DIR=/tmp/phoenix-gitops ./04-seal-db-secrets.sh
cd /tmp/phoenix-gitops && git checkout -b secrets/sealed-db && git commit -am "feat(security): sealed DB secrets" && git push -u origin HEAD
# open the PR on GitHub, wait for "validate" to pass, then merge it
# also merge the CI's "deploy(dev)" PR
cd ~/phoenix/phoenix-infra/scripts && GITOPS_DIR=/tmp/phoenix-gitops ./05-deploy-phoenix.sh
```

Argo CD now creates `phoenix-dev`, `phoenix-qa` and `phoenix-prod`. Prod waits for a manual sync.

## 5. Promote and verify

1. `phoenix-gitops` → Actions → **promote** → dev → qa. Merge the PR and QA syncs automatically.
2. Promote again, qa → prod, and merge (CODEOWNERS approval).
3. In Argo CD, open `phoenix-prod` → **SYNC**.
4. Run `./show-urls.sh` to print Argo CD, Grafana and all three storefront URLs.

## Local alternative (students, Days 1–5)

```bash
kind create cluster --config phoenix-infra/kind/kind-config.yaml
kubectl apply -k phoenix-gitops/environments/dev      # Day 2 style, no Argo CD
kubectl -n phoenix-kz-dev port-forward svc/phoenix-frontend 8080:80
```

## Cost control

The cluster costs roughly ₹10–12 an hour, all covered by free-trial credits. Run `./99-teardown.sh` when you're done for the day. Sealed secrets must be regenerated (step 4) on a new cluster.
