# Relax Inc. Take-Home — User Adoption Analysis

This repo contains a concise analysis of **user adoption** for Relax Inc., based on two provided datasets:

- `takehome_users.csv` — user attributes and acquisition source  
- `takehome_user_engagement.csv` — daily login events (one row per user-day)

The full workflow (EDA → experiment/metrics design → modeling) is in:
- `Relax_Takehome_User_Adoption_Analysis.ipynb`

---

## Adoption Definition

An **adopted user** is one who logged in on **3 distinct days** within **any 7-day window**.

Implementation notes:
- Normalize engagement timestamps to **calendar days**
- Deduplicate to **unique user-day** logins
- For each user, compute a **7-day rolling count** of distinct login days; adopted if any window reaches **≥ 3**

Core logic sketch:

```python
# per user: unique login days -> adopted if any 7-day window contains >=3 days
dates = sorted(unique_login_days[user])
adopted = any(count_in_last_7_days(d) >= 3 for d in dates)
```

---

## Key Findings (EDA)

### Overall adoption
- **Adoption rate:** **13.4%** (1,602 adopted / 12,000 total)

### Adoption by acquisition channel (`creation_source`)
| creation_source | n_users | adoption_rate_% |
|---|---:|---:|
| SIGNUP_GOOGLE_AUTH | 1,385 | 16.8 |
| GUEST_INVITE | 2,163 | 16.6 |
| SIGNUP | 2,087 | 14.0 |
| ORG_INVITE | 4,254 | 13.0 |
| PERSONAL_PROJECTS | 2,111 | 7.8 |

### Adoption by invitation status
| was_invited | n_users | adoption_rate_% |
|---:|---:|---:|
| 0 | 5,583 | 12.3 |
| 1 | 6,417 | 14.2 |

### Adoption by organization size (binned)
| org_size_bin | n_users | adoption_rate_% |
|---|---:|---:|
| 3–5 | 59 | 15.3 |
| 6–10 | 4,538 | 15.7 |
| 11–25 | 4,001 | 14.3 |
| 26–50 | 1,707 | 10.3 |
| 51–100 | 940 | 9.9 |
| 101+ | 753 | 5.3 |

**Interpretation (associational, not causal):**
- Adoption differs meaningfully by **creation_source**.
- **Invited users** show a higher adoption rate than non-invited users.
- **Mid-sized orgs** (6–25 users) show higher adoption than very large orgs in this dataset.

---

## Experiment + Metrics Design (to increase adoption)

**Hypothesis:** Improving onboarding for new users—especially invitees—will increase adoption in the first month.

**Design**
- **Randomization unit:** **organization-level** (reduces spillover within teams)
- **Primary metric:** **28-day adoption rate** (same definition, measured within first 28 days after signup)
- **Secondary diagnostics:** time-to-3rd active day, week-2/week-4 retention, activation funnel events (if available)
- **Guardrails:** unsubscribe / notification opt-outs, support burden, errors/latency

---

## Predictive Modeling (signup-time, non-leaky baseline)

**Objective:** Predict future adoption using only information available at signup (avoid leakage).

**Features used**
- `creation_source`
- `org_size` (derived from `org_id` counts)
- `was_invited` (flag from `invited_by_user_id`)
- `opted_in_to_mailing_list`
- `enabled_for_marketing_drip`

**Holdout results (25% test split, stratified)**
| Model | ROC-AUC | PR-AUC | Precision@0.5 | Recall@0.5 |
|---|---:|---:|---:|---:|
| Logistic Regression (class_weight=balanced) | 0.618 | 0.196 | 0.162 | 0.762 |

**Takeaway:** With only acquisition + org context features, predictive power is **modest** but can still be useful for **ranking/targeting** (e.g., which new users should receive high-touch onboarding).

---

## How to Run

1. Create an environment (example):
   ```bash
   python -m venv .venv
   source .venv/bin/activate  # Windows: .venv\Scripts\activate
   pip install -U pandas numpy matplotlib scikit-learn jupyter
   ```
2. Place the data files in the repo root:
   - `takehome_users.csv`
   - `takehome_user_engagement.csv`
3. Launch:
   ```bash
   jupyter notebook
   ```
4. Open `Relax_Takehome_User_Adoption_Analysis.ipynb` and run all cells.

---

## Repo Contents

- `Relax_Takehome_User_Adoption_Analysis.ipynb` — full analysis (EDA, experiment design, modeling)
- `README.md` — this summary
