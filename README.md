<p align="left">
• <a href="https://github.com/hellomayzune"><strong>GitHub</strong></a> •
<a href="https://orcid.org/0000-0003-0282-2633"><strong>ORCID</strong></a> •
<a href="https://scholar.google.com/citations?user=LmP8B_4AAAAJ&hl=en"><strong>Google Scholar</strong></a> •
<a href="https://www.researchgate.net/profile/May-Zune"><strong>ResearchGate</strong></a> •
<a href="https://www.linkedin.com/in/mayzune/"><strong>LinkedIn</strong></a> •
<a href="https://mayzune.com/"><strong>Blog</strong></a> •

</p>

# 🌡️ HAVEN: Human-centred Assessment of Vulnerability to Environmental heat at Night

> ℹ️ **Plain English Version**: HAVEN is a workflow that turns room-by-room temperature monitoring into three things: how much night-time heat each bedroom accumulates, how long that heat lingers, and whether hot nights can be warned of a day ahead. Applied to one monitored home, HAVEN showed that cumulative night-time exposure ranks rooms consistently, fitted cooling rates do not, and day-ahead warnings beat simple baselines overall but not reliably on the first hot night. 

This repository contains the Jupyter notebooks, shared Python source code, input data documentation and processed outputs for **HAVEN**, an empirical framework for assessing night-time residential heat. HAVEN applies consistent definitions to measured records: indoor heat episodes are identified directly from bedroom temperature measurements, independently of outdoor heatwave dates; night-time exposure and recovery indicators are computed from empirical measurements; and candidate hot-night warnings are generated from room-level forecasts and simple baselines over identical periods.

The analysis uses hourly air temperature, relative humidity and dew-point temperature measurements from seven indoor zones and one outdoor logger installed in a free-running detached dwelling in the East Midlands, UK. The monitoring period runs from 1 May 2023 to 1 October 2026. The analysis is supplemented by hourly observations from the nearest synoptic station, obtained through the Met Office MIDAS Open dataset. The term "human-centred" refers to the spatial and temporal focus of the analysis: bedrooms during sleeping hours. No occupant-level, behavioural, sleep or health measurements were collected.

