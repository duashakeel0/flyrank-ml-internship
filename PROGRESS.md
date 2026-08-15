# PROGRESS — Machine Learning Engineering Track

Certificate requirement: **5 ML assignments + the ML track capstone** (a single linear build, not a pool to choose 5 from — unlike the Front-end track).

Lane: **Refresh / Content Opportunity Scoring** (provisional, can change until end of Week 4).

## Assignments

| # | Card | Week | Status | Notes |
|---|---|---|---|---|
| 1 | ML-01 — Run the Starter Notebooks | 1 | Done, not yet submitted | `notebooks/01_first_look_and_discovery.ipynb` + `02_your_first_readable_model.ipynb`, both executed top to bottom with "your turn" cells; done after 2-4 below (assignments arrived out of order), backfilled 2026-08-15 |
| 2 | ML-02 — Research Question and Provisional Lane | 1 | Done, not yet submitted | `work/notebooks/w01_research_question.ipynb`; lane picked with real numbers from starter CSV |
| 3 | ML-03 — Frame Your Lane as an ML Task | 2 | Done, not yet submitted | `work/notebooks/w02_ml_task_framing.ipynb`; ranking/scoring, Precision@50, real dataframe shown |
| 4 | ML-04 — Search Intelligence Data Contract | 3 | Done, not yet submitted | `work/notebooks/w03_data_contract.ipynb`; real warehouse data (month=2026-03), leakage trap performed live: honest AUC 0.599 vs leaky 1.000 |
| 5 | TBD | | Pending | |

## Capstone

- **Status:** Not started — builds incrementally through `work/notebooks/capstone.ipynb` as weeks progress
- **Deliverable shape:** deployed public research paper + `submission/paper_url.txt` in the repo pointing at it

## Data access

- Starter CSV (`data/raw/content_refresh_anonymized.csv`) — no auth needed, used Weeks 1-2
- Hugging Face warehouse (`FlyRank/internship-warehouse`) — gated, requires own account + READ token. Set up 2026-08-15: token stored as a Windows User environment variable (`HF_TOKEN`), read at runtime via `os.environ`, never hardcoded or committed
