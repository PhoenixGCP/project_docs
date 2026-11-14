# How we work (developer workflow)

## Branching: trunk-based with short-lived branches
- `main` is always deployable and protected. No direct pushes.
- Branch names: `feat/<issue>-short-name`, `fix/<issue>-...`, `chore/...`, `docs/...`
- Bots use `deploy/dev-<sha>` (CI) and `promote/<env>-<run>` (promote workflow).
- One PR = one logical change. Squash-merge. Keep the description linked to the issue (`Closes #12`).

## Commit messages (Conventional Commits)
`feat(api): add category filter` · `fix(frontend): cart total rounding` · `deploy(dev): phoenix abc1234` · `promote(prod): ...`

## Planning: epics → stories → tasks (GitHub Issues + Projects)
| Level | Example | GitHub |
|---|---|---|
| Epic | Online storefront MVP | Issue labelled `epic`, with a task list of stories |
| User story | *As a shopper, I can filter products by category so that I find items faster* | Issue labelled `story`, acceptance criteria as checkboxes |
| Task | Add `?category=` to `/api/products` | Issue labelled `task`, linked to the story |

Suggested epics for Phoenix:
1. **Storefront MVP**: catalogue, cart (done), checkout (Sprint 2), search
2. **Delivery platform**: GitOps, environments, CI (done in this course)
3. **Reliability**: canary, dashboards, runbook (done), alerts, backups
4. **Security**: RBAC, sealed secrets, scanning (done), image signing (stretch)

## Pull request checklist
- [ ] Tests added or updated, and `pytest` passes locally
- [ ] No secrets in the diff
- [ ] For phoenix-gitops: `helm template` / `kubectl kustomize` renders, and the `validate` check is green
- [ ] Prod changes: CODEOWNERS approval + release manager syncs