- **Project**: HAVEN — Human-centred Assessment of Vulnerability to Environmental heat at Night
- **Authors**: [May Zune](https://orcid.org/0000-0003-0282-2633), [Renata Tubelo](https://orcid.org/0000-0001-9607-7616), [Julie Waldron](https://orcid.org/0000-0002-7572-1326)
- **Notebook author**: [May Zune](https://orcid.org/0000-0003-0282-2633)
- **Funding**: No specific grant was received from any funding agency in the public, commercial or not-for-profit sectors.
- **Associated publication**: *Residential night-time heat exposure, thermal recovery and short-term warning: Evidence from a monitored dwelling* — Manuscript submitted for publication. Full bibliographic details and the associated Zenodo DOI will be added when available.

📬 **Contact**: Please open an issue or use the links above.

🆚 **Version**: v1

©️ **Licence**: Code licensed under the *MIT Licence*; authors' processed outputs (tables, figures, and model files) under *CC BY 4.0*. The raw monitoring record, derived hourly data, and third-party weather data are not redistributed (see [Excluded from Repository](#excluded-from-repository-non-distributable)).

📖 Contents

- [Research Questions](#-research-questions)
- [Repository Information](#️-repository-information)
- [Pipeline Notebooks](#️-pipeline-notebooks)
- [Software Requirements & Setup](#-software-requirements--setup)
- [Reproducibility & Execution Steps](#-reproducibility--execution-steps)
- [Project Documentation](#-project-documentation)
- [Manuscript and Supplementary Display Items](#-manuscript-and-supplementary-display-items)
- [Data Availability & Repository Scope](#-data-availability--repository-scope)

---

# ❓ Research Questions

| | Question | Notebooks |
|---|---|---|
| **RQ1** | To be updated | 05 (with 01–02) |
| **RQ2** | To be updated | 05 |
| **RQ3** | To be updated | 02, 05 |
| **RQ4** | To be updated | 03, 04, 06 |

[Back to top](#️-haven-human-centred-assessment-of-vulnerability-to-environmental-heat-at-night)

---

# 🗂️ Repository Information

- **Contents**: Jupyter notebooks (`.ipynb`) with HTML previews, Python modules (`.py`), and generated artefacts (tables, figures, and model files: `.csv`, `.png`, `.pdf`, `.joblib`, `.json`).
- **Programming Language**: `Python` (`pandas`, `numpy`, `scipy`, `scikit-learn`, `statsmodels`, `matplotlib`, `seaborn`, `pyarrow`, `openpyxl`, `joblib`).
- **Methodological Scope**: No numerical building-physics simulations are used; all reported results derive directly from empirical measurements and statistical models fitted to them.
- **Repository Metadata**: Created 2 November 2026. 

📌 **Repository Map**

```text
├── README.md                                    # Project overview (this file)
├── DATASHEET.md                                 # Dataset documentation (Gebru et al., 2021)
├── ADR.md                                       # Architecture decision records
├── MODEL_CARD.md                                # Model cards (Mitchell et al., 2019)
├── requirements.txt                             # Package versions of the reported run
├── src/                                         # Shared Python modules imported across notebooks
│   ├── config.py                                # Paths, dates, thresholds, and event rules (single source of truth)
│   ├── preprocessing.py                         # Workbook/MIDAS loaders, quality control, and imputation helpers
│   ├── features.py                              # Psychrometrics, forecast-origin features, heatwave detection, and climatological baselines
│   ├── metrics.py                               # Night-time exposure, TM59 Criterion (b), decay fits, warning skill, and probabilities
│   ├── events.py                                # Nightly panels, indoor episodes, event envelopes, and recovery indicators
│   ├── evaluation.py                            # Error metrics and moving-block bootstrap confidence intervals
│   ├── xai.py                                   # Linear-model feature contributions grouped by source feature category
│   └── plotting.py                              # Shared figure styling conventions and zone color palettes
├── notebooks/
│   ├── 01_data_cleaning_and_imputation.ipynb
│   ├── 02_exploratory_data_analysis.ipynb
│   ├── 03_feature_engineering_and_psychrometrics.ipynb
│   ├── 04_model_training_and_hyperparameter_tuning.ipynb
│   ├── 05_resilience_metrics_and_evaluation.ipynb
│   ├── 06_night_time_warnings.ipynb
│   └── 0*-preview.html                          # Static HTML exports of executed notebooks (generated at final cell)
├── data/
│   ├── raw/
│   │   ├── NottinghamMeasuredTRH.xlsx           # Hourly monitoring record, 8 sheets (non-distributable)
│   │   └── midas/midas-open_uk-hourly-weather-obs_*.csv # Met Office MIDAS Open, Sutton Bonington, 2023–2025 (third-party; non-distributable)
│   └── processed/                               # Hourly intermediate files generated via pipeline execution (non-distributable)
├── models/                                      # Fitted Ridge models and execution summaries (ridge.joblib, model_summary.json)
└── outputs/
    ├── figures/
    │   ├── manuscript/                          # Main text figures (Figures 2–6; .png, .pdf; Figure 2 combines dual files)
    │   └── supplementary/                       # Supplementary figures (Figures S1–S19 and workflow schematic)
    └── tables/
        ├── manuscript/                          # Main text tabular data (Tables 1–7 and Appendix Table A1; .csv)
        └── supplementary/                       # Supplementary tables (Tables S1–S60 and supporting data files)
```

[Back to top](#️-haven-human-centred-assessment-of-vulnerability-to-environmental-heat-at-night)

---

# ⚙️ Pipeline Notebooks

The six notebooks execute sequentially, sharing modular code from the `src/` directory. [Notebook 01](#-notebook-01--data-cleaning-quality-control-and-imputation) cleans the monitoring record while tracking data provenance. [Notebook 02](#-notebook-02--exploratory-analysis-thermal-microclimates-and-outdoor-heat-hazard-rq3) characterises the dwelling and quantifies outdoor heat hazards across two records. [Notebook 03](#-notebook-03--feature-engineering-and-psychrometrics-rq4-enabling) constructs forecasting features using information available at each forecast origin. [Notebook 04](#-notebook-04--forecasting-the-multi-zone-indoor-state-rq4-enabling) fits multi-zone indoor temperature models and benchmarks. [Notebook 05](#-notebook-05--exposure-indoor-heat-episodes-and-recovery-rq1rq3) computes exposure metrics, indoor heat episodes and thermal recovery from empirical observations. [Notebook 06](#-notebook-06--night-time-warnings-rq4) converts temperature forecasts into candidate hot-night warnings and evaluates their predictive performance.

### 📊 Notebook 01 — Data cleaning, quality control and imputation
*See `DATASHEET.md` Dataset A, `ADR-009`–`ADR-011`.*

| Purpose | Key inputs | Produces |
|---|---|---|
| Load the eight-sheet workbook onto a regular hourly grid; apply physical-bound and rate-of-change screening, with a centred robust z-score used as a comparative screening method; characterise missing data; execute an imputation benchmark on masked gaps; apply PCHIP interpolation to short interior gaps and donor-zone regression to longer and boundary gaps; generate a provenance mask and runtime environment record. | `data/raw/NottinghamMeasuredTRH.xlsx` | `data/processed/01_cleansed_imputed_data.parquet`, `01_qc_flags.parquet`; `imputation_benchmark.csv`, `imputation_log.csv`; raw-versus-cleansed diagnostic figures; framework schematic; `outputs/environment_versions.txt` |

### 📊 Notebook 02 — Exploratory analysis, thermal microclimates and outdoor heat hazard (RQ3)
*See `DATASHEET.md` Datasets A–B, `ADR-012`–`ADR-014`.*

| Purpose | Key inputs | Produces |
|---|---|---|
| Generate zone summary statistics and time-series overviews; evaluate thermal lag, damping, and diurnal profiles; analyse seasonal humidity distributions and serial autocorrelation; construct on-site and station heatwave catalogues using consistent Met Office criteria; assess temporal alignment between the station and on-site records, and examine site–station temperature differences under different wind and solar-radiation conditions. | Notebook 01 outputs, MIDAS files in `data/raw/midas/` | `zone_summary_statistics.csv`, heatwave catalogues and matching tables, site–station comparison tables, temporal alignment and sensitivity tables, exploratory figures; `02_station_hourly.parquet`, `02_heatwaves_onsite.parquet`, `02_heatwaves_station.parquet` |

### 📊 Notebook 03 — Feature engineering and psychrometrics (RQ4, enabling)
*See `MODEL_CARD.md` Model 4, `ADR-015`–`ADR-016`.*

| Purpose | Key inputs | Produces |
|---|---|---|
| Construct forecast-origin feature blocks using information available at or before each prediction time. These include system state, temporal lags, exponential moving averages, diurnal ranges, cyclic time encodings, Humidex, vapour-pressure deficit and dew-point depression for all locations; execute feature-timing and psychrometric validation checks; generate psychrometric visualisations for the peak on-site heatwave; quantify moisture-buffering patterns and compute feature-correlation matrices. | `01_cleansed_imputed_data.parquet` | `data/processed/02_engineered_features.parquet`, `feature_correlation_matrix.csv`, analytical figures |

### 📊 Notebook 04 — Forecasting the multi-zone indoor state (RQ4, enabling)
*See `MODEL_CARD.md` Model 4, `ADR-017`–`ADR-023`.*

| Purpose | Key inputs | Produces |
|---|---|---|
| Implement purged chronological training, validation and test splits; fit Ridge regression models across multiple horizons (1, 6, 12, 24 and 48 h) with penalties optimised on validation sets; evaluate persistence and climatology-plus-anomaly baselines across identical forecast origins; quantify accuracy using block-bootstrap confidence intervals and paired difference tests; conduct residual and heatwave diagnostics; assess sensitivity to imputed observations, the removal of room-level sensor inputs, and the availability of future outdoor temperature measurements as an idealised weather-information upper bound; evaluate a secondary chronological fold, linear quantile regression, model ablation and feature ladders. | Notebook 03 output, `01_qc_flags.parquet`, `02_heatwaves_onsite.parquet` | `03_train_val_test_splits.parquet`, `04_*_predictions.parquet`; `models/ridge.joblib`, `models/model_summary.json`; `forecast_accuracy_*.csv`, `forecast_paired_differences.csv`, `feature_ladder.csv`, `quantile_regression_accuracy.csv` and associated tables; diagnostic figures |

### 📊 Notebook 05 — Exposure, indoor heat episodes and recovery (RQ1–RQ3)
*See `MODEL_CARD.md` Models 1–3, `ADR-024`–`ADR-031`.*

| Purpose | Key inputs | Produces |
|---|---|---|
| Quantify night-time exposure ($I_{\text{night}}$, exceedance frequency and no-relief nights) by room and year while accounting for inter-annual outdoor heat variation; identify indoor heat episodes and align them with outdoor heatwave catalogues; calculate indicative results using the numerical thresholds associated with CIBSE TM59 Criterion (b), applied to measured bedroom air temperatures; map event envelopes and timing profiles; derive thermal recovery endpoints, post-episode excess heat and decay half-lives; evaluate split-half reliability of room indicators and rank concordance across bedrooms (Kendall's $W$); assess bedroom exposure using nights with complete measurements across all three bedrooms; evaluate indicator agreement and test sensitivity to thresholds, episode definitions and operative-temperature offsets. | Notebook 01–02 outputs, raw workbook (TM59 cross-check) | `05_indoor_episodes.parquet`; `tm59_criterion_b_bedrooms.csv`, `indoor_episode_catalogue.csv`, `exposure_and_recovery_by_zone.csv`, `recovery_by_room.csv`, `halflife_split_half_reliability.csv`, `exposure_year_effects.csv` and associated tables; analytical figures |

### 📊 Notebook 06 — Night-time warnings (RQ4)
*See `MODEL_CARD.md` Models 5–6, `ADR-032`–`ADR-040`.*

| Purpose | Key inputs | Produces |
|---|---|---|
| Generate night-level forecasts and identify observed hot nights; estimate probabilities using an updating pool of historical forecast errors; margins and probability thresholds on validation sets; evaluate warning skill via paired block-bootstrap differences; assess probability calibration and prediction-interval coverage; test first-night warning performance under matched false-alarm rates; evaluate warning lead times and relative economic value under specified cost–loss assumptions; assess performance under sensor loss and idealised future-weather information; calculate Ridge feature contributions by source (supplementary); evaluate performance across a second summer test period, alternative night-mean event definitions, fixed decision margins and logistic-regression comparators. | Notebook 04–05 outputs, models in `models/` | `warning_skill.csv`, `warning_skill_both_summers.csv`, `effective_sample_sizes.csv`, `probability_calibration.csv`, `interval_coverage_by_condition.csv`, `first_night_warnings.csv`, `warning_skill_fold2.csv`, `fixed_margin_both_folds.csv`, `probability_warnings_both_summers.csv` and associated tables; evaluation figures |

[Back to top](#️-haven-human-centred-assessment-of-vulnerability-to-environmental-heat-at-night)

---

# 💻 Software Requirements & Setup

### **Environment Specifications**
- **Python Version**: Python 3.10 or later (the reported run used Python `3.13.9`).
- **Interface**: Jupyter Notebook or JupyterLab.
- **Key Dependencies & Pinned Versions** (as recorded in `requirements.txt`):
  - `pandas` (v3.0.5)
  - `numpy` (v2.5.3)
  - `scipy` (v1.18.1)
  - `scikit-learn` (v1.9.1)
  - `statsmodels` (v0.15.0) *(Note: Notebook 02 requires `statsmodels` $\ge 0.15$ when paired with `pandas` 3)*
  - `matplotlib` (v3.11.2)
  - `seaborn` (v0.13.2)
  - `pyarrow` (v21.0.0)
  - `openpyxl` (v3.1.5)
  - `joblib` (v1.6.0)
- **Runtime Tracking**: Notebook 01 automatically records active environment versions to `outputs/environment_versions.txt`.

### **Installation & Setup**
To create the Python environment used for the reported analysis, execute the following commands in your terminal:

```bash
conda create -n haven-env python=3.13
conda activate haven-env
pip install -r requirements.txt
```

[Back to top](#️-haven-human-centred-assessment-of-vulnerability-to-environmental-heat-at-night)

---

# 🔁 Reproducibility & Execution Steps

### **Data Placement & Architecture Rules**
* Place the raw monitoring workbook in `data/raw/` and the Met Office MIDAS files in `data/raw/midas/`.
* **Central Configuration**: All paths, temporal boundaries, thresholds, and event rules are managed centrally in `src/config.py` (`ADR-003`). Parameters dependent on record length (such as heatwaves, episodes, and complete years) are dynamically derived, allowing notebooks to re-run automatically when monitoring data is updated.
* **Immutability**: Raw data files are never modified (`ADR-002`). Each notebook reads from `data/raw/` or `data/processed/`, and writes its intermediate and final outputs to `data/processed/`, `models/`, or `outputs/`.
* **Deterministic Execution**: Stochastic operations that use the configured random-state parameter are initialised with a fixed seed (`RANDOM_SEED = 42` in `src/config.py`). All reported values are read from saved tables rather than typed into the notebooks (`ADR-007`).

### **Sequential Execution Order**
Execute the notebooks in sequence from a fresh kernel:

1. **`01_data_cleaning_and_imputation.ipynb`**
2. **`02_exploratory_data_analysis.ipynb`** *(Generates heatwave catalogues required by Notebooks 04–06)*
3. **`03_feature_engineering_and_psychrometrics.ipynb`**
4. **`04_model_training_and_hyperparameter_tuning.ipynb`** *(Generates forecasts and models required by Notebook 06)*
5. **`05_resilience_metrics_and_evaluation.ipynb`** *(Generates indoor episodes required by Notebook 06)*
6. **`06_night_time_warnings.ipynb`**

### **Re-running from Public Files**
Because the raw monitoring records and derived intermediate files are non-distributable (`ADR-008`), the complete set of results from the reported run are pre-packaged in `outputs/` and `models/`. The notebooks are designed to execute on a locally supplied workbook that matches the schema documented in `DATASHEET.md` (Dataset A). Successful execution may also depend on the availability and compatibility of the required MIDAS files and software environment.

*Tip: Before a clean replication, clear the manuscript and supplementary output folders so that files from earlier runs are not mistaken for new outputs.*

[Back to top](#️-haven-human-centred-assessment-of-vulnerability-to-environmental-heat-at-night)

---

# 📖 Project Documentation

Additional repository documentation follows established open-science and AI reproducibility frameworks:

| Document | Framework / Standard | Scope & Coverage |
| :--- | :--- | :--- |
| [`DATASHEET.md`](./DATASHEET.md) | Gebru et al. (2021) | Motivation, composition, collection, preprocessing, intended uses, distribution limitations and maintenance of the monitoring records, MIDAS observations and derived data products. |
| [`ADR.md`](./ADR.md) | Lightweight MADR | Documented architecture and design decisions across all notebooks, detailing alternatives considered, rationale, and technical consequences. |
| [`MODEL_CARD.md`](./MODEL_CARD.md) | Mitchell et al. (2019) | Intended use, operating factors, evaluation metrics, training data and limitations across six analytical components: (1) night-time exposure and indicative TM59 assessments, (2) indoor heat episodes and event envelopes, (3) thermal recovery indicators, (4) the Ridge forecasting framework and baselines, (5) the retrospective hot-night warning framework, and (6) comparator models based on alternative forecasting and classification approaches. |

[Back to top](#️-haven-human-centred-assessment-of-vulnerability-to-environmental-heat-at-night)

---

# 🧾 Manuscript and Supplementary Display Items

**Article**

| Item | File | Notebook |
|---|---|---|
| Figure 1 | Prepared outside the notebooks (workflow and monitoring configuration) | — |
| Figure 2 | `figures/manuscript/timeseries_overview` (a–c) and `figures/manuscript/tm59_failures_by_heatwave_status` (d–e) | 02, 05 |
| Figure 3 | `figures/manuscript/night_temperature_heatmap` | 05 |
| Figure 4 | `figures/manuscript/event_envelopes_2026` | 05 |
| Figure 5 | `figures/manuscript/forecast_rmse_by_horizon` | 04 |
| Figure 6 | `figures/manuscript/warning_tradeoff` | 06 |
| Table 1 | `tables/manuscript/zone_summary_statistics.csv` | 02 |
| Table 2 | `tables/manuscript/exposure_and_recovery_by_zone.csv` | 05 |
| Table 3 | `tables/manuscript/tm59_criterion_b_bedrooms.csv` | 05 |
| Table 4 | `tables/manuscript/indoor_episode_catalogue.csv` | 05 |
| Table 5 | `tables/manuscript/halflife_split_half_reliability.csv` | 05 |
| Table 6 | `tables/manuscript/warning_skill.csv` | 06 |
| Table 7 | `tables/manuscript/warning_skill_both_summers.csv` | 06 |
| Table A1 | `tables/manuscript/effective_sample_sizes.csv` | 06 |

**Supplementary tables** (`tables/supplementary/`)

| Item | File | Notebook |
|---|---|---|
| S1 | `imputation_benchmark.csv` | 01 |
| S2 | `heatwave_catalogue_onsite.csv` | 02 |
| S3 | `heatwave_catalogue_station.csv` | 02 |
| S4 | `heatwave_episode_matching.csv` | 05 |
| S5 | `site_station_gap_summary.csv`, `site_station_gap_night_by_wind_and_sunshine.csv`, `site_station_gap_midday_by_sunshine.csv` | 02 |
| S6 | `station_alignment_check.csv` | 02 |
| S7 | `station_alignment_sensitivity.csv` | 02 |
| S8 | `night_exposure_by_bedroom_and_year.csv` | 05 |
| S9 | `exposure_and_outdoor_heat_by_year.csv`, `exposure_year_effects.csv` | 05 |
| S10 | `tm59_failure_nights.csv` | 05 |
| S11 | `threshold_sensitivity.csv` | 05 |
| S12 | `episode_definition_sensitivity.csv` | 05 |
| S13 | `operative_offset_sensitivity.csv` | 05 |
| S14 | `event_timing_by_room.csv` | 05 |
| S15 | `exposure_on_heatwave_nights.csv` | 05 |
| S16 | `exposure_by_heatwave_phase.csv` | 05 |
| S17 | `exposure_by_catalogue_agreement.csv` | 05 |
| S18 | `thermal_decay_halflife_by_zone.csv` | 05 |
| S19 | `halflife_sensitivity.csv` | 05 |
| S20 | `recovery_baseline_comparison.csv` | 05 |
| S21 | `recovery_by_room.csv` | 05 |
| S22 | `exposure_recovery_correlation.csv` | 05 |
| S23 | `exposure_recovery_episode_level.csv` | 05 |
| S24 | `indicator_agreement.csv` | 05 |
| S25 | `hyperparameter_search.csv` | 04 |
| S26 | `forecast_accuracy_by_horizon_all_models.csv` | 04 |
| S27 | `forecast_accuracy_by_zone_and_horizon.csv` | 04 |
| S28 | `forecast_accuracy_heatwave_vs_other.csv` | 04 |
| S29 | `forecast_imputation_sensitivity.csv` | 04 |
| S30 | `forecast_accuracy_fold2.csv` | 04 |
| S31 | `feature_ladder.csv` | 04 |
| S32 | `ablation_experiments.csv` | 04 |
| S33 | `perfect_weather_accuracy.csv` | 04 |
| S34 | `sensor_loss_accuracy.csv` | 04 |
| S35 | `quantile_regression_accuracy.csv` | 04 |
| S36 | `warning_settings_validation.csv` | 06 |
| S37 | `warning_skill_all_models.csv` (all bedrooms) | 06 |
| S38 | `warning_skill_differences.csv` | 06 |
| S39 | `warning_skill_by_heatwave_status.csv` | 06 |
| S40 | `probability_calibration.csv` | 06 |
| S41 | `probability_calibration_slope.csv` | 06 |
| S42 | `interval_coverage_by_condition.csv` | 06 |
| S43 | `first_nights_at_matched_false_alarms.csv` | 06 |
| S44 | `false_alarms_at_matched_detection.csv` | 06 |
| S45 | `warning_skill_fold2.csv` | 06 |
| S46 | `warning_skill_differences_fold2.csv` | 06 |
| S47 | `first_night_warnings_fold2.csv` | 06 |
| S48 | `fixed_margin_both_folds.csv` | 06 |
| S49 | `warning_skill_night_mean_event.csv`, `warning_skill_differences_night_mean_event.csv` | 06 |
| S50 | `logistic_hot_night_warnings.csv` | 06 |
| S51 | `sensor_loss_warning_skill.csv` | 06 |
| S52 | `perfect_weather_warning_comparison.csv` | 06 |
| S53 | `warning_lead_time.csv` | 06 |
| S54 | `warning_relative_value.csv` | 06 |
| S55 | `shap_source_type_share_by_zone.csv` | 06 |
| S56 | `tm59_failures_outside_heatwaves.csv` | 05 |
| S57 | `ridge_vs_tree_ensembles.csv` | 04 |
| S58 | `probability_warnings_both_summers.csv` | 06 |
| S59 | `forecast_accuracy_by_horizon.csv` (with `forecast_paired_differences.csv`) | 04 |
| S60 | `bedroom_exposure_common_nights.csv` | 05 |

**Supplementary figures** (`figures/supplementary/`)

| Item | File | Notebook |
|---|---|---|
| S1 | `raw_vs_cleansed_temperature` | 01 |
| S2 | `raw_vs_cleansed_humidity` | 01 |
| S3 | `seasonal_RH_dewpoint_CDFs` | 02 |
| S4 | `ACF_PACF_indoor_temperature` | 02 |
| S5 | `moisture_buffering_VPD_vs_RH` | 03 |
| S6 | `daily_max_onsite_vs_station` | 02 |
| S7 | `site_station_gap_by_conditions` | 02 |
| S8 | `diurnal_profiles_and_thermal_lag` | 02 |
| S9 | `psychrometric_strain_peak_heatwave` | 03 |
| S10 | `halflife_example_fits` | 05 |
| S11 | `residual_density_qq` | 04 |
| S12 | `predicted_vs_measured` | 04 |
| S13 | `ablation_rmse_by_horizon` | 04 |
| S14 | `feature_ladder_by_horizon` | 04 |
| S15 | `warning_relative_value` | 06 |
| S16 | `warning_information_timeline` | 06 |
| S17 | `forecast_to_action_case_episode` | 06 |
| S18 | `shap_source_matrix` | 06 |
| S19 | `probability_reliability` | 06 |

**Supporting data files.** The supplementary folders also hold files that underlie the numbered items but are not reproduced in the Supplementary document: `imputation_log.csv`, `heatwave_matching_onsite_station.csv`, `site_station_gap_by_hour.csv`, `feature_correlation_matrix.csv`, `event_envelopes_by_room.csv`, `thermal_decay_fits.csv`, `zone_episode_indicators.csv`, `exposure_recovery_rank_agreement.csv`, `exposure_recovery_per_episode.csv`, `room_profile.csv`, `tm59_failure_nights_by_heatwave_status.csv`, `first_night_warnings.csv`, `first_night_warnings_by_room.csv`, `first_nights_pooled_folds.csv`, `probability_reliability.csv`, `probability_method_sensitivity.csv`, `warning_worked_example.csv`, and the working schematic `framework_schematic_and_zone_layout` (Notebook 01).

[Back to top](#️-haven-human-centred-assessment-of-vulnerability-to-environmental-heat-at-night)

---

# 📦 Data Availability & Repository Scope

### **Included in Repository**
* **Source Code & Documentation**: All Jupyter notebooks, static HTML previews, Python modules in `src/`, and dependency specifications in `requirements.txt` (licensed under the **MIT Licence**).
* **Generated Artifacts**: All figures, tables and serialised model files in `outputs/` and `models/` are distributed under **CC BY 4.0**. Supplementary files are also deposited under the associated Supplementary DOI.

### **Excluded from Repository (Non-Distributable)**
* **Raw Monitoring Record** (`NottinghamMeasuredTRH.xlsx`): Not distributed because of occupant privacy considerations and confidentiality agreements. The expected data schema and structure are documented in `DATASHEET.md` (Dataset A) to support local execution on compatible datasets.
* **Processed Intermediate Files** (`data/processed/*.parquet`): Not distributed because they contain processed versions of the non-distributable raw monitoring record and could expose information from which the original hourly measurements may be reconstructed (`ADR-008`).
* **Third-Party Weather Observations** (`midas-open_uk-hourly-weather-obs_*.csv`): Met Office MIDAS Open hourly observations from Sutton Bonington (Station ID: 00554; dataset version `dv-202607`) are not redistributed. They can be obtained directly from the Centre for Environmental Data Analysis ([CEDA](https://archive.ceda.ac.uk/)) under the applicable Open Government Licence, subject to the required attribution.
* **Identifying Metadata**: Any details revealing the physical address or occupant identifiers of the monitored dwelling are strictly redacted.

[Back to top](#️-haven-human-centred-assessment-of-vulnerability-to-environmental-heat-at-night)
