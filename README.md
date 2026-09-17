# SPINALIS — site

Two static pages for GitHub Pages, no build step, no JavaScript, no external fonts, no trackers — the
site makes no third-party requests, which is the least a data-sovereignty product's site should do.

| Page | Audience | Contents |
|---|---|---|
| `index.html` | customers | range of results, fit check, frontier comparison, how it works, security answers, no lock-in, cost, pricing, FAQ |
| `investors.html` | investors | the evidence gap, why now, proof, business model, market, competition and moat, plan, team |

Screens in `assets/screens/` are WebP captures of the running product (≈280 KB). The fit check on
`index.html` is computed in CSS with `:has()`; browsers without it show a static reading of the
answers instead.

## Before publishing

1. **Contact address.** `k.khrimpach@gmail.com` is used in every `mailto:` on both pages (several
   carry a pre-filled `?subject=`). To change it: `sed -i 's/k\.khrimpach@gmail\.com/new@domain.eu/g' gitpages/*.html`
2. **Social preview (optional).** `og:image` is relative; most link scrapers need an absolute URL.
3. **Custom domain (optional).** Add a file named `CNAME` containing the domain, e.g. `spinalis.eu`.

## Preview locally

```bash
python3 -m http.server -d gitpages 8000     # then open http://localhost:8000
```

Use a server, not `file://`: the logo is a CSS mask, which browsers refuse to load from `file://`.

## Publish

**Recommended — a separate public repository.** The product repository is proprietary. Publishing
only this folder keeps it that way:

```bash
git subtree push --prefix gitpages git@github.com:<org>/spinalis-site.git main
# then: Settings → Pages → Deploy from a branch → main / (root)
```

Never make the product repository public to get free Pages.

## Where every claim comes from

Market, pricing, economics, competition, plan and team come from **EIC Accelerator Part B v4**. The
results and the frontier comparison come from runs on the development machine on 15 September 2026.

### Results (customer page)

Two rounds of runs: 15 September with the platform's defaults, 16–17 September after the improvements
below. Local models decode greedily; GPT-5.5 (`gpt-5.5-2026-04-23`) ran zero-shot through the OpenAI
API with default reasoning, and those numbers are reused (they cost $13.65 and have not changed).

| Task | Test set | Untrained | Trained on SPINALIS | GPT-5.5 |
|---|---|---|---|---|
| Banking77 intent triage | 1,540 messages, 20 per intent from the public test set | 1.5B 42.3% | **1.5B 90.3%** (90.0% on all 3,080) | 84.9% |
| Text-to-SQL, coffee-roaster DB | 39 questions, 21 queries never in training | 1.5B 4/39 · 7B 12/39 | **1.5B 13/39 (exact 13)** · 7B 15/39 (exact 14) | 18/39, exact 9 |
| DENTEX tooth-crop findings | 523 crops, 102 X-rays never in training | — | **ViT-86M 80.7%**, macro-F1 69.3% | 30.8%, macro-F1 31.6% |
| Retinal photographs | 420 images, split by source image | — | **ViT-86M 91.7%**, ROC-AUC 0.989 | not run (licence unknown) |

Builds: banking `03c45554` (1.5B, 6 epochs, LoRA r=32, 41 min); SQL `36e78e4f` (1.5B, 150 pairs, r=32, 11 min, stopped early at the best checkpoint);
dental `b4cd89a6`; fundus `8fea9937` (first, leaking split `a48ca215`: 93.3%). Superseded runs:
banking `51e5d304` (0.5B 77.7%), `499ba6b6` (1.5B, 3 epochs, 83.4%); SQL `93bde1ca`, `8cec6aa8` (9/39, auto-HPO shrank the adapter to r=8), `c0c52521` (11/39, same data and rank but the last checkpoint, selected against a slice of its own training file), `117c07c6` (7B).

**What moved banking from 83.4% to 90.3%** — six epochs instead of three (accuracy was still climbing
at the end of epoch three) and LoRA r=32 instead of 16. Model selection used the 1,000-message
validation split (82.3% → 88.6% there); the test half was scored once, at the end.

**What did not work, and is worth knowing:** training on the intent alone, without the drafted reply,
*lost* 12 points on validation (70.1% vs 82.3%) — the reply acts as a rationale and helps the model
pick the label. The platform's `accuracy` scorer compares the whole reply string, so on this format it
reads 0.03 and cannot be used to pick checkpoints; loss was used instead.

