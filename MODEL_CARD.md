<p align="left">
• <a href="https://github.com/hellomayzune"><strong>GitHub</strong></a> •
<a href="https://orcid.org/0000-0003-0282-2633"><strong>ORCID</strong></a> •
<a href="https://scholar.google.com/citations?user=LmP8B_4AAAAJ&hl=en"><strong>Google Scholar</strong></a> •
<a href="https://www.researchgate.net/profile/May-Zune"><strong>ResearchGate</strong></a> •
<a href="https://www.linkedin.com/in/mayzune/"><strong>LinkedIn</strong></a> •
<a href="https://mayzune.com/"><strong>Blog</strong></a> •

</p>

<p align="left">
📄 Companion to <a href="./README.md"><strong>README.md</strong></a> •
<a href="./DATASHEET.md"><strong>DATASHEET.md</strong></a> •
<a href="./ADR.md"><strong>ADR.md</strong></a>
</p>

# 🧠 HAVEN — MODEL CARD

This document presents model cards for the six analytical components implemented in the **HAVEN** (Human-centred Assessment of Vulnerability to Environmental heat at Night) notebooks, following the reporting principles proposed by **Mitchell et al. (2019)** in *Model Cards for Model Reporting*. Rule-based indicators are documented alongside statistical models because downstream analytical results depend on both.

Each model card documents the component's specifications, intended uses, relevant factors, evaluation metrics, data sources and limitations. In accordance with `ADR-007`, numerical results are not reproduced in this document; instead, they are reported in the files listed under *Quantitative analyses*.

📖 Contents

