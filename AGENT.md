# Agent Instructions — eliacim.com.br

This is a **static personal website** (plain HTML + CSS, no build step, no frameworks).
Hosting: Cloudflare Pages. CI: GitHub Actions (`.github/workflows/ci.yml`).

---

## Change cheat sheet

### HTML or CSS edit
- Edit `index.html` or `style.css`.
- Optionally bump `<lastmod>` in `sitemap.xml` to today's date.
- **Nothing else to touch.** No commands to run locally.

### Add or replace an image / icon
- Drop the file into `images/` or `icons/`.
- If the filename changed, update the reference in `index.html`.
- **Nothing else to touch.**

### Bump a linter version in `package.json`
First, check whether any updates are actually available:
```bash
docker run --rm -v "$PWD":/app -w /app --user "$(id -u):$(id -g)" node:lts-alpine npm outdated
```
Ignore any `npm notice` lines — those are npm self-promotion and are irrelevant (the container is throw-away).
If no package table is printed, nothing needs bumping. If a table of outdated packages appears:
- Edit the relevant version(s) in `package.json`.
- Regenerate the lockfile and clean up:
  ```bash
  docker run --rm -v "$PWD":/app -w /app --user "$(id -u):$(id -g)" node:lts-alpine npm install && rm -rf node_modules
  ```
- Commit both `package.json` and `package-lock.json`.

### Add or change a CI step
- Edit `.github/workflows/ci.yml`.
- Pin new Actions to a commit SHA (not a mutable tag).

---

## Deploy flow

```
edit files → git commit → git push origin dev
  → CI runs (Gitleaks + htmlhint + stylelint + xmllint + lychee)
  → Cloudflare Pages builds a preview URL automatically

ready for production?
  → open PR: dev → main → merge
  → Cloudflare Pages detects main change, builds, waits for manual approval
  → approve in Cloudflare dashboard → goes live on eliacim.com.br
```

---

## What NOT to touch unnecessarily

| File | Only change when… |
|---|---|
| `package.json` / `package-lock.json` | Linter version bump |
| `.github/workflows/ci.yml` | CI pipeline change |
| `.github/CODEOWNERS` | Ownership / required-reviewer rules change |
| `.github/SECURITY.md` | Security policy change |
| `.github/dependabot.yml` | Dependabot schedule or ecosystem change |
| `robots.txt` | Crawl rules change |
| `sitemap.xml` | New URL added or content date updated |
| `.gitleaks.toml` | New Gitleaks false positive to suppress |
| `.stylelintrc.json` | Stylelint rule change |
| `README.md` | Project-level documentation change |

---

## Run full CI locally (mirrors `.github/workflows/ci.yml`)

Run these in order — same sequence as the CI jobs.

```bash
# 1. Security & secret scanning (job: security-scan)
docker run --rm -v "$PWD":/path \
  zricethezav/gitleaks:latest detect --source /path --verbose

# 2a. HTML linting (job: quality-checks)
docker run --rm -v "$PWD":/app -w /app --user "$(id -u):$(id -g)" node:lts-alpine \
  sh -c "npm ci --silent && npx htmlhint '**/*.html'" && rm -rf node_modules

# 2b. CSS linting
docker run --rm -v "$PWD":/app -w /app --user "$(id -u):$(id -g)" node:lts-alpine \
  sh -c "npm ci --silent && npx stylelint '**/*.css'" && rm -rf node_modules

# 2c. Sitemap XML validation
# Note: no --user flag — apk needs root inside the container, but nothing is written to the host
docker run --rm -v "$PWD":/app -w /app node:lts-alpine sh -c "apk add --no-cache libxml2-utils -q && xmllint --noout sitemap.xml"

# 2d. Broken link check
# Note: -w /input sets cwd; --exclude-path /proc prevents Docker symlink loop; list HTML files explicitly
docker run --rm -v "$PWD":/input -w /input \
  lycheeverse/lychee:latest --verbose --no-progress --exclude-path /proc index.html README.md
```

All steps use Docker - no local Node.js, Go, or Rust required.

---

## Stack summary

- No framework, no bundler, no transpiler.
- `package.json` exists only to pin linter versions for CI (`npm ci`).
- Cloudflare Pages serves files as-is; no build command configured.