**Text-to-SQL, second attempt.** The platform's own synthesis step (`reverse_gen` + round-trip filter)
proposed 182 questions from the 31 *training* queries and kept 57 whose regenerated SQL reproduced the
reference result on the snapshot — 93 authored + 57 synthesised = 150 training pairs. Checkpoints were
selected by execution accuracy on 18 held-out *training* questions (never the test set), with the
DuckDB executor pointed at the snapshot and the clock pinned. Result on the 39 test questions:
**13/39** correct answers, all 13 exact. The ladder: 7/39 for the first 1.5B run, 9/39 with adaptive HPO
on (it shrank the adapter to rank 8), 11/39 at rank 32 with the last checkpoint, 13/39 once checkpoint
selection worked — same data, same rank; the difference is which checkpoint shipped. Both r=32 builds
score 0.167 on the 18-question validation set, so the run quoted here is the one the corrected
procedure produced, not the one that happened to score higher; the superseded number is above.
A 1.5B model now reproduces the reference query exactly more often than GPT-5.5 (13 against 9) but
still answers fewer questions correctly overall (13 against 18). The page says exactly that.

**Platform changes these runs needed** (in the product, not in the benchmark scripts): a DuckDB result
executor so execution scoring works against the analytics snapshot with a pinned clock; `eval` in the
build request (metrics, executor, executor options); default epochs scaled by dataset size; and five
bugs that made advertised features no-ops — eval-driven early stop crashed on import; the text
generator dropped the system turn (so SQL was written without the schema); `reverse_gen` decoded
greedily, so it could only ever propose one question per gold query; in-training evaluation used a
random slice of the *training* file even when the build supplied a validation dataset, and ran once
per 500 optimizer steps, so a short run had exactly one checkpoint to "select". The same weights scored
0.80 on that slice of the training file and 0.17 on the uploaded validation set — which is what the
slice was hiding.

### Sizing (customer page)

All runs on one workstation: RTX 4080 SUPER 16 GB, Core i9-13900K, 64 GB RAM, Ubuntu 22.04.
Build times and GPU memory come from each build's record (`fine_tune/metrics.jsonl`, `gpu_mem_mb` peak).
Serving was measured with the platform's own code (`ModelHolder` for text, the `/v1/classify` path for
images), one request at a time as a deployment answers today, excluding network time:

| Deployed model | Request | Median / p95 | Throughput |
|---|---|---|---|
| Qwen2.5 1.5B (adapter `499ba6b6`; same size as the shipped `03c45554`), bf16, 6.0 GB | triage + drafted reply, ~68 tokens out | 0.88 s / 1.20 s | 1.1 req/s ≈ 3,960 / hour |
| Qwen2.5-Coder 7B (build `117c07c6`), 4-bit, 8.9 GB | SQL, 1,246-token prompt, ~57 tokens out | 1.56 s / 2.62 s | 0.61 req/s ≈ 2,190 / hour |
| ViT-Base 86M (build `8fea9937`), fp32, 0.3 GB | classify one photograph | 6 ms / 48 ms | 115 img/s ≈ 412,000 / hour |

The 7B was loaded with `quantization="4bit"` explicitly: a real deployment's `"auto"` cannot size an
adapter whose base model is a hub id (`_estimate_model_size_gb` looks for `config.json` in the adapter
and downloads by the adapter path), so it loads bf16, offloads to CPU and fails. Fix before selling 7B serving.

Manual evidence cost is shown rounded as €12,000–40,000 per system per year (14–28 person-days at
€900–1,400); with two systems Pro equals the lowest estimate, from three it is cheaper.

### Security and lock-in answers (checked against the code)

- Offline libraries: runtime `.spkg` packages via Settings → Libraries (`POST /runtimes/install-spkg`).
- Offline models: copied into the model directory and registered by `POST /models/scan`.
- Exports: weights, GGUF, Docker service, Python package, archive (`handlers_export.go`).
- Lapsed licence: `Manager.GPUBudget()` returns 1 for an invalid licence; nothing else is gated.
- **Not built in, and said so on the page:** native TLS, single sign-on, encryption at rest, HSM/KMS.

## Deliberately not on these pages

- the US Executive Order on AI (revoked January 2025); a "12–18 month window" before sovereign clouds;
- "open source" as a tier, the old name ModelForge;
- loss-reduction, "6-second training", Loss / Perplexity / ROUGE / BLEU as selling points;
- $149/node pricing, dollar prices, top-down TAM/SAM/SOM, 250 customers / $3M ARR, MRR targets;
- unsourced statistics; "no co-founder needed"; the Together.ai / Predibase / Axolotl / Ludwig competitor set;
- employer names from the founders' past, and regional labels that invite the wrong association;
- Banking77 "12% → 94%" (not reproducible), "mined from production logs" (the log is generated);
- the leaking fundus split's 93.3% as a result; loss or speed cards and the `-dirty` version in screenshots;
- any claim that the small model beats GPT-5.5 everywhere — on SQL it does not (18 vs 9 equivalent answers for a 1.5B, 15 for a 7B).
