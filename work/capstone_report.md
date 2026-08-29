# Capstone Report — Refresh / Content Opportunity Scoring

- **Author:** Dua Shakeel
- **Lane:** Refresh / Content Opportunity Scoring
- **Repo:** https://github.com/duashakeel0/flyrank-ml-internship
- **Date:** 2026-08-20

## 0. Abstract

Which pages should a content editor review first for refresh, given limited weekly capacity? This project builds a decision-support ranking on FlyRank's internship warehouse (March 2026, 9.8M daily rows, 519K content items), comparing a transparent rule baseline against Logistic Regression and Random Forest under a client-holdout split. The baseline never fired on the held-out test set at all; Random Forest reached Precision@50 = 0.64 against a base rate of 0.356, with `avg_position_h1` and `impressions_h1` as its leading features. A deliberate stress test found a naive random split would have reported Precision@50 = 0.84 on the same data — a 20-point gap attributable to client memorization, not real skill. The output is a ranked review queue with reason codes, meant to prioritize human attention, not to replace it.

## 1. Problem framing

**Decision:** which pages an editor with limited weekly review capacity should look at first for refresh (rewrite, expand, protect, prune, or monitor).

**Unit of analysis:** one content item (page), summarized over a 15-day feature window (March 1-15, 2026) with a label built from the following 16-day window (March 16-31).

**Output:** a ranked queue with a risk score, a reason code, an action tier, and a confidence label.

**Action a human takes:** an editor opens the top of the queue, reads the reason code, and decides whether to rewrite, expand, or simply monitor that page.

**Cost of a wrong call:** a false positive wastes scarce editor hours on a page that didn't need attention; a false negative lets a real decline continue unnoticed, a compounding cost. Because editor time is the scarce resource, Precision@K is the metric that matches the real decision — not raw accuracy.

**Why ML helps:** a hand-written rule baseline already exists in this lane (`stale_but_visible`) and is genuinely readable, but it never fired at all on the held-out test population — the pattern here is real (Random Forest clears the base rate by a wide margin) but too spread across interacting signals (position, exposure, age, clicks) for one or two thresholds to capture.

## 2. Data safety

