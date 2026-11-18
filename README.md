# Project Phoenix — GitOps-Driven E-Commerce Platform

> **Scenario.** DataQuaint has won a new client, **Phoenix Electronics**, an electronics retailer going online. We are building their e-commerce platform, and we deliver it the way modern platform teams do: with **GitOps**. Your tech lead is your instructor. You are the delivery team.

This is the companion project for the **GitOps Mastery** 10-day programme.

## Repositories (GitHub org `PhoenixGCP`)

| Repo | Purpose |
|---|---|
| [`phoenix-app`](https://github.com/PhoenixGCP/phoenix-app) | Application source (storefront, product API), tests, Dockerfiles, CI |
| [`phoenix-gitops`](https://github.com/PhoenixGCP/phoenix-gitops) | Desired state: manifests, Kustomize, Helm, Argo CD, ApplicationSet |
| [`phoenix-infra`](https://github.com/PhoenixGCP/phoenix-infra) | Cluster and platform bootstrap scripts (GKE / kind) |
| `project_docs` (this repo) | Architecture, course map, setup guide, runbook, demo script |

## Start here

1. [Architecture](docs/architecture.md): how the pieces fit together
2. [Setup guide](docs/setup-guide.md): from zero to a running platform
3. [Course map](docs/course-map.md): every course day mapped to the files and commands you'll use
4. [How we work](docs/how-we-work.md): branching, PRs, issues, epics and stories
5. [Recovery runbook](docs/runbook.md): when things go wrong
6. [Capstone checklist](docs/capstone-checklist.md): the acceptance criteria and how to prove each one
7. [Demo script](docs/demo-script.md): the end-to-end walkthrough

## Tech stack

Git · GitHub · Docker · Kubernetes (kind / GKE) · Argo CD · Kustomize · Helm · GitHub Actions · GHCR · Trivy · Sealed Secrets · Argo CD RBAC & AppProjects · Argo Rollouts · Prometheus · Grafana · Flask · nginx · PostgreSQL
