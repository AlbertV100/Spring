# Capstone 2 — TF‑IDF vs Transformer Sentiment Analysis (TweetEval)

**Author:** Albert Vempala  
**Program:** Springboard Data Science Career Track  
**Deliverables:** Notebook • Metrics CSV • Final Report PDF • Executive Slide Deck

---

## Executive Summary (Decision)

This capstone evaluates whether a modern **fine‑tuned Transformer** materially outperforms a strong classical baseline (**TF‑IDF + Logistic Regression**) for **tweet sentiment classification** (negative / neutral / positive).

**Bottom line:** The Transformer is the recommended production model, delivering a **test Accuracy lift of +0.1176** and **Macro‑F1 lift of +0.1229** over the TF‑IDF baseline (strict notebook‑derived).

---

## Business Problem

Sentiment classification supports **customer experience**, **brand monitoring**, and **product decision support**.  
Bag‑of‑words methods (TF‑IDF) often miss **context**, especially for:
- negation (“not good”)
- sarcasm / irony
- word order effects and compositional meaning
- ambiguous short tweets

This project quantifies that gap with a consistent split protocol and class‑balanced evaluation.

---

## Data (Facts)

- **Dataset:** TweetEval / sentiment (3 classes: negative, neutral, positive)  
- **Splits used (notebook):** Train **45,615** | Validation **2,000** | Test **12,284**  
- **Data QA (notebook):** Missing values and duplicate texts checked prior to modeling

---

## Methodology (Rubric‑Aligned)

1. **EDA**: class balance + tweet length distributions + key text characteristics  
2. **Baseline**: TF‑IDF vectorization + Logistic Regression with validation tuning  
3. **Transformer**: fine‑tune contextual model for TweetEval sentiment  
4. **Evaluation**: Accuracy + Macro Precision / Recall / F1 (macro emphasizes class balance)  
5. **Interpretation**: confusion matrices + example‑driven error analysis  
6. **Recommendation**: select model based on measurable lift + operational practicality

---

## Models

### Model A — TF‑IDF + Logistic Regression (Baseline)
- TF‑IDF: word n‑grams (1,2), sublinear TF, `min_df=2`, `max_df=0.95`
- Logistic Regression: `class_weight=balanced`, `max_iter=3000`
- Validation tuning: grid search on regularization strength **C** → **Best C = 2.0** (notebook)

### Model B — Transformer (Fine‑tuned)
- Fine‑tuned for 3‑class sentiment classification
- Strength: context/semantics (word order, modifiers, negation), fewer boundary errors

---

## Results (Strict Notebook‑Derived Metrics)

Source file: `metrics/Capstone_2_Model_Metrics_STRICT_Notebook_Derived.csv`

| Model | Split | Accuracy | Macro Precision | Macro Recall | Macro F1 |
|---|---:|---:|---:|---:|---:|
| TF‑IDF + Logistic Regression | Validation | 0.6675 | 0.6376 | 0.6615 | 0.6455 |
| TF‑IDF + Logistic Regression | Test | 0.5947 | 0.5858 | 0.5996 | 0.5911 |
| Transformer (Fine‑tuned) | Test | **0.7123** | **0.7040** | **0.7303** | **0.7140** |

### Lift (Transformer vs TF‑IDF on Test)
- **Accuracy:** 0.5947 → 0.7123 (**+0.1176**)  
- **Macro Precision:** 0.5858 → 0.7040 (**+0.1182**)  
- **Macro Recall:** 0.5996 → 0.7303 (**+0.1307**)  
- **Macro F1:** 0.5911 → 0.7140 (**+0.1229**)  

**Interpretation:** The largest gain is in **Macro Recall** (fewer misses across classes), which is important for business use because it reduces the chance of missing strong positive/negative signals.

---

## Key Figures (From Notebook Outputs)

Place these images under `figs/` so GitHub renders them in-line:

- Benchmark comparison: `figs/benchmark_clean.png`  
- Confusion matrices (test): `figs/confusion_matrices_side_by_side.png`  
- EDA label distribution: `figs/cell011_out00.png`  
- EDA token length distribution: `figs/cell011_out01.png`  

---

## Recommendations (Executive‑Ready)

1. **Deploy the fine‑tuned Transformer** for decision‑critical sentiment intelligence.  
2. **Retain TF‑IDF baseline** for low‑cost fallback and regression testing.  
3. **Operationalize with monitoring**: track macro metrics over time and refresh if drift occurs.

---

## Repository Structure (Suggested)

```
.
├── notebooks/
│   └── TweetEval_TFIDF_vs_Transformer_ProductionReady.ipynb
├── report/
│   └── Capstone_2_TFIDF_vs_Transformer_Final_Report_Story.pdf
├── slides/
│   └── Capstone_2_TFIDF_vs_Transformer_Executive_Deck_19_Strict_Notebook_Figures.pptx
├── metrics/
│   ├── Capstone_2_Model_Metrics_STRICT_Notebook_Derived.csv
└── figs/
    ├── benchmark_clean.png
    ├── confusion_matrices_side_by_side.png
    ├── cell011_out00.png
    └── cell011_out01.png
```

---

## How to Run

- Open and run: `notebooks/TweetEval_TFIDF_vs_Transformer_ProductionReady.ipynb`
- Typical dependencies: `pandas`, `numpy`, `scikit-learn`, `matplotlib`, `transformers`, `datasets`, `torch`

---

## Deliverables (Mentor Review)

- **Report (PDF):** `report/Capstone_2_TFIDF_vs_Transformer_Final_Report_Story.pdf`
- **Slides (PPTX):** `slides/Capstone_2_TFIDF_vs_Transformer_Executive_Deck_19_Strict_Notebook_Figures.pptx`
- **Metrics (CSV):** `metrics/Capstone_2_Model_Metrics_STRICT_Notebook_Derived.csv`
- **Notebook:** `notebooks/TweetEval_TFIDF_vs_Transformer_ProductionReady.ipynb`

---

**Attribution:** TweetEval is a public NLP benchmark dataset. This repository is for educational/portfolio use.