**Source:** [`FlyRank/internship-warehouse`](https://huggingface.co/datasets/FlyRank/internship-warehouse) on Hugging Face (gated, requires a free account + read token — never committed, read from an environment variable at runtime).

**Tables used:** `fact_content_daily_performance` (month=2026-03 partition, 9,841,378 rows) and `dim_content` (519,606 rows, for word count and content dates).

**Date windows:** feature window March 1-15, 2026; target window March 16-31, 2026; decision point March 15, 2026.

**Deliberately excluded, with reasons:**
- `sessions_ai` and every `ai_*` column — too sparse to be a reliable signal at this scale (30,177 AI-session rows vs. 28.9M search-impression rows in the starter slice checked in Week 1).
- Any March 16-31 metric as a *feature* — that window defines the label; using it as an input would be circular. Demonstrated live (Week 3 and Week 6): honest AUC ~0.60-0.69 vs. leaky AUC 0.94-1.00 when a target-window column is added on purpose, then removed.
- Rows where `content_updated_date` or `content_created_date` fell after the March 15 decision point — `dim_content` is a single latest-state snapshot (export date 2026-07-03), not point-in-time, so these dates could be reading the future. Dropping them removed 74,624 of 92,247 candidate rows (81%) — a real cost paid for honesty, not a rounding error.

**IDs** (`client_hash_id`, `content_hash_id`) are pseudonymous, used only for joins, grouping, and the train/test split — never as model features. No client names, domains, URLs, or raw queries appear anywhere in this repo's `work/` folder.

## 3. Baseline

**Rule, in plain words:** a page is worth reviewing if it still has real search demand (`impressions_h1 >= 80`, rescaled from the starter CSV's 90-day threshold of 500 for a 15-day window) *and* it hasn't been touched in a long time (`days_since_last_update >= 180`). Score it by exposure, so among equally-stale-and-visible pages, higher-traffic ones surface first.

**Fairness:** recomputed on the exact same warehouse slice, same feature window, and same client-holdout test rows as the models — not reused from the starter-CSV number.

**Result:** the baseline scored **zero of 1,979 test rows above zero** — it never fires on this held-out client population at all. Reporting a Precision@K from an all-tied, all-zero score array would measure row order, not the rule, so it is reported as **N/A**, not a number that looks real but isn't.

## 4. Model / analysis

**Method:** Logistic Regression first (readable, gives inspectable coefficients), then Random Forest (captures interactions the linear model can't, e.g. "stale AND poorly-positioned" mattering more together than apart). No heavier model was tried, per the explicit pre-commitment: "stop at Logistic Regression if Random Forest doesn't clearly earn its complexity."

**Features (8), all knowable by the March 15 decision point:** `impressions_h1`, `clicks_h1`, `avg_position_h1`, `sessions_h1`, `engaged_sessions_h1`, `days_since_last_update`, `content_age_days`, `word_count`.

**Target:** `is_declining_label = 1` if a page's March 16-31 impressions are lower than its March 1-15 impressions, else 0 (minimum `impressions_h1 >= 50` to avoid labeling near-zero-volume noise). This is a proxy, not a causal outcome — flagged as such since Week 2.

## 5. Evaluation

**Split:** client-holdout (grouped by `client_hash_id`), not a random row split. 28 clients total, 22 train / 6 test, verified 0 client overlap. Chosen because rows from the same client likely share systematic traits (site design, publishing cadence) that a random split would let a model memorize instead of generalize.

**Why this matters, proven not just asserted:** a deliberate before/after test (Week 6) reran the identical model on the identical data under a naive random row split — Precision@50 = **0.840** — versus the honest client-holdout split — Precision@50 = **0.640**. A 20-point gap on the same underlying data, reproduced identically across two independent full reruns.

**The honest result** (test set: 1,979 rows, 6 held-out clients, base rate 0.356):

| Method | Precision@20 | Precision@50 |
|---|---|---|
| Baseline (rescaled rule) | N/A (never fires) | N/A (never fires) |
| Logistic Regression | 0.65 | 0.46 |
| **Random Forest** | 0.60 | **0.64** |

Random Forest wins at K=50, loses narrowly at K=20 — reported as a real split result, not smoothed into a single "winner."

**Errors:** false positives (222 in the Week 5 run) skew toward high-impression pages (4,461-16,986) the model flagged confidently that turned out fine — a genuinely ambiguous case, since high current traffic can mean "still strong" or "about to fall from a height." False negatives (510) skew toward low-volume pages (54-283 impressions) the model was *uncertain* about, not confidently wrong — less signal to work with, not a blind spot.

**Leakage audit, repeated on the final feature set (Week 6):** honest AUC 0.685 vs. leaky AUC 0.936 when a target-window column is deliberately added — confirms the test harness actually catches leakage rather than just asserting it would.

## 6. Interpretation

**Random Forest feature importances:** `avg_position_h1` (0.244), `impressions_h1` (0.201), `content_age_days` (0.167), `clicks_h1` (0.130), `sessions_h1` (0.123), `word_count` (0.103), `engaged_sessions_h1` (0.016), `days_since_last_update` (0.016). All plausible; none suspiciously dominant the way a leaked column would be.

**The consistent negative result:** `days_since_last_update` — the exact "staleness" feature this project fought hardest to make trustworthy (Week 5's point-in-time fix cost 81% of the candidate rows) — ends up with the *lowest* importance of all 8 features, tying for last place. This isn't a one-off: Week 4's own signal check on the starter CSV already returned a **MIXED** verdict for staleness vs. decline rate (0.512 → 0.611 → 0.467 → 0.600 across buckets, non-monotonic). Two independent checks, two datasets, one consistent finding: staleness alone is a weaker signal here than the lane's own name ("Refresh Opportunity Scoring") might suggest. Position and current exposure carry the real weight.

## 7. Recommendation

The final ranked queue (Week 7) is built **only** from the 1,979-row client-holdout test population — the one population the validated Precision@50 = 0.64 actually describes.

| Action tier | Rows | Confidence |
|---|---:|---|
| `prioritize_review` | 11 | high |
| `monitor` | 1,763 | medium |
| `no_action_needed` | 205 | low |

Reason codes among `prioritize_review` rows: `high_exposure_declining_risk` (9), `poor_position_declining_risk` (2) — both tied to the model's own top two features, not arbitrary labels.

**How an editor would use this tomorrow:** open the 11 `prioritize_review` rows first, read each reason code, and — before doing anything — run the human-review checklist below. This is intentionally a short list: most of the test population (89%) sits in the ambiguous `monitor` tier, reported honestly rather than pushed into a more dramatic-looking bucket.

**Human review required before acting on any flagged row:**
- Consolidation check — did a sibling page absorb the demand?
- Seasonality check — does this topic naturally dip on a calendar?
- Noise check — is the volume large enough that this isn't just normal wobble?
- Position sanity check — does the underlying number actually support the reason code assigned?

**Never automated:** no auto-publishing from this queue; a `prioritize_review` tag is not proof of decline (Precision 0.64 means roughly 1 in 3 flagged pages were not actually declining); the queue never justifies removing a page or a client decision on its own.

## 8. Reproducibility

**Repo:** https://github.com/duashakeel0/flyrank-ml-internship — every notebook below is executed with real outputs visible on GitHub, not re-run copies.

| Notebook | What it proves |
|---|---|
| `work/notebooks/w01_research_question.ipynb` | Lane pick, backed by real numbers from the starter CSV |
| `work/notebooks/w02_ml_task_framing.ipynb` | Task type, target, metric, real dataframe |
| `work/notebooks/w03_data_contract.ipynb` | Data contract on real warehouse data; leakage trap performed live |
| `work/notebooks/w04_baseline_score.ipynb` | Two signal checks (one MIXED, one CONFIRMED); the rule baseline |
| `work/notebooks/w05_model.ipynb` | Model vs. baseline; point-in-time leakage caught and fixed; reproducibility bug caught and fixed |
| `work/notebooks/w06_validation_audit.ipynb` | Paper audit; before/after split proof; leakage re-test |
| `work/notebooks/w07_action_playbook.ipynb` | The ranked queue, reason codes, no-go list |
| `work/notebooks/capstone.ipynb` | This report, synthesized |

**Seeds:** `random_state=42` throughout (numpy `default_rng(42)` for the client split, sklearn `random_state=42` for both models). Reproducibility was not assumed — two real bugs that broke it were found and fixed (Week 5): DuckDB does not guarantee row order on a remote parquet scan without `ORDER BY`, which silently broke both a "seeded" shuffle and `argsort` tie-breaking. Fixed by sorting the client list before seeding, and using `kind="stable"` with a fixed row order. Verified identical across 3 independent full reruns.

**To reproduce:** clone the repo, `pip install -r requirements.txt`, generate a Hugging Face read token (after accepting the [`FlyRank/internship-warehouse`](https://huggingface.co/datasets/FlyRank/internship-warehouse) gate), set it as an `HF_TOKEN` environment variable (never hardcoded), then run any `work/notebooks/w0*.ipynb` top to bottom.

**Metrics receipts** (committed, not just claimed): `work/outputs/w03_*`, `w04_baseline_metrics.json`, `w05_model_metrics.json`, `w06_validation_audit_metrics.json`, `w07_action_playbook_metrics.json`.

## 9. Acknowledgments & data credit

Built on the FlyRank ML Internship dataset — [flyrank.ai](https://flyrank.ai).
