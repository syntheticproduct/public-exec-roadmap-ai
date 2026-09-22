# exec-roadmap-ai — project notes

Single-page marketing site for **https://exec-roadmap.ai**, the Executive AI
Roadmap service. No build step, no framework: `index.html` is served verbatim.

The page is plain static HTML — no scripts, no JS dependencies. Its only external
requests are Google Fonts (Newsreader, IBM Plex Sans, IBM Plex Mono). It supports
light and dark via `prefers-color-scheme` and a `data-theme` override.

## Two remotes, one working copy — read this before pushing

This checkout has **two** remotes, and they are not interchangeable:

| Remote   | Repo                                     | Visibility | Role                       |
| -------- | ---------------------------------------- | ---------- | -------------------------- |
| `origin` | `syntheticproduct/exec-roadmap-ai`        | private    | primary project repo       |
| `public` | `syntheticproduct/public-exec-roadmap-ai` | **public** | serves the live site       |

`main` tracks `origin`, so a bare `git push` updates the **private** repo only
and the live site does not change.

**To deploy, push to both:**

    git push origin main && git push public main

GitHub Pages is enabled on the *public* repo (branch `main`, path `/`). Pages
requires a public repo on the free plan, which is why the mirror exists.

## The public repo is world-readable

Anything committed here reaches a public repo, and git history keeps it even
after a later edit.

Published site copy is public *by design* — the marketing text, the bio, and the
price on the page are the point of the page. What must stay out is anything not
meant for the site: credentials, client and prospect names, and unpublished drafts
or pricing still being decided. There is no private-only file here, because both
remotes receive identical content.

## Do not delete CNAME

`CNAME` contains `exec-roadmap.ai` and is what tells Pages its custom domain.
Deleting it drops the domain on the next build.

Two behaviours worth knowing if you ever touch the domain via the API:

- Clearing the domain (`PUT .../pages -f cname=`) does **not** remove the
  `CNAME` file; the file re-asserts the domain on the next build. A genuine
  reset needs the file removed *and* the API field cleared.
- Re-adding the domain makes GitHub commit its own `Create CNAME` straight to
  the public repo, which will reject your next push as non-fast-forward. Fetch
  and reconcile rather than forcing.

## HTTPS

Let's Encrypt cert covers `exec-roadmap.ai` and `www.exec-roadmap.ai`, auto-renews,
and `https_enforced` is on (HTTP 301s to HTTPS; www 301s to apex).

If a cert ever fails to issue, the diagnostic is `https_certificate` in
`gh api repos/syntheticproduct/public-exec-roadmap-ai/pages`. Staying `null` with
no intermediate state means issuance is **stuck, not slow** — waiting will not fix
it. Remove the custom domain (file + API), let it rebuild clean, then re-add.

## DNS

Registrar is **Porkbun**; nameservers are `*.ns.porkbun.com`. Apex has all four
GitHub Pages A records (`185.199.108-111.153`) — all four are required — and
`www` is a CNAME to `syntheticproduct.github.io`. No CAA records are set.
