# Understanding Long-Term ODSP Reliance Across Ontario CMAs (2003–2024)
**A regional analysis of how long people stay on the Ontario Disability Support Program (ODSP), and a short-term forecast of where average duration is likely to stay highest**

---

## 📌 Purpose

The Ontario Disability Support Program (ODSP) provides income and employment support to Ontarians with long-term disabilities. System pressure is shaped not only by how many people receive assistance, but by how long they remain on it.

This project examines structural patterns of long-term ODSP reliance across Ontario's 15 Census Metropolitan Areas (CMAs) over two decades, using administrative data to identify which regions face the highest risk of sustained long-term dependency and what factors drive regional differences.


## 🛢️ Data

- **Source:** Ontario Social Assistance case characteristics by census metropolitan area
Government of Ontario, Open Data Portal
https://data.ontario.ca/dataset/ontario-social-assistance-case-characteristics-by-census-metropolitan-area
- **Coverage:** 15 Ontario CMAs, 2003–2024. Quarterly snapshots (March, June, September, December) aggregated to CMA-year. 2025 is incomplete and excluded.
- **Caseload index:** `total_cases` is the sum of four quarterly snapshots. It is an index of caseload, not a count of unique people.
- **Average duration (estimated):** the data only report three duration groups (0–35, 36–59, 60+ months). I assigned typical values of 17.5, 47.5 and 72 months and took the case-weighted mean. The 60+ group has no upper limit, so 72 is an assumption. Valuing it at 84 or 96 months leaves the CMA ranking essentially unchanged (only Kingston and Thunder Bay swap places), but the levels in months depend on it.
- **Checks:** household, age, duration and family-size categories add exactly to total cases; gender differs by at most 8 cases in one CMA-year. `family_size_1` is identical to single-adult households, so only one is used in models.
- All analysis is at the CMA level. No individual-level data.

## Methods

- **Descriptive:** system-wide trends, a four-way CMA segmentation (median split on average duration and cumulative contribution), and CMA-level correlations with demographic shares.
- **Prediction:** average duration in year *t* predicted from durations in *t-1*, *t-2*, *t-3* and demographic shares lagged one year.
- **Splits (chronological):** train 2006–2019, validation 2020–2021 (tuning only), test 2022–2024.
- **Models:** a naive baseline ("next year equals this year"), linear regression, and Elastic Net (alpha chosen on validation data, lambda by cross-validation).
- **Diagnostics:** Breusch-Pagan and Durbin-Watson tests on the linear model; standard errors clustered by CMA.

## Findings

### 1–2. Duration and caseload

Average duration was not a steady rise. It fell from 55.9 months (2003) to 53.3 (2010), rose to 57.9 (2016), fell back to 56.4 (2019), then rose sharply to 60.7 (2023) and edged down to 60.0 (2024).

The caseload index peaked in 2020 and has been slightly lower since. From 2019 to 2022 the share of cases in the 0–35 month group fell from 23.4% to 15.5%, while the 60+ share rose from 64.9% to 70.7% (73.0% in 2024). This is consistent with fewer recent entrants, longer stays, or both; the data cannot separate them.

![Average duration over time](figures/q1_duration_trend.png)
![Caseload over time](figures/q2_caseload_trend.png)

### 3. Regional segments

| CMA | Avg. duration (months, 2003–2024) | Contribution (millions) | Segment |
|---|---|---|---|
| Greater Sudbury | 59.2 | 29.4 | Low Contribution – High Duration |
| Kingston | 58.7 | 30.0 | Low Contribution – High Duration |
| Thunder Bay | 58.7 | 24.0 | Low Contribution – High Duration |
| St. Catharines-Niagara | 58.0 | 66.3 | High Contribution – High Duration |
| Brantford | 58.0 | 24.5 | Low Contribution – High Duration |
| Hamilton | 57.9 | 99.4 | High Contribution – High Duration |
| Peterborough | 57.9 | 20.5 | Low Contribution – High Duration |
| Ottawa-Gatineau | 57.5 | 109.1 | High Contribution – Lower Duration |
| London | 56.9 | 68.4 | High Contribution – Lower Duration |
| Windsor | 56.6 | 47.8 | High Contribution – Lower Duration |
| Oshawa | 56.5 | 38.6 | Low Contribution – Lower Duration |
| Guelph | 56.2 | 13.8 | Low Contribution – Lower Duration |
| Kitchener-Cambridge-Waterloo | 55.9 | 48.3 | High Contribution – Lower Duration |
| Barrie | 55.7 | 19.5 | Low Contribution – Lower Duration |
| Toronto | 55.1 | 476.0 | High Contribution – Lower Duration |

Duration is estimated (see Data). Contribution is average duration × caseload index, summed over 2003–2024.

Toronto's contribution (about 476M) is several times larger than any other CMA's, but its average duration is the lowest. Ottawa-Gatineau (duration) and Oshawa (contribution) sit exactly on a median, so their placement is borderline.

