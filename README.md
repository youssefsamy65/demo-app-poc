# Demo App - CI/CD POC

[![Build & Deploy](https://github.com/OWNER/demo-app-poc/actions/workflows/build.yml/badge.svg)](https://github.com/OWNER/demo-app-poc/actions/workflows/build.yml)

> Replace `OWNER` in the badge URL with your GitHub username once the repo exists.

A tiny, throwaway static web app (a demo login + dashboard) used to demonstrate a
CI/CD pipeline. Credentials are hardcoded demo values shown on the page
(`demo` / `demo123`) — there is nothing real here.

Open `index.html` in a browser to try it.

## CI/CD

This repo is the **app** side of the demo. A code change here runs a build
pipeline, and a successful build triggers the **test** pipeline in the separate
Playwright POC repo:

```
Commit ─▶ Build / validate ─▶ Deploy to Pages ─▶ Trigger test pipeline
 (here)                                              (Playwright POC repo)
```

Defined in [`.github/workflows/build.yml`](.github/workflows/build.yml):

| Job | What it does |
|---|---|
| `build` | Validates `index.html` with `tidy` — a broken commit fails here and never deploys |
| `deploy` | Publishes the site to GitHub Pages |
| `trigger-tests` | Sends a `repository_dispatch` (`app-updated`) to the test repo, with the deployed URL |

### One-time setup for the cross-repo trigger

The `trigger-tests` job needs permission to dispatch into the test repo:

1. **Variable** — in this repo: Settings → Secrets and variables → Actions →
   *Variables* → add `TEST_REPO` = `OWNER/playwright-poc` (the test repo's `owner/name`).
   If `TEST_REPO` is unset, the trigger step is skipped (build + deploy still run).
2. **Token** — create a fine-grained Personal Access Token with **Contents: read &
   write** (or classic PAT with `repo` scope) on the test repo. Add it in this repo
   as a *secret* named `TEST_REPO_TOKEN`.
3. The test repo's workflow already listens for `repository_dispatch: [app-updated]`.

Enable Pages: Settings → Pages → Source: **GitHub Actions**.
