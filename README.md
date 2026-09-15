# SPINALIS — business-card site

A single static page for GitHub Pages: `index.html` + `assets/`. No build step, no JavaScript,
no external fonts, no trackers — the page makes no third-party requests, which is the least a
data-sovereignty product's site should do. Screens in `assets/screens/` are WebP captures of the
running product (≈520 KB in total).

## Before publishing

1. **Contact address.** `CONTACT_EMAIL` appears 11 times in `index.html` (header, calls to action,
   contact block; several carry a pre-filled `?subject=`). Replace all of them at once:
   `sed -i 's/CONTACT_EMAIL/hello@your-domain.eu/g' gitpages/index.html`
2. **Social preview (optional).** `og:image` is relative; most link scrapers need an absolute URL —
   set it once the domain is known.
3. **Custom domain (optional).** Add a file named `CNAME` containing the domain, e.g. `spinalis.eu`.

## Preview locally

```bash
python3 -m http.server -d gitpages 8000     # then open http://localhost:8000
```

Use a server, not `file://`: the logo is a CSS mask, which browsers refuse to load from `file://`.

## Publish

**Recommended — a separate public repository.** The product repository is proprietary (trade
secrets, licence enforcement). Publishing only this folder keeps it that way:

```bash
# once: create an empty public repo, e.g. github.com/<org>/spinalis-site
git subtree push --prefix gitpages git@github.com:<org>/spinalis-site.git main
# then: Settings → Pages → Deploy from a branch → main / (root)
```

Repeat the `git subtree push` after every change (commit first).

**Alternative — Pages from this repository.** GitHub Pages on a *private* repository needs a paid
plan. Never make the product repository public to get free Pages. If the plan allows it, add
`.github/workflows/pages.yml`:

```yaml
name: pages
on: { push: { branches: [main], paths: ["gitpages/**"] } }
permissions: { contents: read, pages: write, id-token: write }
jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: github-pages
    steps:
      - uses: actions/checkout@v4
      - uses: actions/upload-pages-artifact@v3
        with: { path: gitpages }
      - uses: actions/deploy-pages@v4
```

## Where every claim comes from

The source of truth is **EIC Accelerator Part B v4** (`SPINALIS_EIC_PartB_v4.pdf`), not the Canva
deck. Keep them in step: if Part B changes, change this page.

| Section | Part B |
|---|---|
| Evidence gap, Chapter III, Annex IV, Art. 27 | 1 — The problem |
| Execution-grounded verification, cryptographic provenance, generated artefacts, profile packs | 1 — Novelty |
| Three-option comparison, compliance and inference cost (with assumptions) | 1 — Why this is better |
| Banking77 12% → 94%, NL-to-SQL 138/12, synthesis without egress, TRL 5 | 1 — Empirical demonstration, TRL |
| Retinal photographs 91.7%, dental X-rays 80.7% | not in Part B — see below |
| Proprietary licence, permissive-only stack | 1 — IP protection and strategy |
| December 2027, 4,633 → €270M → €54M → €7–11M | 2 — Market opportunity and sizing |
| Tiers per governed AI system | 2 — Business and revenue model |
| Five competitor categories, sovereign-cloud risk | 2 — Competition and its limits |
| M6–M24 milestones | 3 — Implementation plan |
| Founders | 3 — Team capability |

### Demonstrations run on the platform (not in Part B)

Numbers are copied from each build's own `evaluate/eval_results.json` on the development machine.

| Demo | Build | Held-out set | Result |
|---|---|---|---|
| Retinal photographs, 4 classes (public Kaggle archive, licence unknown) | `8fea9937` | 420 images, split by source image (`group_pattern ^_?(\d+)_(?:left\|right\|\d+)$`) | accuracy 91.7%, macro-F1 91.3%, ROC-AUC 0.989 |
| Same data, first split (patient number only) | `a48ca215` | 421 images | accuracy 93.3% — inflated by differently named copies of the same photograph |
| Dental X-ray findings (DENTEX, CC BY-NC-SA 4.0) | `b4cd89a6` | 523 crops from 102 X-rays never in training | accuracy 80.7%, macro-F1 69.3%; identical confusion matrix to build `0874571b` |

Near-duplicate check (DINOv2-small, cosine ≥ 0.98, held-out vs training): first fundus split 24
images, corrected split 4 (none sharing a source number), dental 0.

Patient images are blurred in every screen; DENTEX images are never shown (non-commercial licence).
The text-to-SQL database is synthetic, modelled on a real coffee roaster; its query log was
generated, so the page says "query log", never "production logs".

Two facts come from outside Part B and were checked:

- **Unsloth Studio UI is AGPL-3.0** (core Unsloth: Apache-2.0) — https://unsloth.ai/docs/new/studio
- CTO's prior platform (~400 data scientists, ~3,000 production models, financial services) — stated
  by the founder.

## Deliberately not on this page

These were errors or contradictions in the earlier deck. Do not reintroduce them:

- the US Executive Order on AI (revoked January 2025);
- a "12–18 month window" before hyperscalers ship sovereign clouds (they have);
- "open source" as a tier (the product is proprietary; that is the procurement moat);
- the old name ModelForge;
- loss-reduction and "6-second training" as proof of quality;
- Loss / Perplexity / ROUGE / BLEU as selling points (the thesis is execution-grounded verification);
- $149/node pricing, dollar prices, top-down TAM/SAM/SOM, 250 customers / $3M ARR, MRR targets;
- unsourced statistics (CIO surveys, "stalled initiatives", inference bills, MLOps timelines);
- "no co-founder needed" and an Enterprise AE hire (the COO covers that);
- Together.ai / Predibase / Axolotl / Ludwig as the competitor set, and non-governance comparison axes;
- employer names from the founders' past, and regional labels that invite the wrong association;
- "mined from production logs" for the SQL demo (the log is generated);
- the first fundus split's 93.3% as a result (it leaked; the corrected split is the number);
- training loss, learning rate or speed cards in screenshots, and the build's `-dirty` version string.
