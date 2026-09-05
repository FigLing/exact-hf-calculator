# EXACT-HF Bedside Risk Calculator

A ten-variable calculator estimating 30-day and 1-year all-cause mortality after hospitalisation
for heart failure. Runs entirely in the browser — no server, no data leaves the page.

> ## ⚠️ Research use only
>
> This model has **not** been prospectively or interventionally validated and is **not a medical
> device**. It must not be used alone to guide care for an individual patient. It was developed in
> a predominantly HFrEF, multi-ethnic Singaporean cohort; performance in other health systems, and
> in HFpEF, is untested.

## The model

Elastic-net logistic regression (L1 ratio 0.5, 5-fold CV) on the ten predictors that a SHAP-ranked
minimal-feature analysis found sufficient to reproduce full-model 30-day discrimination.

| | 30-day | 1-year |
|---|---|---|
| External AUROC (95% CI) | **0.862** (0.834–0.889) | **0.739** (0.724–0.755) |
| Brier score | 0.031 | 0.155 |
| Calibration slope | 0.94 | 0.93 |
| Youden threshold | 3.5% | 17.7% |

External validation: 5,016 patients across five hospitals with no patient or site overlap with
development (235 thirty-day and 1,208 one-year deaths).

For context, the full 85-variable models reach 0.862 (XGBoost) and 0.864 (logistic) at 30 days —
**the ten-variable model gives up essentially nothing at 30 days**, and is better calibrated than
the full XGBoost model at one year (slope 0.93 versus 0.72). At one year it does trade some
discrimination for portability (0.739 versus 0.757), though it still exceeds recalibrated MAGGIC
(0.699).

### Inputs

Serum sodium · age · serum urea · BMI · mean arterial pressure · diastolic blood pressure ·
NYHA class · beta-blocker, loop diuretic and statin at discharge.

A blank field falls back to the training-set median, which is exactly how the model handled
missing data during development. Values outside the training range are winsorised to it, and the
interface says so when it happens.

### On the medication inputs

Beta-blocker and loop-diuretic prescription are among the strongest predictors in the model, but
they are **prognostic markers, not causal levers**. Non-prescription frequently reflects a
clinical judgement that the patient could not tolerate the drug — acute kidney injury, hypotension,
bradycardia — rather than an untreated opportunity. Toggling them in the calculator does not
estimate the effect of prescribing them.

This matters quantitatively: removing all medication variables drops external 30-day AUROC to
0.774, which overlaps recalibrated MAGGIC (0.754). At 30 days the medication block carries most of
the model's advantage over established scores.

## Guideline-directed therapy audit

The page includes a four-pillar GDMT checklist that is deliberately **separate from the risk
model** and does not feed it. Among externally validated high-risk patients with LVEF <40%, a mean
of 3.08 of the four pillars were absent, against 2.13 across all HFrEF patients — and 36.3% were
receiving none of the four, versus 2.0% of model-flagged low-risk patients.

Most non-prescription in the source cohort had a documented renal or haemodynamic contraindication,
so a flagged gap is a prompt to check, not a prescription to write.

## How it works

`model.json` carries the coefficients along with everything needed to reproduce the training
pipeline's preprocessing — imputation medians, winsorising bounds, and scaler constants. The page
applies them in the same order as training: impute → winsorise → standardise → linear predictor →
logistic.

This is verified, not assumed: the browser arithmetic reproduces scikit-learn's `predict_proba` to
4.4e-16 across all 5,016 external validation patients.

```
files
├── index.html   the calculator (no build step, no dependencies)
├── model.js     model.json wrapped for <script> loading, so file:// works
└── model.json   canonical coefficients + preprocessing constants
```

## Running locally

Open `index.html` directly, or serve the directory:

```bash
python -m http.server 8000
```

## Regenerating the model

From the analysis repository:

```bash
python step28_export_calculator_model.py
```

This refits the ten-variable model, writes `model.json` and `model.js` here, and records
performance in `tables/TableS13_calculator_model.csv`. Do not edit `model.js` by hand.

## Citation

Manuscript under review. Citation to follow.

## Licence

MIT — see [LICENSE](LICENSE). The licence covers the code; it is not a warranty of clinical
fitness, and the research-use restriction above still applies.
