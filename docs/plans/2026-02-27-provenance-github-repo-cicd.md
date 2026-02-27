# Provenance Landing — GitHub Repo + CI/CD Setup Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Create `datafund/provenance-landing` GitHub repo, wire up SSH-based GitHub Actions CI/CD to the campaigns server, then clean up the orphaned feature branch from `datafund/website`.

**Architecture:** Static Astro site deployed via rsync over SSH to the campaigns DigitalOcean droplet. GitHub Actions on `ubuntu-latest` builds the site and rsyncs `dist/` to `/var/www/sites/provenance.datafund.io/` on push to main. No self-hosted runner needed (unlike the website repo which uses Docker).

**Tech Stack:** Astro, Tailwind CSS, GitHub Actions, SSH/rsync, Caddy (on server)

---

### Task 1: Create GitHub repo

**Files:** none

**Step 1: Create the repo**

```bash
gh repo create datafund/provenance-landing \
  --public \
  --description "Landing page for provenance.datafund.io" \
  --source /Users/gregor/Data/1-datafund/2-projects/provenance-landing \
  --remote origin \
  --push
```

Expected: repo created at github.com/datafund/provenance-landing, initial commit pushed.

---

### Task 2: Add GitHub Actions deploy workflow

**Files:**
- Create: `.github/workflows/deploy.yml`

**Step 1: Create the workflow file**

```yaml
name: deploy

on:
  push:
    branches:
      - main

jobs:
  deploy:
    name: build and deploy
    runs-on: ubuntu-latest

    steps:
      - name: checkout
        uses: actions/checkout@v4

      - name: setup node
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: install dependencies
        run: npm ci

      - name: build
        run: npm run build

      - name: setup ssh
        run: |
          mkdir -p ~/.ssh
          echo "${{ secrets.SSH_PRIVATE_KEY }}" > ~/.ssh/deploy_key
          chmod 600 ~/.ssh/deploy_key
          ssh-keyscan -H ${{ secrets.DEPLOY_HOST }} >> ~/.ssh/known_hosts

      - name: deploy
        run: |
          rsync -avz --delete \
            -e "ssh -i ~/.ssh/deploy_key" \
            dist/ \
            deploy@${{ secrets.DEPLOY_HOST }}:/var/www/sites/provenance.datafund.io/

      - name: verify
        run: |
          HTTP_STATUS=$(curl -s -o /dev/null -w "%{http_code}" https://provenance.datafund.io)
          echo "HTTP Status: $HTTP_STATUS"
          if [ "$HTTP_STATUS" != "200" ]; then
            echo "Verification failed"
            exit 1
          fi
```

**Step 2: Commit and push**

```bash
cd /Users/gregor/Data/1-datafund/2-projects/provenance-landing
git add .github/workflows/deploy.yml
git commit -m "ci: add GitHub Actions deploy workflow"
git push
```

Expected: workflow file pushed, but workflow will fail until secrets are set (Task 3).

---

### Task 3: Set GitHub Secrets

**Files:** none (secrets set via `gh` CLI)

**Step 1: Set DEPLOY_HOST**

```bash
source /Users/gregor/Data/.datacore/env/.env
gh secret set DEPLOY_HOST --repo datafund/provenance-landing --body "$DO_DROPLET_IP"
```

**Step 2: Set SSH_PRIVATE_KEY**

```bash
gh secret set SSH_PRIVATE_KEY \
  --repo datafund/provenance-landing \
  < /Users/gregor/Data/.datacore/env/credentials/deploy_key
```

Expected: both secrets set, visible in repo settings (values hidden).

---

### Task 4: Trigger and verify CI/CD

**Step 1: Verify the workflow ran on push from Task 2**

```bash
gh run list --repo datafund/provenance-landing --limit 3
```

**Step 2: If workflow failed (secrets weren't set in time), trigger manually**

```bash
gh workflow run deploy.yml --repo datafund/provenance-landing
```

**Step 3: Watch the run**

```bash
gh run watch --repo datafund/provenance-landing
```

Expected: green deploy run, provenance.datafund.io returns HTTP 200.

---

### Task 5: Update deploy-site.sh to include provenance.datafund.io

**Files:**
- Modify: `/Users/gregor/Data/.datacore/modules/campaigns/scripts/deploy-site.sh`

Add provenance.datafund.io to the case statement for manual/emergency deploys:

```bash
"provenance.datafund.io")
    DEFAULT_SOURCE="$DATA_ROOT/1-datafund/2-projects/provenance-landing/dist"
    SITE_TYPE="landing"
    ;;
```

Commit in the Data repo:

```bash
cd /Users/gregor/Data
git add .datacore/modules/campaigns/scripts/deploy-site.sh
git commit -m "feat(campaigns): add provenance.datafund.io to deploy-site.sh"
```

---

### Task 6: Clean up datafund/website

**Step 1: Delete remote feature branch**

```bash
cd /Users/gregor/Data/1-datafund/2-projects/website
git push origin --delete feature/provenance-landing
```

**Step 2: Delete local feature branch**

```bash
git branch -d feature/provenance-landing
```

Expected: branch gone locally and on GitHub.

---

### Task 7: Update CLAUDE.md in provenance-landing

**Files:**
- Create: `/Users/gregor/Data/1-datafund/2-projects/provenance-landing/CLAUDE.md`

```markdown
# Datafund Provenance Landing

Landing page for [provenance.datafund.io](https://provenance.datafund.io)

## Overview

Standalone Astro + Tailwind landing page for the Datafund Provenance product.

## Tech Stack

- Framework: Astro
- Styling: Tailwind CSS
- Build: npm

## Development

```bash
npm install
npm run dev       # http://localhost:4321
npm run build     # Output to dist/
npm run preview
```

## Deployment

**Target:** https://provenance.datafund.io

**CI/CD:** GitHub Actions on push to main → rsync to campaigns server

**Manual (emergency):** `~/.datacore/modules/campaigns/scripts/deploy-site.sh provenance.datafund.io`

## Related

- GitHub: https://github.com/datafund/provenance-landing
- Campaigns server: `deploy@$DO_DROPLET_IP:/var/www/sites/provenance.datafund.io/`
- Parent space: `/Users/gregor/Data/1-datafund/`
```

Commit:

```bash
cd /Users/gregor/Data/1-datafund/2-projects/provenance-landing
git add CLAUDE.md
git commit -m "docs: add CLAUDE.md"
git push
```

---

## Summary

After completion:
- `github.com/datafund/provenance-landing` is the source of truth
- Every push to main auto-deploys to `provenance.datafund.io`
- `datafund/website` `feature/provenance-landing` branch is deleted
- Manual deploy fallback remains via `deploy-site.sh`
