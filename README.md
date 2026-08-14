# NDU Project — Staging Deployment Repository

This repository is the **static hosting target** for the NDU Project staging environment.

**Live URL:** https://staging.nduproject.com

---

## ⚠️ Do Not Edit Manually

This repository is **automatically populated** by the
`deploy-to-staging-repo.yml` GitHub Actions workflow in the source repository:

- **Source repo:** [`Ndu-Project-LLC/NDU-Project`](https://github.com/Ndu-Project-LLC/NDU-Project)
- **Trigger:** Any successful deploy to the `staging` environment in the source repo
- **What gets pushed:** The contents of `build/web/` from a `flutter build web --release` run

Manual commits to this repository will be **overwritten** on the next automated deploy.

---

## How It Works

```
┌─────────────────────────────────┐
│  Ndu-Project-LLC/NDU-Project    │   ← source repo (Flutter app)
│  branch: main / ci-pipeline     │
│                                 │
│  on: push to main               │
│       OR workflow_dispatch      │
│                                 │
│  ┌───────────────────────────┐  │
│  │ deploy-to-staging-repo   │  │   ← GitHub Actions workflow
│  │  1. checkout source      │  │
│  │  2. flutter build web    │  │
│  │  3. force-push build/web │  │
│  │     → ndu-staging:main   │  │
│  └───────────────────────────┘  │
└────────────┬────────────────────┘
             │
             │ git push --force
             │ (using STAGING_REPO_DEPLOY_TOKEN secret)
             ▼
┌─────────────────────────────────┐
│  Ndu-Project-LLC/ndu-staging    │   ← THIS repo (static hosting)
│  branch: main                   │
│                                 │
│  GitHub Pages: enabled          │
│  CNAME: staging.nduproject.com  │
│                                 │
│  → https://staging.nduproject.com │
└─────────────────────────────────┘
```

---

## Repository Structure

```
ndu-staging/
├── .nojekyll          ← tells GitHub Pages to not run Jekyll
├── CNAME              ← custom domain: staging.nduproject.com
├── README.md          ← this file
├── index.html         ← placeholder (overwritten on first real deploy)
├── main.dart.js       ← Flutter compiled JS (added on first deploy)
├── index.html         ← Flutter bootstrapper (overwritten on first deploy)
├── flutter.js
├── flutter_service_worker.js
├── manifest.json
├── favicon.png
├── favicon.ico
├── icons/
├── canvaskit/
├── assets/
└── ...                ← all other build/web/ artifacts
```

---

## GitHub Pages Configuration

This repository must have GitHub Pages enabled:

1. **Settings → Pages**
2. **Source:** Deploy from a branch
3. **Branch:** `main` / `(root)`
4. **Custom domain:** `staging.nduproject.com` (read from `CNAME` file)
5. **Enforce HTTPS:** ✅ (recommended)

---

## DNS Configuration

Add a CNAME record in your DNS provider (Cloudflare, Namecheap, etc.):

| Type  | Name     | Value                          | TTL  |
|-------|----------|--------------------------------|------|
| CNAME | staging  | ndu-project-llc.github.io.     | Auto |

> **Note:** The value is `ndu-project-llc.github.io.` (the organization's GitHub Pages domain), NOT the repo name. GitHub Pages will route from there to this repo based on the CNAME file.

---

## Required Secret in Source Repo

The source repo (`Ndu-Project-LLC/NDU-Project`) must have the following secret configured:

| Secret Name                   | Value                                                                 |
|-------------------------------|-----------------------------------------------------------------------|
| `STAGING_REPO_DEPLOY_TOKEN`   | A GitHub PAT with `repo` scope (or fine-grained PAT with `Contents: Write` on `Ndu-Project-LLC/ndu-staging`) |

See `SETUP_STAGING_REPO.md` in the source repo's root for step-by-step setup instructions.

---

## Rollback

To roll back to a previous staging deploy:

```bash
cd ndu-staging
git log --oneline                   # find the commit you want to roll back to
git checkout <commit-sha> -- .
git commit -m "rollback: revert to <commit-sha>"
git push origin main
```

GitHub Pages will rebuild within ~60 seconds.

---

## Questions?

- **Source repo:** `Ndu-Project-LLC/NDU-Project`
- **Workflow file:** `.github/workflows/deploy-to-staging-repo.yml`
- **Setup guide:** `SETUP_STAGING_REPO.md` in the source repo's root
