# openscientist-docs

Source for https://docs.openscientist.io. Built with Zensical (the Material-for-MkDocs successor; it reads the standard `mkdocs.yml`, so `mkdocs build` still works too) and deployed by GitHub Actions to GitHub Pages on every push to `main`.

Local preview: `pip install -r requirements.txt && zensical serve`

Setup checklist (one-time): repo Settings > Pages > Source = GitHub Actions; Settings > Pages > Custom domain = docs.openscientist.io (the `docs/CNAME` file keeps it); Cloudflare DNS: CNAME `docs` -> `openscientist-io.github.io`, DNS only, then enable Enforce HTTPS.

Why a separate repo (Luca, 2026-10-29): one deployment workflow per repo; no cross-wiring with a future automated OpenScientist deployment; no harder than a branch.