- [Model 1 — Night-time exposure and TM59 indicators](#-model-1--night-time-exposure-and-tm59-indicators)
- [Model 2 — Indoor heat episodes and event envelopes](#-model-2--indoor-heat-episodes-and-event-envelopes)
- [Model 3 — Recovery indicators and their reliability](#-model-3--recovery-indicators-and-their-reliability)
- [Model 4 — Ridge multi-horizon forecaster and baselines](#-model-4--ridge-multi-horizon-forecaster-and-baselines)
- [Model 5 — Hot-night warning layer](#-model-5--hot-night-warning-layer)
- [Model 6 — Comparator models used as sensitivity analyses](#-model-6--comparator-models-used-as-sensitivity-analyses)

[Back to README](./README.md#️-haven-human-centred-assessment-of-vulnerability-to-environmental-heat-at-night)

---

# 🌙 Model 1 — Night-time exposure and TM59 indicators

### Model details
- **Notebook:** `05_resilience_metrics_and_evaluation.ipynb` (Sections 1, 2, 4, 8c, 11, 13); functions in `src/metrics.py` and `src/events.py`.
- **Type:** deterministic, rule-based indicators computed from measured hourly temperatures; not fitted.
- **Rules:**
  - Night = 22:00–07:00, labelled by its start date; a night is evaluated only when all nine hours are measured (`ADR-004`).
  - **I_night** = Σ max(0, T − 26 °C) over the night (°C·h); exceedance hours; exceedance night (at least one hour above 26 °C); no-relief night (a complete night that never falls to 26 °C).
  - **CIBSE TM59:2026 Criterion (b)** (bedrooms): 23:00–08:00, May–September; nights with at least 6 readings evaluated; failure when the night mean exceeds 27 °C (Category II) or 26 °C (Category I); pass with no more than 4 failure nights per season (`ADR-029`).
  - **Year effect:** Year effects on nightly dwelling-level I_night are estimated with outdoor heat held fixed, using regression models incorporating outdoor night-time degree-hours above 18 °C, their squared term, the preceding day's maximum outdoor temperature and year indicators. Both least-squares and Poisson-family models are fitted, with heteroscedasticity and autocorrelation-consistent (HAC) standard errors (`ADR-031`).
  - **Sensitivity:** thresholds of 24, 25 and 27 °C; operative-minus-air offsets of −0.06 and −0.15 K (`ADR-030`).
- **References:** CIBSE TM59 (2017, 2026); CIBSE Guide A (2021).

### Intended use
- **Uses:** identifying which rooms accumulate night-time heat, when, and how robustly; relating the study indicators to TM59.
- **Users:** the study authors; researchers and practitioners looking for a worked example of night-time indicators from monitoring data.
- **Out of scope:** compliance with TM59 or Approved Document O; health or sleep outcomes; other dwellings without checking sensor placement.

### Factors
Room (seven indoor zones, three bedrooms), floor, orientation, year and threshold. Results are reported by room and never pooled across dwellings.

### Metrics
I_night, exceedance hours and nights, no-relief nights and the longest run, TM59 failure nights and pass/fail by bedroom-season, year effects with 95% intervals, rank stability across thresholds (Spearman ρ), split-half reliability of the room ranking (Model 3), and agreement with conventional indicators.

### Data
Dataset A, measured values only, full record. No train/test split.

### Quantitative analyses
`exposure_and_recovery_by_zone.csv`, `night_exposure_by_bedroom_and_year.csv`, `bedroom_exposure_common_nights.csv`, `exposure_and_outdoor_heat_by_year.csv`, `exposure_year_effects.csv`, `tm59_criterion_b_bedrooms.csv`, `tm59_failure_nights.csv`, `threshold_sensitivity.csv`, `operative_offset_sensitivity.csv`, `indicator_agreement.csv`, `room_profile.csv`.

### Ethical considerations
Temperature and humidity from one private home; no occupant-identifying, health or demographic data.

### Caveats and recommendations
- Air temperature is a proxy for operative temperature; read results with the operative-offset sensitivity.
- The 26 °C threshold is a screening level, not a health threshold.
- The relationship between outdoor heat conditions and I_night is non-linear. Comparisons between years should therefore account for differences in outdoor heat exposure.

[Back to top](#-haven--model-card)

---

# 📅 Model 2 — Indoor heat episodes and event envelopes

### Model details
- **Notebooks:** `02_exploratory_data_analysis.ipynb` (heatwave catalogues) and `05_resilience_metrics_and_evaluation.ipynb` (Sections 3, 5, 9, 12).
- **Type:** deterministic event definitions; not fitted.
- **Rules:**
  - **Outdoor heatwave:** at least three consecutive days with a daily maximum of at least 27 °C (Met Office, Nottinghamshire), applied to the on-site logger and to the MIDAS station (`ADR-012`).
  - **Indoor heat episode:** A sequence of at least three consecutive complete nights during which each bedroom records at least one hourly air temperature above 26 °C. Indoor episodes are matched to outdoor heatwaves using a one-day temporal tolerance (`ADR-025`).
  - **Event envelope:** For each episode and monitored zone, the event envelope describes the sequence of consecutive exceedance nights, their temporal relationship to the corresponding outdoor heatwave, the peak indoor temperature and its lag relative to the outdoor maximum, cumulative exposure during the episode, and the proportion of exposure occurring outside the heatwave period.
  - **Night classes:** lead-in (up to three nights before a heatwave), during, tail (up to three nights after) and outside; and, where both records exist, heatwave nights in both catalogues, one or neither.

### Intended use
- **Uses:** comparing the timing of indoor heat with outdoor heatwaves; showing how the choice of outdoor record changes the exposure attributed to heatwaves.
- **Out of scope:** forecasting episodes; official heatwave declarations.

### Factors
Episode, room, outdoor record (on-site or station), year, threshold, minimum nights and number of bedrooms required.

### Metrics
Episode count, length and exposure; matching to heatwaves; start and end of each room's run relative to the heatwave; peak lag; shares of exposure by night class and catalogue agreement; sensitivity of the episode set to the definition.

### Data
Dataset A (measured values) and Dataset B (2023–2025).

### Quantitative analyses
`indoor_episode_catalogue.csv`, `heatwave_catalogue_onsite.csv`, `heatwave_catalogue_station.csv`, `heatwave_matching_onsite_station.csv`, `heatwave_episode_matching.csv`, `event_envelopes_by_room.csv`, `event_timing_by_room.csv`, `exposure_on_heatwave_nights.csv`, `exposure_by_heatwave_phase.csv`, `exposure_by_catalogue_agreement.csv`, `episode_definition_sensitivity.csv`, `site_station_gap_*.csv`, `station_alignment_*.csv`.

### Ethical considerations
As Model 1.

### Caveats and recommendations
- The on-site outdoor record generally indicates higher temperatures than the station record. Results conditional on heatwave classification should therefore be reported with explicit reference to the outdoor record used to define each heatwave.
- In long heatwaves with several hot days, the lag of the room peak behind the single hottest outdoor hour is unstable; do not read it as a room property.
- The station comparison covers 2023–2025 only.

[Back to top](#-haven--model-card)

---

# 🔁 Model 3 — Recovery indicators and their reliability

### Model details
- **Notebook:** `05_resilience_metrics_and_evaluation.ipynb` (Sections 6–8b).
- **Type:** rule-based endpoints and a fitted exponential relaxation per room-episode.
- **Indicators:**
  - **Relief:** Number of nights following the episode until the first complete night without a temperature exceedance.
  - **Return to baseline:** Number of nights until the nightly mean temperature returns to within 1 K of the seasonal baseline, defined as the median over a ±15-day window excluding episode and heatwave nights. Alternative definitions use the pre-event baseline and a 0.5 K tolerance. Follow-up is limited to 21 nights (`ADR-026`).
  - **Post-episode excess:** Cumulative positive temperature anomaly, calculated as the sum of the positive differences between nightly mean temperature and the baseline over the first three complete post-episode nights (K·nights).
  - **Anomaly half-life:** Estimated time (in nights) required for the nightly mean temperature anomaly above baseline to decrease by 50%, measured from the final night of the episode.
  - **Decay half-life:** T(t) = T∞ + (T_peak − T∞)·e^(−λt) fitted to the 72 h after the room's peak, t½ = ln 2 / λ; fits with R² ≤ 0.3 excluded (`ADR-027`).
- **Reliability assessment:** Split-half reliability is evaluated using Spearman's rank correlation between zone-level medians calculated from randomly selected halves of the 12 episodes (2,000 random splits), alongside an odd–even split analysis. For the three bedrooms, concordance in room rankings across episodes is assessed using Kendall's coefficient of concordance (W), with statistical significance evaluated by permutation testing (`ADR-028`).

### Intended use
- **Uses:** describing how long night-time heat persists after episodes; deciding which recovery indicators can compare rooms.
- **Out of scope:** building time constants; separating building response from weather or occupant action.

### Factors
Zone, episode, baseline definition, tolerance, fitting window and R² filter.

### Metrics
Median and maximum endpoints per zone; half-life medians with bootstrap intervals, fit R² and the share of fits at the parameter bound; split-half reliability with 95% range; bedroom Kendall's W with permutation p-value; correlation and within-episode agreement with exposure.

### Data
Dataset A, measured values, the 12 indoor heat episodes (84 room-episodes).

### Quantitative analyses
`recovery_by_room.csv`, `recovery_baseline_comparison.csv`, `thermal_decay_fits.csv`, `thermal_decay_halflife_by_zone.csv`, `halflife_sensitivity.csv`, `halflife_split_half_reliability.csv`, `zone_episode_indicators.csv`, `exposure_recovery_correlation.csv`, `exposure_recovery_rank_agreement.csv`, `exposure_recovery_episode_level.csv`, `exposure_recovery_per_episode.csv`.

### Ethical considerations
As Model 1.

### Caveats and recommendations
- Check `halflife_split_half_reliability.csv` before comparing rooms on any recovery indicator; an indicator whose ranking does not reproduce between halves of the events cannot rank rooms.
- Associations between exposure and recovery indicators should be interpreted in light of the reliability of the latter. A weak correlation involving an unreliable recovery indicator cannot, by itself, be interpreted as evidence that exposure and recovery represent distinct underlying processes.
- The relatively small number of episodes (n = 12) limits the precision of the estimates and contributes to uncertainty in the corresponding confidence intervals.

[Back to top](#-haven--model-card)

---

# 📈 Model 4 — Ridge multi-horizon forecaster and baselines

### Model details
- **Notebooks:** `03_feature_engineering_and_psychrometrics.ipynb` (features) and `04_model_training_and_hyperparameter_tuning.ipynb`.
- **Model type:** Multi-output Ridge regression (`sklearn.linear_model.Ridge`), with a separate model fitted for each forecast horizon (1, 6, 12, 24 and 48 h). Each model predicts air temperature across all seven indoor zones simultaneously. Features are scaled using Min–Max scaling, with scaling parameters estimated exclusively from the training period. The regularisation parameter (α) is selected separately for each horizon from the range 0.01–100, using validation-set mean absolute error (MAE) (`ADR-017`).
- **Predictors:** A total of 161 predictors, comprising current system-state variables; lagged observations over 1–24 h; exponential moving averages (6, 24 and 72 h for outdoor variables and 24 and 72 h for indoor variables); 24 h diurnal temperature ranges; cyclical temporal encodings; a weekend indicator; and psychrometric variables, including Humidex, vapour-pressure deficit and dew-point depression, for each monitored location (`ADR-015`).
- **Baselines (same origins):** persistence; climatology plus damped anomaly (`ADR-018`).
- **Variants (sensitivity):** training without imputed origins; each bedroom without its own sensor; perfect outdoor temperature at the target time (upper bound); second chronological fold; ablation and feature ladder (`ADR-021`, `ADR-022`).
- **Artefacts:** `models/ridge.joblib` (scaler, models by horizon, penalties) and `models/model_summary.json`.
- **References:** Hoerl & Kennard (1970); Künsch (1989).

### Intended use
- **Uses:** supplying bedroom forecasts for hot-night warnings; testing what information makes room-level forecasts skilful.
- **Out of scope:** forecasts for other dwellings without refitting; control of building systems; forecasts in conditions far outside the training record without checking the hot-night bias.

### Factors
Zone, horizon, condition (on-site heatwave hours, other hours, bedroom night hours above 26 °C), test summer and feature group.

### Metrics
Mean absolute error (MAE), root mean squared error (RMSE) and coefficient of determination (R²), with uncertainty intervals estimated using a seven-day moving-block bootstrap. Forecast skill is assessed against the persistence baseline, while paired RMSE differences are calculated relative to both baselines. Prediction bias is examined during heatwave periods and hot-night hours. Changes in RMSE are also evaluated across the sensitivity analyses, ablation experiments and feature-ladder configurations.

### Data
Dataset A (complete record for inputs). Training to 30 April 2025, validation 1 May–30 November 2025, test from 1 December 2025; second fold as in `ADR-019`.

### Quantitative analyses
`hyperparameter_search.csv`, `forecast_accuracy_by_horizon.csv`, `forecast_accuracy_by_horizon_all_models.csv`, `forecast_paired_differences.csv`, `forecast_accuracy_by_zone_and_horizon.csv`, `forecast_accuracy_heatwave_vs_other.csv`, `forecast_imputation_sensitivity.csv`, `sensor_loss_accuracy.csv`, `perfect_weather_accuracy.csv`, `forecast_accuracy_fold2.csv`, `ablation_experiments.csv`, `feature_ladder.csv`, `feature_correlation_matrix.csv`.

### Ethical considerations
As Model 1; the model has no demographic inputs.

### Caveats and recommendations
- Report Ridge against the stronger baseline at each horizon; at 6–12 h persistence is out of phase with the daily cycle.
- Ridge exhibits under-prediction during the hottest bedroom nights. The warning layer partially compensates for this systematic bias through the application of forecast margins or probability-based thresholds.
- At forecast horizons of 24–48 h, residual prediction error is partly attributable to uncertainty in future outdoor weather conditions. The perfect-weather sensitivity analysis provides an estimate of the potential improvement achievable if future outdoor temperature were known accurately.
- Feature attribution describes the distribution of model contributions across potentially correlated input variables; it should not be interpreted as a direct measure of the independent information or causal influence associated with each predictor (`ADR-039`).
- Kitchen forecasts and indicators for the 2026 heatwaves may reflect a portable air conditioner used there during the daytime, whose operation was not logged.

[Back to top](#-haven--model-card)

---

# 🚨 Model 5 — Hot-night warning layer

### Model details
- **Notebook:** `06_night_time_warnings.ipynb`.
- **Model type:** A post-processing framework that converts the forecasts generated by Model 4, together with those from the baseline and sensitivity models, into bedroom-level hot-night warnings.
- **Rules:**
  - **Event:** a complete measured bedroom night with at least one hour above 26 °C (`ADR-032`).
  - **Margin warning:** forecast night maximum above 26 °C minus a margin of 0–2 K.
  - **Probability-based warning:** The probability that the nightly maximum temperature will exceed 26 °C is estimated using forecast errors from the 60 historical bedroom nights with the closest forecast-night maximum. Only observations available before the warning issue time are included. A corresponding 90% prediction interval is derived from the 5th and 95th percentiles of the same error distribution (`ADR-034`).
  - **Decision thresholds:** The forecast margin and probability threshold are optimised on the validation period by maximising the Critical Success Index (CSI). The selected thresholds are then held fixed throughout the corresponding test period (`ADR-033`).
  - **Warning lead time:** The warning is issued 16 h before the start of the target night for the 24 h forecast horizon and 40 h before the target night for the 48 h horizon, with the target night beginning at 22:00.

### Intended use
- **Uses:** retrospective evaluation of whether room-level forecasts support useful hot-night warnings, with stated uncertainty.
- **Out of scope:** operational warnings to occupants without prospective testing; autonomous control; health warnings.

### Factors
Model (Ridge, baselines, variants), horizon, warning form and setting, bedroom, test summer, heatwave status of the night, first night of an episode, and event definition.

### Metrics
Warning performance is assessed using hits, misses and false alarms, together with probability of detection (POD), false alarm ratio (FAR) and Critical Success Index (CSI). Differences in CSI between competing warning methods are evaluated using paired weekly block-bootstrap procedures. Probabilistic performance is assessed using the Brier score and Brier skill score, reliability diagrams with Wilson confidence intervals, and calibration slope and intercept. Additional analyses examine prediction interval coverage across different conditions, first-night warning performance with Clopper–Pearson confidence intervals, performance at equal numbers of false alarms, false alarms at equal detection, warning lead time, relative economic value across cost–loss ratios and effective sample sizes (`ADR-035`–`ADR-037`).

### Data
Validation and test forecasts from Notebook 04; observed events from Dataset A (measured values). Second test summer from the second fold.

### Quantitative analyses
`warning_settings_validation.csv`, `warning_skill.csv`, `warning_skill_both_summers.csv`, `probability_warnings_both_summers.csv`, `warning_skill_all_models.csv`, `warning_skill_differences.csv`, `warning_skill_by_heatwave_status.csv`, `probability_calibration.csv`, `probability_calibration_slope.csv`, `probability_reliability.csv`, `probability_method_sensitivity.csv`, `interval_coverage_by_condition.csv`, `first_night_warnings.csv`, `first_night_warnings_by_room.csv`, `first_nights_at_matched_false_alarms.csv`, `false_alarms_at_matched_detection.csv`, `warning_lead_time.csv`, `warning_relative_value.csv`, `sensor_loss_warning_skill.csv`, `perfect_weather_warning_comparison.csv`, `warning_skill_fold2.csv`, `warning_skill_differences_fold2.csv`, `first_night_warnings_fold2.csv`, `first_nights_pooled_folds.csv`, `fixed_margin_both_folds.csv`, `warning_skill_night_mean_event.csv`, `warning_skill_differences_night_mean_event.csv`, `warning_worked_example.csv`, `effective_sample_sizes.csv`, `shap_source_type_share_by_zone.csv`.

### Ethical considerations
Incorrect reliance on a warning system could leave occupants vulnerable to unanticipated indoor heat exposure. Accordingly, results are reported alongside their associated uncertainty, prediction interval coverage is assessed separately during heatwave conditions, and the absence of a warning is not interpreted as evidence that indoor conditions are safe.

### Caveats and recommendations
- The 1.0 K margin applied in the second fold was selected using the May–November 2025 validation period. Consequently, its result for the 2025 test summer of the second fold is an in-sample check rather than an independent evaluation.
- The chosen setting depends on the validation summer; check `warning_skill_both_summers.csv`, `fixed_margin_both_folds.csv` and the second-fold tables before relying on a setting.
- First-night warning results are based on a relatively small number of episodes. They should therefore always be reported alongside the number of episodes and the corresponding number of bedroom-level first-night observations.
- Interval coverage falls on heatwave nights, with misses on the hot side.
- The worked example is an illustration; no warning was issued to the household (`ADR-040`).

[Back to top](#-haven--model-card)

---

# 🧪 Model 6 — Comparator models used as sensitivity analyses

### Model details
- **Notebooks:** `04_model_training_and_hyperparameter_tuning.ipynb` (Section 12c) and `06_night_time_warnings.ipynb` (Section 16).
- **Linear quantile regression:** `sklearn.linear_model.QuantileRegressor` is fitted at the 0.5 and 0.9 quantiles for each bedroom and forecast horizon (24 and 48 h). Models are trained using forecast origins for which the target observation falls within the night-time period. The predictor set is identical to that used in Model 4. The L1 regularisation parameter is selected using validation-set pinball loss. Resulting forecasts are passed to the warning layer for evaluation alongside those from the other forecasting models (`ADR-023`).
- **Logistic regression:** `sklearn.linear_model.LogisticRegression` is fitted separately for each bedroom and forecast lead time to predict the occurrence of a hot night. Predictions use the scaled input features available at the issue time of the latest forecast incorporated into the warning framework. The L2 regularisation parameter is selected using validation-set Brier score, and the classification threshold is selected by maximising CSI on the validation period (`ADR-038`).

### Intended use
- **Uses:** assessing whether under-prediction of hot-night temperatures is associated with the use of a squared-error loss function, and whether direct event prediction provides better warning performance than the two-stage approach of forecasting temperature and subsequently applying a precautionary warning threshold.
- **Out of scope:** replacing Model 4 as the forecasting model.

### Factors
Bedroom, horizon, quantile level and event.

### Metrics
Pinball loss, share of hours below the forecast quantile, bias on all night hours and on hours above 26 °C (quantile); CSI, Brier score, first nights warned and the paired CSI difference from the Ridge probability warning (logistic).

### Data
As Model 4 (training, validation and test periods of the primary split).

### Quantitative analyses
`quantile_regression_accuracy.csv`, `logistic_hot_night_warnings.csv`, and the quantile rows of the warning tables of Model 5.

### Ethical considerations
As Model 1.

### Caveats and recommendations
- Each bedroom has few hot training nights, so the logistic model is fitted on a small number of events.
- The quantile models are fitted on night-time origins only and are not general-purpose forecasters.

[Back to top](#-haven--model-card)
