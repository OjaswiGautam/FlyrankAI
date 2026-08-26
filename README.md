# Content Performance Archetypes — FlyRank ML Internship Capstone

An unsupervised K-Means clustering pipeline that discovers behavioral content archetypes
from real search-performance data, audits an existing rule-based content-flagging system
against those archetypes, and turns the result into a ranked, confidence-tagged action
playbook for human review.

**Live paper:** https://ojaswigautam.github.io/FlyrankAI/
**Author:** Ojaswi Gautam

---

## What it does, and for whom

This project answers one question: *what recurring performance archetypes exist across a
large content inventory, based on observable search and engagement signals — and does an
existing rule-based flagging system actually target those archetypes correctly?*

It's built for **content/SEO reviewers** who need a starting point for triaging a large
inventory when reviewer time is limited — not a fully automated decision system. The output
is a ranked review queue with reason codes and confidence tags, meant to sit in front of a
human, not replace one.

## Setup (reproducible by a stranger)

**Requirements:** Python 3.10+, a Hugging Face account with access requested to
[`FlyRank/internship-warehouse`](https://huggingface.co/datasets/FlyRank/internship-warehouse)
(instant approval), and a Hugging Face **Read**-type access token.

```bash
git clone https://github.com/OjaswiGautam/FlyrankAI.git
cd FlyrankAI
pip install -r requirements.txt
```

In Google Colab, store your Hugging Face token as a **Secret** named `HF_TOKEN`
(the key icon in the left sidebar) — never paste it directly into a cell, since this repo
is public.

Run the full pipeline end to end:

```bash
jupyter nbconvert --to notebook --execute work/notebooks/capstone.ipynb
```

This single notebook rebuilds the entire pipeline from the raw warehouse query through the
final ranked recommendation queue — no external state required beyond the token.

## Usage example

```python
# Inside capstone.ipynb — the core pipeline in miniature

# 1. Pull and aggregate one month of real search-performance data
raw = con.sql(f"""
    SELECT client_hash_id, content_hash_id,
           SUM(gsc_impressions) AS gsc_impressions, ...
    FROM read_parquet('{TABLE}') WHERE gsc_data_available = TRUE
    GROUP BY client_hash_id, content_hash_id
""").df()

# 2. Fit the model on a client-grouped split (never a naive random split — see Limitations)
final_km = KMeans(n_clusters=4, random_state=42, n_init=30)
final_km.fit(X_train)

# 3. Overlay an existing baseline rule and measure where it concentrates
lift_by_cluster = baseline_base.groupby('cluster')['baseline_flag'].mean() / global_rate
```

## Architecture

Hugging Face warehouse (79M-row fact table + dimension table)
│
▼
DuckDB query — aggregate to one row per (client, content), March 2026
│
▼
Feature engineering — log-transform skewed counts, impute + flag missingness
│
▼
Client-grouped split (GroupShuffleSplit) ──► Train (35 clients) / Val (12 clients)
│
▼
K-Means (k swept 3–8, selected k=4 on validation silhouette + Davies-Bouldin)
│
▼
Seed-stability check (5 seeds, ARI) ──► Overlay existing baseline rule as diagnostic
│
▼
Ranked, reason-coded, confidence-tagged action queue ──► Deployed research paper


## v2 evaluation results — what changed, and why it matters

The first version of the seed-stability check (`n_init=10`) reported a misleadingly high
mean ARI of 0.9957 across 5 seeds. Re-running it properly (checking each seed's actual
pairwise agreement, not just the mean) revealed one seed had silently converged to a
meaningfully worse solution — a real local-optimum failure, not noise:

| | v1 (`n_init=10`) | v2 (`n_init=30`) |
|---|---|---|
| Mean ARI (5 seeds) | 0.7773 (misleading — one seed masked the rest) | **0.9981** |
| Min ARI | 0.4454 | **0.9963** |
| Cause | One seed's K-Means run stopped at a worse local optimum | Fixed by raising `n_init` from 10 to 30 |

A second, separate correction was made during final paper verification: the baseline-flag
lift-by-archetype figures in an earlier draft were computed against a stale intermediate
dataframe and were wrong. The corrected, independently re-verified figures (final v2 model):

| Archetype | v1 (incorrect draft) | v2 (verified, final) |
|---|---|---|
| Missing/Imputed Metadata | 1.110 | **1.296** |
| Low-Traffic/Emerging | 0.811 | **1.275** |
| Core Established Content | 1.196 | **0.690** |
| Sparse/No-Rank Risk | 0.000 | 0.000 (unchanged) |

Both corrections are disclosed in the deployed paper's Limitations section and in
`capstone.ipynb`, rather than silently edited away.

## Limitations

- **Coverage is partial**: the model covers ~34% of the full content inventory — only
  content with real March 2026 Google Search Console data.
- **Single-month snapshot**: no trend, seasonality, or multi-month lifecycle claim is possible.
- **Two of four archetypes are partly missingness-driven**, not purely behavioral — per-row
  inspection found one cluster's "word count" values were entirely the population median
  (an imputed placeholder), not real per-page data.
- **No causal claims anywhere**: every finding is observed and directional within this
  dataset. Cluster membership is decision-support for human review, never an automated
  quality judgment or a guaranteed outcome.
- **Client-population validation, not per-client validation**: the model generalizes to
  unseen clients as a group; it does not guarantee accuracy for any single specific client.

## Built with AI — what and how

This project was built in collaboration with **Claude** (Anthropic), used throughout as a
coding and review partner: drafting SQL/pandas pipelines, proposing validation designs,
and reviewing my own written analysis against the project's methodology skills. Claude did
**not** independently choose the lane, invent the findings, or decide what counted as
"done" — every real number in this README and the linked paper was executed and verified
by rerunning the actual code against the actual warehouse, not taken on faith from a draft.
Two real bugs (the `n_init` seed-instability issue and the baseline-lift discrepancy above)
were caught specifically *because* of that verification discipline, not despite using AI
assistance — I treated Claude's output as a draft to check, not a finished answer.

## Reproducibility & links

- **Deployed paper:** https://ojaswigautam.github.io/FlyrankAI/
- **Full notebook history:** `work/notebooks/` (w01 through capstone.ipynb)
- **Metrics receipts:** `work/outputs/*.json`, `work/outputs/*.csv`
- **Figures:** `work/figures/`

---

## Full development history

This capstone was built over an 8-week track, one notebook per stage, each committed to
`work/notebooks/`. Every weekly notebook is real, executed work — not a skeleton left
unfilled — and each one is a checkpoint in how the final model and paper were arrived at.

| Week | Card | Notebook | What it covers |
|---|---|---|---|
| 1 | ML-02 | `w01_research_question.ipynb` | Lane selection, research question, cost of a wrong call |
| 2 | ML-03 | `w02_ml_task_framing.ipynb` | Task type (unsupervised), success metric, unit of analysis |
| 3 | ML-04 | `w03_data_contract.ipynb` | Warehouse grain/window verification, field classification, feature frame, leak trap |
| 3 | ML-05 | `w03_feature_leakage_check.ipynb` | Full-depth feature vector build and leakage hunt |
| 4 | ML-06 | `w04_signal_audit.ipynb` | Signal verification behind the baseline rule |
| 4 | ML-07 | `w04_baseline_score.ipynb` | Hand-coded CTR-fix baseline, reason codes, top-20 review |
| 5 | ML-08 | `w05_model.ipynb` | K-Means model, method choice, split design, model vs. baseline |
| 6 | ML-09 | `w06_validation_audit.ipynb` | Paper-methodology audit, honest split before/after, leakage audit, claim rewrite |
| 7 | ML-10 | `w07_action_playbook.ipynb` | Ranked actions, intended use, no-go list, monitoring triggers |
| 8 | ML-11 | `capstone.ipynb` | Full pipeline assembly, corrected results, deployed paper source |

### Data safety

Only anonymized, hashed identifiers (`client_hash_id`, `content_hash_id`) and aggregate
metrics appear anywhere in this repo or the deployed paper. No client names, domains, URLs,
titles, or search query text are present at any stage. See `DATA_USE.md` for the full data
handling policy this project followed.

---

*Built as part of the FlyRank ML Engineering Internship. Code under MIT (see `LICENSE`);
data under `DATA_USE.md`.*
