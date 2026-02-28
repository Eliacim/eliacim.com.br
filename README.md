# eliacim.com.br

This is my personal website.

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
├── .github/          # GitHub Actions workflows
├── .well-known/
│   └── security.txt  # Security disclosure policy
├── icons/            # Favicons (16×16 → 256×256)
├── .gitleaks.toml    # Gitleaks allowlist (false-positive suppression)
├── eliacim.jpg       # Profile photo
├── index.html        # Main page
├── robots.txt        # Crawl rules
├── sitemap.xml       # Sitemap for search engines
├── style.css         # Styles and design tokens
└── README.md
```

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

| Branch / Event | Environment | Promotion |
|----------------|-------------|-----------|
| `dev` push | Preview URL (Cloudflare Pages subdomain) | Automatic |
| `main` PR merged | Production (`eliacim.com.br`) | **Manual approval required** in Cloudflare dashboard |

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
