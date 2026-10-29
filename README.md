# openscientist-docs

Source for https://docs.openscientist.io. Built with Zensical (the Material-for-MkDocs successor; it reads the standard `mkdocs.yml`, so `mkdocs build` still works too) and deployed by GitHub Actions to GitHub Pages on every push to `main`.

Local preview: `uv sync && uv run zensical serve` (dependencies are pinned in `uv.lock`; `uv lock --upgrade` to bump Zensical)

Setup checklist (one-time, in this order; from GitHub's docs, verified 2026-10-29). Status 2026-10-29: steps 1, 2, 4, 5 and 6 done (CNAME added in Route 53, hosted zone Z10166663OZ8U0M4F0H68 in the Monarch AWS account 440333305415); 7 pending certificate issuance; 3 (org-level verification) still to do.

1. Create the org repo `openscientist-io/openscientist-docs`, push `main`.
2. Repo Settings > Pages > Build and deployment > Source = **GitHub Actions**. The first workflow run deploys to `openscientist-io.github.io/openscientist-docs`.
3. Optional but recommended: Org Settings > Pages > Add a domain `openscientist.io`, then add the TXT record GitHub shows at `_github-pages-challenge-openscientist-io.openscientist.io` in the DNS zone and click Verify. This stops any other GitHub user from claiming `docs.openscientist.io` (or any immediate subdomain) later. Keep the TXT record.
4. Repo Settings > Pages > Custom domain = `docs.openscientist.io`, Save. Do this BEFORE the DNS record exists (GitHub's own ordering, to prevent takeover). With Actions deployment GitHub does not write a CNAME file and ignores `docs/CNAME`; it is kept here only as documentation.
5. DNS for openscientist.io: add `CNAME docs -> openscientist-io.github.io` (the org domain, NOT the repo name, NOT the `*.pages.github.io` host shown in settings). As of 2026-10-29 the zone is served by AWS Route 53 (registrar: Amazon Registrar), so add the record there as a plain CNAME. If the zone is ever moved to Cloudflare, keep the record **DNS only (grey cloud)**: the Cloudflare proxy blocks GitHub's Let's Encrypt HTTP-01 challenge, which is the usual cause of "domain is not properly configured to support HTTPS". No wildcard records.
6. Check: `dig docs.openscientist.io +nostats +nocomments +nocmd` should show the CNAME to openscientist-io.github.io.
7. Wait for the certificate (minutes to a few hours), then tick **Enforce HTTPS**. If it stays greyed out after an hour, remove and re-add the custom domain to retrigger issuance.
