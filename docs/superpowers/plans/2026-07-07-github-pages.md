# GitHub Pages Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Automatically deploy the current `signal.html` from `main` to GitHub Pages with GitHub's official Pages Actions.

**Architecture:** A two-job GitHub Actions workflow validates and packages the single-file game, then deploys the uploaded Pages artifact into the `github-pages` environment. The generated site never lives on `main`; the existing `gh-pages` branch is removed only after the artifact deployment and public URL are verified.

**Tech Stack:** GitHub Actions, Node.js syntax checking, GitHub Pages artifact deployment

---

### Task 1: Add the Pages workflow

**Files:**
- Create: `.github/workflows/deploy-pages.yml`

- [ ] **Step 1: Confirm no Pages workflow exists**

Run:

```bash
rtk rg --files .github/workflows
```

Expected: no workflow files are listed.

- [ ] **Step 2: Add the workflow**

Create `.github/workflows/deploy-pages.yml` with:

```yaml
name: Deploy GitHub Pages

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: pages
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Check out repository
        uses: actions/checkout@v6

      - name: Validate JavaScript syntax
        run: |
          node -e "const fs=require('fs');const html=fs.readFileSync('signal.html','utf8');const match=html.match(/<script>([\\s\\S]*?)<\\/script>/);if(!match)throw new Error('No script block found');fs.writeFileSync('/tmp/signal-script.js',match[1]);"
          node --check /tmp/signal-script.js

      - name: Prepare site
        run: |
          mkdir _site
          cp signal.html _site/index.html
          touch _site/.nojekyll

      - name: Configure Pages
        uses: actions/configure-pages@v5

      - name: Upload Pages artifact
        uses: actions/upload-pages-artifact@v4
        with:
          path: _site
          include-hidden-files: true

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy Pages artifact
        id: deployment
        uses: actions/deploy-pages@v4
```

- [ ] **Step 3: Validate workflow and artifact locally**

Run:

```bash
rtk python3 -c "import yaml; yaml.safe_load(open('.github/workflows/deploy-pages.yml')); print('YAML OK')"
rtk node -e "const fs=require('fs');const html=fs.readFileSync('signal.html','utf8');const match=html.match(/<script>([\s\S]*?)<\/script>/);if(!match)throw new Error('No script block found');fs.writeFileSync('/tmp/signal-script.js',match[1]);"
rtk node --check /tmp/signal-script.js
rtk git diff --check
```

Expected: `YAML OK`, no JavaScript syntax error, and no whitespace errors.

- [ ] **Step 4: Commit the workflow**

```bash
rtk git add .github/workflows/deploy-pages.yml docs/superpowers/plans/2026-07-07-github-pages.md
rtk git commit -m "Add GitHub Pages deployment workflow"
```

Expected: one commit containing the workflow and implementation plan.

### Task 2: Deploy and verify Pages

**Files:**
- Verify: `.github/workflows/deploy-pages.yml`
- Verify: `signal.html`

- [ ] **Step 1: Push `main`**

Run:

```bash
rtk git push origin main
```

Expected: the local Pages commits are pushed and a `Deploy GitHub Pages` workflow run starts.

- [ ] **Step 2: Ensure Pages uses GitHub Actions**

In GitHub repository settings, Pages must use **GitHub Actions** as its build and deployment source. If the workflow reports that Pages is not enabled for Actions, change **Settings → Pages → Build and deployment → Source** to **GitHub Actions**, then rerun the failed workflow.

- [ ] **Step 3: Monitor the deployment**

Run after GitHub CLI authentication is valid:

```bash
rtk gh run list --workflow deploy-pages.yml --limit 1
rtk gh run watch --exit-status
```

Expected: both `build` and `deploy` jobs complete successfully.

- [ ] **Step 4: Verify the public artifact**

Fetch `https://hendrism.github.io/signal_game/` and confirm it returns HTTP 200 and contains both `t2_extended_survey` and `fragCost:0`.

- [ ] **Step 5: Remove the stale generated branch**

Only after Step 4 succeeds, run:

```bash
rtk git push origin --delete gh-pages
rtk git fetch origin --prune
rtk git branch -r
```

Expected: `origin/gh-pages` is absent while the verified Pages URL remains available through the Actions deployment.

- [ ] **Step 6: Confirm final repository state**

Run:

```bash
rtk git status -sb
rtk git rev-list --left-right --count origin/main...main
```

Expected: a clean `main` and `0 0` divergence from `origin/main`.
