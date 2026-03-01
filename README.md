# eliacim.com.br

This is my personal website.

## Status

![Build](https://img.shields.io/github/actions/workflow/status/eliacim/eliacim.com.br/ci.yml?logo=github&style=flat-square&label=Build&color=A6E110)
![Security](https://img.shields.io/github/actions/workflow/status/Eliacim/eliacim.com.br/github-code-scanning/codeql?logo=github&style=flat-square&label=Security&color=A6E110)
![Updates](https://img.shields.io/github/actions/workflow/status/Eliacim/eliacim.com.br/dependabot/dependabot-updates?logo=dependabot&style=flat-square&label=Updates&color=A6E110)
![Commit](https://img.shields.io/github/last-commit/eliacim/eliacim.com.br?style=flat-square&logo=github&label=Commit&color=A6E110)
![Website](https://img.shields.io/website?url=https%3A%2F%2Feliacim.com.br&style=flat-square&logo=cloudflare&logoColor=white&label=Site&color=A6E110)

## Overview

A minimal, retro terminal-themed aesthetic single-page site built with plain HTML and CSS.

## Stack

- **HTML** — single `index.html` page
- **CSS** — custom properties, no frameworks
- **Hosting** — Cloudflare
- **Analytics** — Google Tag Manager, Cloudflare

## Project Structure

```
.
├── .github/
│   ├── workflows/
│   │   └── ci.yml        # CI pipeline (security scan + quality checks)
│   ├── CODEOWNERS        # Pull request review requirements
│   ├── SECURITY.md       # Vulnerability reporting policy
│   └── dependabot.yml    # Automated dependency update schedule
├── AGENT.md              # AI agent cheat sheet (workflow, deploy flow, what not to touch)
├── .well-known/
│   └── security.txt      # Security disclosure policy
├── icons/                # Favicons (16×16 → 256×256)
├── images/
│   └── eliacim.jpg       # Profile photo
├── .gitignore
├── .gitleaks.toml        # Gitleaks allowlist (false-positive suppression)
├── .stylelintrc.json     # Stylelint configuration
├── index.html            # Main page
├── package.json          # Lint tooling dependencies
├── package-lock.json     # Pinned lockfile for npm ci
├── robots.txt            # Crawl rules
├── sitemap.xml           # Sitemap for search engines
├── style.css             # Styles and design tokens
└── README.md
```

## Agent Instructions

[`AGENT.md`](AGENT.md) is a cheat sheet for AI agents (Claude Code, Gemini, Google Antigravity, etc.). It documents which files to touch (and which to leave alone) for each type of change, the deploy flow, and the Docker command for lockfile regeneration. Update it whenever the workflow changes.

## CI/CD

### Continuous Integration — GitHub Actions

Defined in [`.github/workflows/ci.yml`](.github/workflows/ci.yml), the pipeline runs on every push to `main`/`dev` and on pull requests targeting `main`.

**Jobs (run in sequence):**

1. **Security & Secret Scanning** (`security-scan`)
   - Runs [Gitleaks](https://github.com/gitleaks/gitleaks-action) across the full git history to detect leaked secrets or credentials.

2. **Code & Link Checks** (`quality-checks`) — only runs if the security scan passes
   - **HTML linting** — `htmlhint` on all `.html` files
   - **CSS linting** — `stylelint` with `stylelint-config-standard` on all `.css` files
   - **Sitemap validation** — `xmllint` validates `sitemap.xml` against the XML spec
   - **Broken link check** — [Lychee](https://github.com/lycheeverse/lychee-action) scans all HTML files and `README.md`

### Continuous Deployment — Cloudflare Pages

Cloudflare Pages is connected directly to the GitHub repository. No separate workflow is needed — deployments are triggered automatically based on branch activity.

| Branch / Event   | Environment                              | Promotion                                            |
| ---------------- | ---------------------------------------- | ---------------------------------------------------- |
| `dev` push       | Preview URL (Cloudflare Pages subdomain) | Automatic                                            |
| `main` PR merged | Production (`eliacim.com.br`)            | **Manual approval required** in Cloudflare dashboard |

**Production deploy flow:**

1. PR is merged into `main` on GitHub.
2. Cloudflare Pages detects the change and starts the build automatically.
3. The deploy is held until manually approved in the Cloudflare Pages dashboard.
4. Once approved, the new build goes live on all production domains.

## Local Development

Open `index.html` in a browser — no build step required.

## SEO

- Canonical URL, Open Graph, and Twitter Card meta tags
- JSON-LD `Person` structured data
- XML sitemap and `robots.txt`

## License

All rights reserved © Eliacim Lopes.
