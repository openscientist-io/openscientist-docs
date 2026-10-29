# openscientist-docs

Source for https://docs.openscientist.io. Built with Zensical (the Material-for-MkDocs successor; it reads the standard `mkdocs.yml`, so `mkdocs build` still works too) and deployed by GitHub Actions to GitHub Pages on every push to `main`.

Local preview: `pip install -r requirements.txt && zensical serve`

Setup checklist (one-time, in this order; from GitHub's docs, verified 2026-10-29)

1. Create the org repo `openscientist-io/openscientist-docs`, push `main`.
2. Repo Settings > Pages > Build and deployment > Source = **GitHub Actions**. The first workflow run deploys to `openscientist-io.github.io/openscientist-docs`.
3. Optional but recommended: Org Settings > Pages > Add a domain `openscientist.io`, then add the TXT record GitHub shows at `_github-pages-challenge-openscientist-io.openscientist.io` in Cloudflare and click Verify. This stops any other GitHub user from claiming `docs.openscientist.io` (or any immediate subdomain) later. Keep the TXT record.
4. Repo Settings > Pages > Custom domain = `docs.openscientist.io`, Save. Do this BEFORE the DNS record exists (GitHub's own ordering, to prevent takeover). With Actions deployment GitHub does not write a CNAME file and ignores `docs/CNAME`; it is kept here only as documentation.
5. Cloudflare DNS for openscientist.io: add `CNAME docs -> openscientist-io.github.io` (the org domain, NOT the repo name, NOT the `*.pages.github.io` host shown in settings). Set it to **DNS only (grey cloud)**. The Cloudflare proxy blocks GitHub's Let's Encrypt HTTP-01 challenge, which is the usual cause of "domain is not properly configured to support HTTPS". No wildcard records.
6. Check: `dig docs.openscientist.io +nostats +nocomments +nocmd` should show the CNAME to openscientist-io.github.io.
7. Wait for the certificate (minutes to a few hours), then tick **Enforce HTTPS**. If it stays greyed out after an hour, remove and re-add the custom domain to retrigger issuance.
