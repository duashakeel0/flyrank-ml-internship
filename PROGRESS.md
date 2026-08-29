# PROGRESS — Machine Learning Engineering Track

Certificate requirement: **5 ML assignments + the ML track capstone** (a single linear build, not a pool to choose 5 from — unlike the Front-end track).

Lane: **Refresh / Content Opportunity Scoring** — locked as of Week 4 (this week's card: "lanes lock this week"). Confirmed by four weeks of consistent evidence: real-number lane pick (W01), task framing (W02), a working data contract on real warehouse data (W03), and a baseline with one CONFIRMED signal (W04).

## Assignments

| # | Card | Week | Status | Notes |
|---|---|---|---|---|
| 1 | ML-01 — Run the Starter Notebooks | 1 | Done, not yet submitted | `notebooks/01_first_look_and_discovery.ipynb` + `02_your_first_readable_model.ipynb`, both executed top to bottom with "your turn" cells; done after 2-4 below (assignments arrived out of order), backfilled 2026-08-15 |
| 2 | ML-02 — Research Question and Provisional Lane | 1 | Done, not yet submitted | `work/notebooks/w01_research_question.ipynb`; lane picked with real numbers from starter CSV |
| 3 | ML-03 — Frame Your Lane as an ML Task | 2 | Done, not yet submitted | `work/notebooks/w02_ml_task_framing.ipynb`; ranking/scoring, Precision@50, real dataframe shown |
| 4 | ML-04 — Search Intelligence Data Contract | 3 | Done, not yet submitted | `work/notebooks/w03_data_contract.ipynb`; real warehouse data (month=2026-03), leakage trap performed live: honest AUC 0.599 vs leaky 1.000 |
| 5 | ML-07 — Baseline Action Score and Top-10 Review | 4 | Done, not yet submitted | `work/notebooks/w04_baseline_score.ipynb`; Signal 1 (staleness) MIXED, Signal 2 (position->CTR) CONFIRMED; 17/30,000 pages flagged; metrics in `work/outputs/w04_baseline_metrics.json` |

**5 of 5 required ML assignments done, capstone deployed and live** — certificate requirement fully met as of 2026-08-29. Ready to submit.

## Capstone roadmap

- [x] Phase 1 — Foundations (Weeks 1-4): ML-01, 02, 03, 04, 07 — all done above, lane locked
- [x] Phase 2 — Modeling (Week 5 / ML-08): `work/notebooks/w05_model.ipynb` — Random Forest (P@50=0.64) vs Logistic Regression (P@50=0.46) vs baseline (never fires, N/A) on real March 2026 warehouse data, client-holdout split. Caught + fixed real point-in-time leakage (dropped 81% of rows) and a real reproducibility bug (DuckDB remote row order). Metrics: `work/outputs/w05_model_metrics.json`
- [x] Phase 3 — Validation (Week 6 / ML-09): `work/notebooks/w06_validation_audit.ipynb` — audited 2 FlyRank paper findings (selection bias, unstated split strategy); before/after split test confirmed a real 20-point P@50 gap between naive random split (0.840) and honest client-holdout (0.640) on my own model; re-ran the leakage test on Week 5's final features (honest 0.685 vs leaky 0.936); found a new named limitation (population selection excludes recently-edited pages). Metrics: `work/outputs/w06_validation_audit_metrics.json`
- [x] Phase 4 — Action output (Week 7 / ML-10): `work/notebooks/w07_action_playbook.ipynb` — ranked queue built only from the validated client-holdout population (1,979 rows); reason codes tied to the model's own top features; action tiers (11 prioritize_review, 1,763 monitor, 205 no_action_needed — honestly reported concentration, not threshold-adjusted); human-review checklist + no-go list; retrain triggers. Figure + metrics: `work/outputs/figures/w07_action_tier_breakdown.png`, `work/outputs/w07_action_playbook_metrics.json`
- [x] Phase 5 — Capstone synthesis (Week 8 / ML-11 + ML-12): `work/capstone_report.md` (full 9-section report) + `work/notebooks/capstone.ipynb` (verifies every number against committed metrics, generates the 2 headline figures) + ML-12 closing cells (demo outline, social cut, employer summary). Deployed as `docs/index.html` via GitHub Pages, verified live (HTTP 200, all 3 charts loading)
- [x] Phase 6 — Submission: `submission/paper_url.txt` set to https://duashakeel0.github.io/flyrank-ml-internship/ — **capstone complete**

- **Deployed paper:** https://duashakeel0.github.io/flyrank-ml-internship/
- **Deliverable shape:** deployed public research paper + `submission/paper_url.txt` in the repo pointing at it — both done

## Data access

- Starter CSV (`data/raw/content_refresh_anonymized.csv`) — no auth needed, used Weeks 1-2
- Hugging Face warehouse (`FlyRank/internship-warehouse`) — gated, requires own account + READ token. Set up 2026-08-15: token stored as a Windows User environment variable (`HF_TOKEN`), read at runtime via `os.environ`, never hardcoded or committed