![CMA segmentation](figures/q3_segmentation.png)

### 4. Demographics

Household and age mix are similar across segments (single adults about 78–80% everywhere). Across 15 CMAs, only the 25–54 age share correlates significantly with average duration (r = 0.60, p = 0.018), which is one result out of nine uncorrected tests and should be read as descriptive.

![Correlation](figures/q4_correlation_scatter.png)
![Household composition](figures/q4_household_composition.png)
![Age composition](figures/q4_age_composition.png)

### 5. Prediction and forecast

| RMSE (months) | Train | Validation | Test |
|---|---|---|---|
| Naive (last year) | 0.717 | 1.298 | 0.887 |
| Linear regression | 0.659 | 1.234 | 0.930 |
| Elastic Net | 0.537 | 0.968 | 0.481 |

Elastic Net beat the naive forecast in each of the three test years (2022: 0.68 vs 1.33; 2023: 0.29 vs 0.45; 2024: 0.38 vs 0.63). The linear model's errors are heteroscedastic (Breusch-Pagan p ≈ 0.01) and strongly autocorrelated (Durbin-Watson 0.95), so I report CMA-clustered standard errors.

The Elastic Net coefficients are best read as a level term (about 0.91) plus extrapolation of the most recent year-over-year change, i.e. momentum. Adding demographics improved validation error (1.05 → 0.97) but worsened test error (0.45 → 0.48), so they add no consistent predictive value. This is a statement about prediction, not about causes.

![Alpha selection](figures/alpha_selection.png)
![Predicted vs actual](figures/predicted_vs_actual.png)
![Feature importance](figures/feature_importance.png)
![Residual diagnostics](figures/lm_residual_diagnostics.png)

**2025 forecast (ranking, not a level forecast).** Highest predicted average duration: Kingston, Peterborough, Thunder Bay, Greater Sudbury, St. Catharines-Niagara. The gaps between these and the next CMAs are small, and 14 of 15 CMAs are predicted to dip slightly, mirroring 2024's decline.

| CMA | 2024 actual (months) | 2025 predicted (months) | Change |
|---|---|---|---|
| Kingston | 63.0 | 62.7 | −0.25 |
| Peterborough | 62.5 | 62.1 | −0.47 |
| Thunder Bay | 62.0 | 61.3 | −0.77 |
| Greater Sudbury | 61.6 | 61.2 | −0.43 |
| St. Catharines-Niagara | 61.7 | 61.1 | −0.67 |
| Oshawa | 60.9 | 60.8 | −0.09 |
| Brantford | 61.2 | 60.6 | −0.63 |
| Hamilton | 60.8 | 60.4 | −0.40 |
| Ottawa-Gatineau | 60.4 | 59.8 | −0.55 |
| Windsor | 60.2 | 59.8 | −0.47 |
| Barrie | 59.9 | 59.7 | −0.24 |
| Guelph | 59.2 | 59.3 | +0.08 |
| Kitchener-Cambridge-Waterloo | 59.6 | 59.1 | −0.49 |
| Toronto | 59.2 | 58.7 | −0.53 |
| London | 58.8 | 58.3 | −0.57 |

Predicted changes from 2024 are small (between −0.8 and +0.1 months, average about −0.4), which is within the model's typical test error of about 0.5 months. The forecast is therefore best read as a ranking of where average duration is likely to remain highest, not as a prediction that any region will improve or worsen.

![2025 forecast](figures/cma_risk_forecast.png)

## Limitations

- 15 regions and 19 usable years; observations within a region are strongly related, so precision is limited. Bootstrapping over CMAs gives a 39–52% error reduction versus the naive forecast, but that reflects variation across regions only, not across future years, and is likely too narrow.
- Only three test years, and the validation period (2020–2021) is unusual.
- Duration is estimated from three groups with an assumed value for the open-ended group.
- Regional data only: nothing here applies to individuals, and results are associations and short-term predictions, not causal claims.
- The forecast is one-step-ahead extrapolation and does not account for policy or economic change.
- - The penalty (lambda) was chosen with standard 10-fold cross-validation inside the training years, which uses random rather than time-ordered folds and may be slightly optimistic for correlated years. The mixing parameter was chosen on a separate validation period, and the 2022–2024 test years were never used for tuning.

## Reproduce

1. Put `ODSP_CMA.csv` in the same folder as `ODSP_Reliance_Across_Ontario.Rmd`.
2. Install: `dplyr, tidyr, ggplot2, readr, stringr, glmnet, lmtest, sandwich, ggrepel, scales, knitr, rmarkdown`.
3. Knit the Rmd. Figures are written to `figures/`.

## Files

- `ODSP_Reliance_Across_Ontario.Rmd` – full analysis
- `ODSP_Reliance_Across_Ontario.html` – knitted output
- `figures/` – plots used above
