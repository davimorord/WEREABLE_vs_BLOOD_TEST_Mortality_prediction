# WEARABLE_vs_BLOOD_TEST_Mortality_prediction

## Can a wrist‑worn accelerometer replace a blood test for predicting 10‑year mortality?

**Author:** David Moreno  
**Project type:** Master's thesis / Health data science portfolio  
**Data source:** NHANES 2003‑2004 (n = 2,755, 216 events)  
**Methods:** XGBoost, threshold analysis, SHAP (planned)

---

## Device note: NHANES accelerometer vs modern smartwatches

The accelerometry data in NHANES 2003‑2004 was collected using an **ActiGraph model 7164**. This is a uniaxial hip‑worn device that records vertical acceleration in one‑minute epochs. Participants wore it for seven consecutive days, removing it only for sleep and water activities.

Modern smartwatches differ in three key ways:

- **Triaxial measurement**: They capture movement in three planes (vertical, horizontal, and lateral) instead of just vertical.
- **Wrist placement**: This improves compliance and captures upper body movements that hip‑worn devices miss.
- **Additional sensors**: Optical heart rate monitors, accelerometers, and sometimes electrodermal activity provide complementary physiological signals.

These technological advances would likely improve predictive performance. Triaxial measurement captures a wider range of activities (cycling, upper body movement). Wrist placement increases wear time compliance, reducing non‑wear misclassification. Additional sensors could provide independent risk information beyond raw movement counts. Therefore, the results presented here probably **underestimate** the predictive potential of current consumer wearables.

---

## The problem

Most cardiovascular risk tools (Framingham, ACC/AHA) require a blood draw. That means a clinic visit, a needle, a lab, and days of waiting. Millions of people never get tested – they do not see a doctor, live in low‑resource settings, or simply do not know they are at risk.

A wrist‑worn accelerometer is cheap, passive, and already on millions of wrists. But can it actually predict who will die from natural causes in the next ten years, without a single blood test?

This project answers exactly that question.

---

## What I did

I trained three XGBoost models on NHANES 2003‑2004 (adults 20‑74 years, 10‑year natural mortality, excluding accidents):

| Model | Predictors |
|-------|-------------|
| **A (Blood test)** | age, sex, bmi, sbp, total_chol, hdl, non_hdl, hba1c |
| **B (Wearable‑only)** | age, sex, bmi, TLAC_mean, sedentary_mean, mvpa_mean, valid_days |
| **C (Combined)** | all of the above |

**Key choice:** I kept `age`, `sex`, and `bmi` in all models. The comparison isolates the *incremental value* of laboratory markers versus accelerometry features, while controlling for basic demographics.

---

## Key results

| Model | AUC | Sensitivity | Specificity |
|-------|-----|-------------|-------------|
| Blood test | 0.822 | 64.8% | 81.8% |
| Wearable‑only | **0.828** | 65.8% | 81.6% |
| Combined | 0.842 | 65.3% | 86.2% |

**The wearable‑only model performs comparably to the full laboratory panel (AUC 0.828 vs 0.822).**  
At default thresholds, both miss about one third of deaths – not enough for clinical deployment.  
But by lowering the threshold to 0.2‑0.3, the wearable model achieves **100% sensitivity** (no death missed), at the cost of 32‑40 false positives per 100 alive individuals.

---

## Clinical takeaway

A wearable device is **not ready to replace blood tests** for definitive risk stratification.  
However, it **can serve as a low‑cost, non‑invasive prescreening tool** in:

- Resource‑limited settings (no lab, no phlebotomist)
- Community‑based screening programmes (initial triage)
- Remote monitoring (chronic disease management)

**Proposed workflow:**  
Wearable prescreening (threshold 0.2‑0.3) → high‑risk individuals referred for confirmatory blood testing → definitive clinical assessment.

---

## Feature importance (combined model)

| Domain | Key variables | Combined importance |
|--------|---------------|---------------------|
| Demographic | age, sex | 31.2% |
| Wearable | TLAC_mean, sedentary_mean, mvpa_mean | **29.9%** |
| Laboratory | hba1c, hdl, sbp, bmi, lipids | 38.9% |

Accelerometry captures nearly one‑third of the model's predictive power – even when clinical and laboratory data are available. This confirms that physical activity patterns carry mortality risk information that traditional biomarkers do not fully explain.

---

## Limitations

| Limitation | Impact |
|------------|--------|
| Single NHANES cycle (2003‑2004) | Temporal validity unknown |
| No external validation | Results may not generalise to other cohorts |
| Moderate event count (n = 216) | Limited statistical power for complex interactions |
| Default thresholds used for training | Sensitivity can be improved with cost‑sensitive learning |
| **No sensitivity optimisation** | The 65% sensitivity is a lower bound, not a ceiling |
| **Accelerometer technology gap** | NHANES used a uniaxial hip‑worn device (ActiGraph 7164). Modern smartwatches are triaxial, wrist‑worn, and include additional sensors (heart rate, etc.). Results likely **underestimate** current wearable potential. However, direct application to modern devices would require calibration or dataset adaptation. |

**On sensitivity optimisation:**  
No explicit effort was made to maximise sensitivity (e.g., lowering decision threshold or using cost‑sensitive learning). The reported sensitivities reflect *baseline performance* at threshold 0.5. For any real‑world clinical application, the threshold would be adjusted to prioritise sensitivity – trading off specificity.

---

## Future work

**External validation.** The most natural next step is validating the model on NHANES 2005‑2006, which used the same accelerometer model and mortality linkage protocol. A stronger validation would use NHANES 2011‑2014 (wrist‑worn triaxial accelerometer), testing both temporal and technological generalisability.

**SHAP analysis.** Adding SHAP (SHapley Additive exPlanations) would make the model more transparent, showing which features drive predictions for individual patients. This is particularly useful for clinical adoption.

**MLOps deployment.** The model could be deployed as an endpoint using Azure ML, with experiment tracking via MLflow and a simple API for real‑time risk scoring.