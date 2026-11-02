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
<a href="./MODEL_CARD.md"><strong>MODEL_CARD.md</strong></a>
</p>

# 🗺️ HAVEN — Architecture Decision Records

This document records the main design decisions behind the **HAVEN** (Human-centred Assessment of Vulnerability to Environmental heat at Night) notebooks, in a lightweight **MADR** (Markdown Architectural Decision Records) format. Each record gives the **Context**, the **Decision**, the **Alternatives considered** and the **Consequences**. Repository-wide decisions come first, followed by the decisions of each notebook in run order.

📖 Contents

- [Cross-cutting decisions](#-cross-cutting-decisions) (ADR-001–008)
- [Notebook 01 — Data cleaning, quality control and imputation](#-notebook-01--data-cleaning-quality-control-and-imputation) (ADR-009–011)
- [Notebook 02 — Outdoor heat hazard and the station record](#-notebook-02--outdoor-heat-hazard-and-the-station-record) (ADR-012–014)
- [Notebook 03 — Features](#-notebook-03--features) (ADR-015–016)
- [Notebook 04 — Forecasting](#-notebook-04--forecasting) (ADR-017–023)
- [Notebook 05 — Exposure, episodes and recovery](#-notebook-05--exposure-episodes-and-recovery) (ADR-024–031)
- [Notebook 06 — Night-time warnings](#-notebook-06--night-time-warnings) (ADR-032–040)

[Back to README](./README.md#️-haven-human-centred-assessment-of-vulnerability-to-environmental-heat-at-night)

---

# 🌐 Cross-cutting decisions

### ADR-001 — One Record and One Set of Definitions for Every Indicator

**Context.**

Exposure, event timing, recovery and warnings are often studied separately, with different periods, thresholds and night windows, so their results cannot be compared night by night.

**Decision.**

Every stage uses a consistent hourly record, the same night-time window (22:00–07:00, labelled by its start date), the same exposure threshold (26 °C) and the same completeness criterion for a measured night. Indoor heat episodes, exposure indicators, recovery indicators and warnings therefore refer to the same nights.

**Alternatives considered.**

Choosing the period and definitions separately for each analysis, for example a different threshold for warnings than for exposure, was rejected because results would no longer be comparable across research questions.

**Consequences.**

A result in one notebook can be traced to the same nights in another. Changing a definition in `src/config.py` changes it everywhere, which is intended; alternative definitions are run as explicit sensitivity analyses rather than by editing the defaults.

---

### ADR-002 — `data/raw/` → `data/processed/` → `outputs/` Directory Convention

**Context.**

Six notebooks share the raw workbook and pass intermediate data down the pipeline.

**Decision.**

Notebooks read raw data from `data/raw/` and never modify it. Intermediate hourly products are written to `data/processed/` using the number of the notebook that generates them (`01_…` to `05_…`), fitted models to `models/`, and tables and figures to `outputs/`, separated into `manuscript/` and `supplementary/`. The `manuscript/` folders hold only the display items of the article, which are submitted with it; the `supplementary/` folders hold the items of the Supplementary material and their supporting data files, which are deposited with the Supplementary DOI.

**Alternatives considered.**

Writing results back into the workbook, or keeping each notebook's files in its own folder, was rejected because both make the authoritative source of a value unclear.

**Consequences.**

Any notebook can be re-run from a fresh kernel once the files of the notebooks before it exist, and the hand-offs between notebooks are visible from file names alone. When an item moves between the article and the Supplementary material, only its destination folder changes in the notebook that writes it; the README maps every numbered item to its file.

---

### ADR-003 — Single Configuration Module and Values Derived From the Data

**Context.**

Monitoring continues and months are appended to the workbook; hard-coded dates and counts would make the pipeline fragile.

**Decision.**

Every path, date boundary, threshold and event rule is set once in `src/config.py`. Quantities that depend on the length of the record — heatwaves, indoor episodes, the peak event, complete years and the test period's end — are derived from the data at run time. Shared computations live in `src/` modules rather than in notebook cells.

**Alternatives considered.**

Setting constants in each notebook was rejected because the same value would be maintained in several places and could silently diverge.

**Consequences.**

The pipeline can be rerun without modification when the monitoring record is extended. The split dates (`TRAIN_END`, `VAL_END` and the second-fold dates) are the exception: they are fixed so that the reported test summers do not move when data are added.

---

### ADR-004 — Measured Values Only for Every Outcome Indicator

**Context.**

Imputation is needed to give the forecasting model an unbroken history, but reconstructed values would bias exposure, episode, recovery and warning outcomes.

**Decision.**

A provenance mask from Notebook 01 identifies values that have been corrected or imputed, retaining the distinction between the two operations. For exposure, episodes, recovery, TM59 and the observed events that warnings are scored against, these values are treated as missing, and a night is evaluated only when all nine hours from 22:00 to 07:00 were measured. Imputed values enter only the forecasting inputs.

**Alternatives considered.**

Computing outcomes on the imputed record was rejected because the indicators would partly measure the imputation method.

**Consequences.**

Rooms with later logger installation have fewer evaluable nights, and the June 2023 heatwave cannot enter the episode analysis. The effect of imputed values on forecast accuracy is tested separately (`ADR-021`).

---

### ADR-005 — Logger Clock as the Single Time Base

**Context.**

The logger's time convention is undocumented, while MIDAS observations are in UTC; misalignment would shift night windows and heatwave timing.

**Decision.**

All records are analysed on the logger clock. Cross-correlation between the logger and station records identifies a best alignment shift of +1 h across the overall record and the examined seasonal, annual and GMT/BST subsets. The analysis therefore adopts a constant UTC+1 logger-time convention and shifts station timestamps by +1 h (`STATION_TO_LOGGER_SHIFT_H`). Logger timestamps remain unchanged.

**Alternatives considered.**

Assuming that logger timestamps follow UK local civil time with daylight-saving changes was rejected because the same alignment shift was identified in the examined GMT and BST subsets. Shifting the logger instead of the station was rejected because all indoor indicators would then move.

**Consequences.**

Night windows and the TM59 sleeping period refer to local clock time in summer. Station-based results are recomputed with shifts of 0 and 2 h to show which of them depend on the alignment.

---

### ADR-006 — Screening Indicators Kept Separate From Compliance and Health Outcomes

**Context.**

Indicators such as I_night, no-relief nights and TM59 failure nights could be read as compliance results or health thresholds.

**Decision.**

The 26 °C threshold is described as a screening level and is varied from 24 to 27 °C. TM59 Criterion (b) is applied to measured air temperature and reported as indicative, not as compliance. Humidex and vapour-pressure deficit are reported as environmental descriptors. No indicator is described as a measure of sleep, physiological strain or health.

**Alternatives considered.**

Pass/fail language for the study indicators was rejected because it would blur screening and compliance.

**Consequences.**

Interpretation is limited to the environment of the rooms; any link to occupants requires data this study did not collect.

---

### ADR-007 — Reported Values Generated From Saved Tables

**Context.**

Values entered manually in the documentation can become outdated when notebooks are rerun.

**Decision.**

Notebook markdown cells describe methods and interpretation and do not quote results; results are printed by code cells and written to `outputs/tables/`. Reported values are given to two decimal places, and manuscript tables are rounded by one helper (`evaluation.ms_round`).

**Alternatives considered.**

Quoting headline values in markdown was rejected because it is the main source of inconsistencies between runs.

**Consequences.**

Specific values must be read from the regenerated files in `outputs/` after a clean run.

---

### ADR-008 — Withholding the Monitoring Record and the Hourly Data Derived From It

**Context.**

The monitoring record comes from an occupied private home, and the station data are third-party data with their own licence.

**Decision.**

The code is released under the MIT Licence and the tables, figures and model files in `outputs/` and `models/` under CC BY 4.0. The raw workbook is not distributed, in order to honour confidentiality agreements and protect occupant privacy. The hourly files in `data/processed/` are also withheld, because they contain the cleaned hourly record and forecasts from which it could be reconstructed. MIDAS observations are not redistributed; users obtain them from CEDA. The structure of the withheld workbook is documented in `DATASHEET.md`.

**Alternatives considered.**

Publishing an anonymised hourly record was rejected because a multi-year hourly room-level series from one home cannot be fully protected against re-identification.

**Consequences.**

The notebooks cannot be re-run from public files alone; the results of the reported run are provided in `outputs/` and `models/`, and the notebooks run unchanged on any workbook of the same structure.

[Back to top](#️-haven--architecture-decision-records)

---

# 📍 Notebook 01 — Data cleaning, quality control and imputation

### ADR-009 — Rate-of-Change Screening Calibrated Against Known Weather Transients

**Context.**

A spike filter must remove sensor artefacts without removing genuine outdoor weather changes or solar-driven warming indoors.

**Decision.**

Values outside physical bounds are flagged, and hour-to-hour changes are screened with separate limits for the outdoor and indoor loggers (outdoor 6.5 °C h⁻¹ and 45% RH h⁻¹; indoor 5.0 °C h⁻¹ and 25% RH h⁻¹). The outdoor limits sit just above three independently confirmed storm and frontal transients (`QC_REFERENCE_TRANSIENTS`), which the filter must retain. A centred robust z-score is also computed, but is used for comparison rather than as the primary screening method.

**Alternatives considered.**

Adopting the robust z-score as the main filter was rejected because its local baseline follows the diurnal cycle and flags legitimate solar warming.

**Consequences.**

The screening retains confirmed weather events, and the flags of both methods are reported. Dew point is recalculated after any correction.

---

### ADR-010 — Imputation Method by Gap Mechanism, Chosen by Benchmark

**Context.**

Missing data arise through two mechanisms: short logger dropouts inside the record, and the later installation of the Hall and Bedroom 4 loggers (a 1,488 h gap at the start).

**Decision.**

Interior gaps up to 24 h (`MAX_PCHIP_GAP_HOURS`) are filled by monotone piecewise cubic Hermite interpolation (PCHIP); longer gaps and the boundary gap are filled by linear regression on the most correlated zone of the same floor and orientation, fitted on the 30 days next to the gap. The methods are benchmarked by masking known values in two zones at gap lengths of 1–168 h and by withholding a boundary gap of the same length as the real one.

**Alternatives considered.**

Linear interpolation and cubic splines were benchmarked and not adopted; splines overshoot on longer gaps. Leaving the boundary gap empty would remove the first two months from the forecasting history of every zone.

**Consequences.**

Every filled value is logged with its method and fit statistics (`imputation_log.csv`) and marked in the provenance mask (`ADR-004`).

---

### ADR-011 — Recording the Software Environment With Every Run

**Context.**

Package updates (for example, pandas 3 used with older versions of statsmodels) can cause compatibility problems or alter computational results.

**Decision.**

Notebook 01 writes the Python and package versions of the run to `outputs/environment_versions.txt`, and `requirements.txt` pins the versions of the reported run.

**Alternatives considered.**

Relying on a minimum-version list was rejected because it does not identify the environment that produced the reported tables.

**Consequences.**

Differences between runs on different machines can be traced to package versions.

[Back to top](#️-haven--architecture-decision-records)

---

# 📍 Notebook 02 — Outdoor heat hazard and the station record

### ADR-012 — Two Heatwave Catalogues With the Same Rule

**Context.**

The on-site logger records the heat the dwelling experienced, while the nearest synoptic station is the closest stand-in for officially recognised heatwaves; the two can disagree.

**Decision.**

The Met Office definition (at least three consecutive days with a daily maximum of at least 27 °C, the Nottinghamshire threshold) is applied identically to the on-site logger over the whole record (`HWyyyy-k`) and to MIDAS Open observations from Sutton Bonington for the years available (`STyyyy-k`). A station day is included in the heatwave calculation only when at least 20 hourly observations are available.

**Alternatives considered.**

Using only one record was rejected: the on-site record alone would hide how much the conclusion depends on the outdoor source, and the station alone would not describe the conditions at the envelope.

**Consequences.**

Results that depend on heatwave dates are reported for both catalogues where both exist. The station comparison covers 2023–2025 only.

---

### ADR-013 — Site–Station Difference Treated as a Measured Contrast

**Context.**

The on-site logger reads warmer than the station. The difference could reflect neighbourhood microclimate, local shelter, logger siting or radiative heating of the sensor.

**Decision.**

The difference is stratified by station wind speed and preceding-day sunshine at night, and by sunshine at midday, and reported as an observed contrast; the stratified patterns are presented as supporting evidence, not as proof of causation.

**Alternatives considered.**

Correcting the on-site record to the station, or attributing the difference to an urban heat island, was rejected because two records cannot separate these explanations.

**Consequences.**

The difference is used to show how the choice of outdoor record changes the interpretation of indoor exposure, not to adjust any indicator.

---

### ADR-014 — Quality-Flagged Station Hours Listed and Retained

**Context.**

Some MIDAS hours carry quality code 106 during summer 2025.

**Decision.**

Flagged hours are listed per heatwave and retained, because their values are plausible against the logger and neighbouring hours.

**Alternatives considered.**

Removing flagged hours was rejected because it would break station days inside a heatwave and change the catalogue without evidence that the values are wrong.

**Consequences.**

The number of flagged hours is reported with each station heatwave.

[Back to top](#️-haven--architecture-decision-records)

---

# 📍 Notebook 03 — Features

### ADR-015 — Causal, Physically Motivated Features Built on the Complete Record

**Context.**

The forecasting model needs inputs that encode thermal inertia, multi-day heat and the daily cycle, available at the forecast origin.

**Decision.**

All predictors at forecast origin t are calculated exclusively from observations available at or before t: lags of 1–24 h, exponential moving averages (6, 24 and 72 h outdoors; 24 and 72 h indoors), the 24 h diurnal range, time encodings and psychrometric descriptors. Causality is checked in the notebook. Features are built on the complete (imputed) record because the model needs an unbroken history (`ADR-004`).

**Alternatives considered.**

Including future outdoor conditions in the features was rejected for the main model because they are not available when a warning is issued; it is used only as an explicit upper bound (`ADR-021`). Describing the features as "physics-guided" was rejected because no physical constraint enters the learning objective.

**Consequences.**

The forecasts can be issued operationally from measurements alone. The ablation and feature ladder in Notebook 04 test which feature groups add skill.

---

### ADR-016 — Psychrometric Variables as Descriptors, Not Strain Measures

**Context.**

Humidex and vapour-pressure deficit distinguish hot-humid from hot-dry conditions but are not physiological measurements.

**Decision.**

Humidex is computed from temperature and dew point (vapour pressure by the Magnus relation) and classified with the Environment Canada classes (30, 40 and above 45). It is used as a feature and as environmental context.

**Alternatives considered.**

Reporting Humidex classes as heat-strain outcomes was rejected because the classes were developed for outdoor conditions and are not validated for sleeping indoors.

**Consequences.**

Humidity-based results are interpreted as descriptions of the environment only.

[Back to top](#️-haven--architecture-decision-records)

---

# 📍 Notebook 04 — Forecasting

### ADR-017 — Ridge Regression as a Lightly Regularised ARX Model

**Context.**

A forecaster is needed for the bedroom temperatures from which warnings are issued, at 1–48 h, for a free-running dwelling with about two training summers.

**Decision.**

A separate multi-output Ridge model is fitted for each forecast horizon, with the seven indoor zones as simultaneous prediction targets and the penalty chosen by validation mean absolute error. With lagged indoor and outdoor inputs, the model has an autoregressive structure with exogenous inputs (ARX). Its use of lagged thermal measurements provides a data-driven representation of thermal memory, although it does not explicitly impose a resistance–capacitance network structure. The L2 penalty stabilises the fit in the presence of strongly correlated inputs.

**Alternatives considered.**

Tree ensembles were rejected as the main model because they cannot predict beyond the range of their training targets, while the test summer was hotter than training. Recurrent networks, Gaussian processes and foundation models were not pursued because the effective sample of heat events is small and the remaining error at 24–48 h comes mainly from unknown future weather. A systematic comparison of model families is outside the scope of HAVEN; tree ensembles are compared as a sensitivity analysis (Table S57).

**Consequences.**

The model is transparent and computationally inexpensive. Its under-prediction of the hottest nights is reported and handled by the warning layer (`ADR-033`), and tested with quantile regression (`ADR-023`).

---

### ADR-018 — Two Baselines That Need No Model

**Context.**

Indoor temperature is strongly autocorrelated, so a forecast must be judged against baselines rather than in absolute terms. Persistence alone is a weak baseline at 6–12 h because it ignores the daily cycle.

**Decision.**

Every result is compared with persistence (the forecast for t + k equals the observation at t) and with a climatology-plus-anomaly baseline (the training-period climatology at the target hour and day of year, ±7 days, plus the current anomaly damped by its lag-k coefficient), both evaluated on identical test origins.

**Alternatives considered.**

Persistence alone was rejected because at 6–12 h it is half a diurnal cycle out of phase, which flatters any model that knows the time of day.

**Consequences.**

The gain of Ridge is reported against the stronger baseline at each horizon, and the same baselines are carried into the warning analysis.

---

### ADR-019 — Purged Chronological Split With a Second Fold

**Context.**

Random splits leak information between adjacent hours, and a single test summer gives few episode onsets.

**Decision.**

Training runs to 30 April 2025, validation from 1 May to 30 November 2025, and testing from 1 December 2025 to the end of the record, so the 2026 summer is out of sample. Origins whose target would cross a boundary are removed. A second chronological fold (training to April 2024, validation May–November 2024, and testing December 2024–November 2025) re-estimates the models, scaling and baselines using the data available within that fold. It provides a separate temporal evaluation covering the 2025 summer.

**Alternatives considered.**

Random or blocked cross-validation across the whole record was rejected because it uses future summers to predict past ones.

**Consequences.**

Results describe temporal generalisation within one dwelling. The second fold is reported as a sensitivity analysis; the primary configuration is declared in each notebook header.

---

### ADR-020 — Moving-Block Bootstrap With Seven-Day Blocks

**Context.**

Hourly forecast errors are autocorrelated, so resampling hours would understate uncertainty.

**Decision.**

Confidence intervals for accuracy and paired differences resample contiguous seven-day blocks of forecast origins (500 replicates).

**Alternatives considered.**

Hourly or daily resampling was rejected because errors remain correlated over several days during heat.

**Consequences.**

The resulting intervals account for temporal dependence and are more appropriate than intervals based on independent hourly observations; a difference is described as clear only when its interval excludes zero.

---

### ADR-021 — Sensitivity Analyses of the Forecast Inputs

**Context.**

The forecasts could depend on imputed values, on each bedroom's own sensor, or on the lack of a weather forecast.

**Decision.**

Ridge is refitted (i) without training origins whose 144 h history or target touches an imputed value, (ii) for each bedroom without any of its own features, and (iii) with the measured outdoor temperature at the target time and its 24 h mean and maximum, an upper bound on what a weather forecast could add. Validation and test forecasts of (ii) and (iii) are kept for the warning analysis.

**Alternatives considered.**

Using archived numerical weather forecasts was not possible for the study period; the perfect-weather bound states the most such inputs could add.

**Consequences.**

The analysis shows how much skill depends on each input and how much is left to gain from weather forecasts.

---

### ADR-022 — Ablation and Feature Ladder to Explain the Skill

**Context.**

A full feature set does not show which information makes the forecast work.

**Decision.**

Two analyses are reported: an ablation that removes one design element at a time from the full model, and a feature ladder that adds one physically interpretable group at a time (own history, outdoor forcing, thermal memory, psychrometrics, time, other rooms), each refitted with the penalty re-selected and compared with a paired block-bootstrap interval.

**Alternatives considered.**

Relying on feature attribution to explain the skill was rejected because correlated inputs make attributions describe how the model spreads weight, not how much information each source adds (`ADR-039`).

**Consequences.**

The contribution of each group of inputs is measured directly as a change in test error.

---

### ADR-023 — Linear Quantile Regression as a Test of the Hot-Night Bias

**Context.**

Ridge predicts the conditional mean and under-predicts the hottest bedroom nights; it is unclear whether this comes from the squared loss or from the information available.

**Decision.**

Linear quantile regressions of bedroom night-time temperatures at the median and the 0.9 quantile are fitted with the same inputs at 24 and 48 h, with the L1 penalty chosen by validation pinball loss, and their forecasts enter the warning analysis like any other model.

**Alternatives considered.**

Distributional neural models were not pursued for the reasons in `ADR-017`.

**Consequences.**

If the median regression shows a similar bias, the under-prediction is attributed to missing information rather than to the model form.

[Back to top](#️-haven--architecture-decision-records)

---

# 📍 Notebook 05 — Exposure, episodes and recovery

### ADR-024 — Night-Time Exposure as Cumulative Exceedance Above 26 °C

**Context.**

Counts of exceedance hours do not distinguish a night that barely passes the threshold from one that never cools.

**Decision.**

The principal indicator is I_night, the sum over the night (22:00–07:00) of the hourly exceedance above 26 °C, in °C·h, reported with exceedance hours, exceedance nights and no-relief nights. All indicators are recomputed at 24, 25 and 27 °C.

**Alternatives considered.**

A single seasonal exceedance count was rejected because it loses magnitude and timing.

**Consequences.**

The indicators can be recomputed for any protection level. The threshold is a screening level (`ADR-006`).

---

### ADR-025 — Indoor Heat Episodes Defined From the Bedrooms

**Context.**

Defining events by outdoor heatwave dates would assume what the study tests, that indoor exposure follows outdoor heat.

**Decision.**

An indoor heat episode is defined as at least three consecutive complete nights during which each bedroom records at least one hourly air-temperature value above 26 °C; a night without complete measurements in any bedroom breaks the run. Episodes are matched to on-site and station heatwaves with a one-day tolerance, and heatwaves without an episode are listed with the reason.

**Alternatives considered.**

Episodes requiring two of three bedrooms, two or four nights, and other thresholds are reported as a sensitivity grid.

**Consequences.**

Indoor timing can be compared with outdoor heat, including episodes without a heatwave and heatwaves without an episode.

---

### ADR-026 — Seasonal Baseline for Recovery

**Context.**

Recovery is measured against a reference temperature, which itself moves through the summer.

**Decision.**

The main baseline is the room's median nightly mean within ±15 days of the episode midpoint, excluding episode and heatwave nights, with a 1 K tolerance; a pre-event baseline (nights 4–10 before onset) and a 0.5 K tolerance are reported for comparison. Follow-up is limited to 21 nights, and a night with incomplete data ends follow-up as censored.

**Alternatives considered.**

The pre-event baseline was not used as the main reference because its window already contained heat in a substantial share of room-episodes.

**Consequences.**

Recovery times depend on a stated reference, and the baseline choice is reported with the results.

---

### ADR-027 — Decay Half-Life as an Empirical Descriptor

**Context.**

A recovery rate can be fitted to the cooling after a peak, but in a free-running home the cooling depends on the following weather and on occupants' actions as well as on the fabric.

**Decision.**

A free-asymptote exponential relaxation is fitted to the 72 h after each room's peak by bounded least squares, fits with R² ≤ 0.3 are excluded, and the half-life is described as an empirical descriptor of the room's observed post-peak cooling, not as a building time constant. Window length, R² filter and the start of the fit are varied as sensitivity analyses.

**Alternatives considered.**

A grey-box resistance–capacitance model to separate building response from forcing was outside the scope of this observational study.

**Consequences.**

The half-life is reported with its fit statistics, the share of fits at the parameter bound and its reliability (`ADR-028`) before any interpretation.

---

### ADR-028 — Split-Half Reliability Before Comparing Rooms

**Context.**

A low correlation between exposure and a recovery indicator could mean the two are distinct, or that the recovery indicator is too noisy to rank rooms.

**Decision.**

Every room-level indicator is tested for split-half reliability: the 12 episodes are split at random into halves of six (2,000 splits), each zone is summarised by its median in each half, and the Spearman correlation across zones between the halves is reported, with the fixed odd–even split. Because the seven-zone correlation partly reflects the contrast between the ground and first floors, the three bedrooms are also tested on their own with Kendall's coefficient of concordance W across the episodes in which all three have a usable value, with a permutation test that shuffles the bedroom ranks within each episode (2,000 permutations).

**Alternatives considered.**

Interpreting correlations between indicators without a reliability estimate was rejected because any correlation with an unreliable indicator is attenuated towards zero.

**Consequences.**

Only indicators whose room ranking reproduces between independent halves of the events, and among the bedrooms across episodes, are used to compare rooms.

---

### ADR-029 — TM59 Criterion (b) Applied to Measured Air Temperature

**Context.**

CIBSE TM59 is a design-stage method based on modelled operative temperature; the loggers measure air temperature.

**Decision.**

The study applies the numerical thresholds and assessment period associated with TM59 Criterion (b) (TM59:2026, Section 2.4.2 and Table 2) to measured bedroom air temperatures: 23:00–08:00 sleeping hours in May–September, with nights evaluated when at least six readings are available. A failure is recorded when the night mean exceeds 27 °C (Category II) or 26 °C (Category I); a pass requires no more than four failure nights per season. The six-reading requirement is this study's completeness rule for measured data, not a requirement specified by TM59. Counts are cross-checked against the raw workbook, and each failure night is linked to its indoor episode and heatwave. Because operative temperature is not measured, these results are indicative and do not constitute a formal TM59 compliance assessment.

**Alternatives considered.**

Aligning the TM59 sleeping period with the 22:00–07:00 exposure night was rejected because it would no longer be the criterion.

**Consequences.**

TM59 results are indicative, and their relationship to I_night is reported rather than assumed.

---

### ADR-030 — Operative-Temperature Offset as a Sensitivity

**Context.**

Mean radiant temperature was not measured, so operative temperature is unknown.

**Decision.**

As a sensitivity analysis, an estimated night-time operative-minus-air temperature offset of −0.06 to −0.15 K is applied to bedroom indicators and TM59 calculations. The offset is derived from a steady-state inner-surface heat-balance calculation using the dwelling's envelope U-values. It is an approximation, not a direct measurement of operative temperature.

**Alternatives considered.**

A dynamic radiant model was rejected because surface temperatures were not measured.

**Consequences.**

The size and direction of the air-temperature proxy effect are reported for each indicator.

---

### ADR-031 — Year Comparisons With Outdoor Heat Held Fixed

**Context.**

Exposure rises sharply in hotter years, and because I_night counts heat above 26 °C it grows faster than linearly with outdoor heat.

**Decision.**

Year effects are estimated using two regression specifications. The first uses least squares; the second uses a Poisson-family model with a log link. Both include outdoor night-time degree-hours above 18 °C, their squared term, the preceding day's outdoor maximum and year indicators, with heteroscedasticity- and autocorrelation-consistent (HAC) standard errors.

**Alternatives considered.**

Ratios of exposure to outdoor degree-hours were rejected as the main comparison because they rise with outdoor heat even for an unchanged building.

**Consequences.**

A change in exposure between years is attributed to the building only if it remains with outdoor heat held fixed.

[Back to top](#️-haven--architecture-decision-records)

---

# 📍 Notebook 06 — Night-time warnings

### ADR-032 — Hot Bedroom Night as the Warning Event

**Context.**

A warning needs an event definition that occupants can act on and that can be scored night by night.

**Decision.**

An observed event is a complete measured bedroom night with at least one hour above 26 °C. A warning is derived from the forecast night maximum, defined as the largest of the nine hourly forecasts at a fixed horizon of 24 or 48 h. A stricter event, a night whose 22:00–07:00 mean exceeds 26 °C, is reported as a sensitivity analysis.

**Alternatives considered.**

Warning on episodes rather than nights was rejected because there are too few episodes to score.

**Consequences.**

The warning event is distinct from an indoor heat episode, and the two definitions are kept separate throughout.

---

### ADR-033 — Settings Chosen on the Validation Period by CSI

**Context.**

Without a precautionary margin, a model that under-predicts the hottest nights misses them; choosing the margin on the test period would overstate skill.

**Decision.**

For each model and horizon, the margin (0–2 K) and the probability threshold are chosen on the validation period as the settings with the highest critical success index, ties going to the smaller setting, and applied unchanged to the test period. The full trade-off between detection and false alarms is reported.

**Alternatives considered.**

Choosing settings by cost–loss ratios was rejected because costs were not available; relative economic value is reported across cost–loss ratios instead (`ADR-037`).

**Consequences.**

Settings chosen on a cool validation summer may not transfer; this is tested with a second fold and with the primary-split margin carried over unchanged (`ADR-036`).

---

### ADR-034 — Analogue Probability With an Updating Error Pool

**Context.**

A probability of a hot night should reflect the errors the model makes under similar forecast conditions, without using information that would not be available operationally.

**Decision.**

The probability estimate is based on the empirical distribution of forecast errors from the 60 historical bedroom-nights with the closest forecast night maxima. The pool contains the validation nights and every test night already observed at the issue time; validation probabilities exclude the night itself and the three nights either side. The 5th and 95th percentiles of the same errors give a 90% interval. A static pool and pool sizes of 30 and 120 are sensitivity analyses.

**Alternatives considered.**

A parametric error distribution was rejected because errors are skewed on hot nights.

**Consequences.**

No future information enters a probability. Calibration, reliability and interval coverage by condition are reported.

---

### ADR-035 — First-Night Test, Also at Matched False Alarms

**Context.**

On the first night of an episode, a baseline that carries forward the previous night cannot anticipate the onset, so first nights test anticipation. Comparing models at their own chosen settings mixes forecast quality with how many false alarms each tolerates.

**Decision.**

First nights warned are counted for each bedroom and test episode, with exact (Clopper–Pearson) intervals, and also compared at equal numbers of false alarms by scanning all settings on the test period; the latter is labelled as descriptive.

**Alternatives considered.**

Reporting first nights only at the chosen settings was rejected because a model with a larger margin would appear better at anticipation.

**Consequences.**

First-night results are reported with the number of episodes and bedroom first nights they rest on.

---

### ADR-036 — Robustness Across Test Summers and Settings

**Context.**

A single test summer with five episodes cannot show whether a warning result is general.

**Decision.**

The warning analysis is repeated on the 2025 test summer of the second fold, with settings chosen on its own validation period. The 1.0 K margin chosen for Ridge on the primary-split validation period is also carried over unchanged to the second fold. It was not declared before results were seen, and because it was selected on May–November 2025, which overlaps the second fold’s test period, the 2025 result at this margin is an in-sample check reported as a sensitivity analysis only.

**Alternatives considered.**

Reporting only the primary split was rejected because the first-night result could depend on one summer.

**Consequences.**

Claims about warning skill are limited to those that hold in both summers.

---

### ADR-037 — Units of Analysis and Effective Sample Sizes

**Context.**

Hours are autocorrelated, bedrooms share the weather and nights within an episode are linked, so counts of hours overstate the evidence.

**Decision.**

Each result is reported with the unit that carries its uncertainty: seven-day blocks for accuracy and CSI differences, episodes for exposure–recovery comparisons, and bedroom first nights for the first-night test. A table of effective sample sizes is written with the results.

**Alternatives considered.**

Treating bedroom-nights as independent was rejected because it would understate uncertainty.

**Consequences.**

Uncertainty intervals may be wide; findings are interpreted as evidence specific to the monitored dwelling.

---

### ADR-038 — Comparator Models for the Warning Layer

**Context.**

The warnings are derived from a temperature forecast; a model that predicts the event directly might do better.

**Decision.**

A logistic regression predicting the hot night from the same scaled inputs at the issue time of the latest forecast used by the warning (06:00 on the day of the night for 24 h, the day before for 48 h) is fitted per bedroom and lead, with the L2 penalty chosen by validation Brier score, and compared with the probability warning on the same nights.

**Alternatives considered.**

Classifiers with more parameters were rejected because each bedroom has few hot training nights.

**Consequences.**

The comparison shows whether forecasting temperature and then setting a precaution is preferable to predicting the event directly.

---

### ADR-039 — Exact Linear Attribution, Grouped by Source and Labelled as Weight

**Context.**

Feature attribution is often read as evidence of what drives a forecast, but lags and averages of the same series are strongly correlated.

**Decision.**

For the linear Ridge model, feature contributions are calculated as the product of each fitted coefficient and the corresponding centred input value, then aggregated by spatial source and source type. The result is described as how the model spreads weight among correlated signals; the information each source adds is measured by the feature ladder (`ADR-022`).

**Alternatives considered.**

Reporting attribution as evidence of heat transfer between rooms, or of a gain from the multi-zone formulation, was rejected.

**Consequences.**

Attribution is a supplementary diagnostic, not a finding.

---

### ADR-040 — Worked Example Labelled as Illustration

**Context.**

Translating warnings into actions makes the output concrete, but no action was taken or observed.

**Decision.**

The worked example for the largest test episode lists the nightly probabilities, warnings and night-ventilation hours with suggested passive measures (shading, ventilation when outdoor air is cooler, use of the coolest bedroom), described as rules of thumb, not tested interventions.

**Alternatives considered.**

Reporting the example as evidence of avoided exposure was rejected because occupant responses were not observed.

**Consequences.**

The example illustrates use; evaluating whether warnings change behaviour or reduce exposure requires prospective study.

[Back to top](#️-haven--architecture-decision-records)
