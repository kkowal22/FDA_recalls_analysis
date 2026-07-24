# FDA Class I Drug Recall Classification (2013–2026): A Decade-Long Decline, Not a Backlog

## Project Overview
This pipeline investigates whether the FDA's drug recall classification process changes pace over time, and whether any such change tracks high-volume recall years. It utilizes publicly available Class I drug recall records pulled from the FDA Data Dashboard [1] covering firm recall initiations from January 1, 2013, through the data pull date of July 12, 2026.

The core investigation centers on a specific macro-metric: **Does the time elapsed between a firm initiating a recall and the FDA Center formally classifying it ($Recall\ Initiation\ Date \rightarrow Center\ Classification\ Date$) show a meaningful trend over time?** More importantly, is that trend a reflection of real regulatory delays/backlog pressure, or simply an artifact of how the raw dataset is structurally organized?

## Findings
Classification latency fell roughly 4x between 2013 and 2018, from a median of 141 days down to 35, then held flat through 2025. Concurrent recall volume doesn't explain that drop or the plateau that followed it (coef = 0.325 days/recall, p = 0.745). The change tracks *calendar year*, not caseload, which reads as a step-change from something that happened per year rather than a queueing effect.

---

## Project Structural Architecture

| Goal (Why) | Means (How) | Characteristics (What) | Target Data (Where) | Workflow (When) | Roles (Who) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Investigate backlogs:** Evaluate if administrative latency grows or spikes during high-volume recall years. | **Automated Pipeline:** Programmatic data ingestion, type-coercion, and structural data validation routines. | **Class I Categorization:** Focuses on the most severe recall tier involving high-risk medical threats [2]. | **FDA Data Dashboard:** Publicly available historical datasets, 2013–2026 [1]. | **Post-Ingestion Gate:** Standardizes variables and executes an operational quality check before building charts. | **Data Scientist / Auditor:** Single-investigator execution handling end-to-end extraction and visualization. |
| **Expose structural skewing:** Trace how uncollapsed item-level data artificiality distorts aggregate metrics. | **Unit-of-Analysis Correction:** De-duplicating SKU clusters down to standalone unique `Event ID` entities. | **SKU-to-Event Mapping:** Resolves many-to-one relationships where single actions map to hundreds of rows [3]. | **Target Dataset:** Isolated itemized product spreadsheets containing repeated administrative event records. | **Prior to Aggregation:** Deduplication runs immediately before calculating annual descriptive metrics. | **Data Architect / Reviewer:** Auditing raw reporting patterns vs. true regulatory definitions [2][4]. |
| **Isolate cause from correlation:** Test whether latency is driven by concurrent caseload/backlog rather than just the passage of time. | **Fixed-Effects Regression:** OLS of `Latency_Days` on a 45-day concurrent-volume window, controlling for `Year`. | **Backlog-vs-Load Test:** Distinguishes a genuine queueing effect (more concurrent work → slower processing) from a year-level step-change. | **Derived Field:** `Concurrent_Volume`, computed via a sorted-date window search on `Recall Initiation Date`. | **After Aggregation:** Runs on the deduplicated, dated event table once `Latency_Days` and `Year` exist. | **Data Scientist:** Specifies and interprets the regression model. |

---

## Pipeline Architecture

### Section 1 — Environment Setup
* Installs and imports `pandas`, `numpy`, `altair`, `statsmodels`, and `vl-convert-python`.
* Defines a shared Lean Six Sigma color palette used consistently across every chart, so color always carries the same meaning: blue = neutral process data/central tendency, grey = contextual/denominator data (no signal), amber = warning zone, red = confirmed out-of-spec/critical, dark neutral = statistical overlay (trend/fit line, not a verdict).

### Section 2 — Data Loading
* Reads three raw CSV extracts (Jan 2013–Dec 2018, Jan 2019–Dec 2023, Jan 2024–Jul 12 2026) sourced from the FDA's advanced search portal [1] and concatenates them into a single combined file.
* Filters to `Recall Initiation Date >= 2013-01-01` establishing the analysis window.

### Section 3 — Date Standardization
* Converts `Recall Initiation Date`, `Center Classification Date` and `Report Date` strings into datetime objects using `pd.to_datetime(..., errors='coerce')`.

### Section 4 — Unit-of-Analysis Correction
* The raw source data is logged at the SKU/item level: a single recall action can generate many near identical rows if it covers various pack sizes or dosages sharing one `Event ID`.
* The pipeline collapses to one row per unique `Event ID` via `df.drop_duplicates(subset=['Event ID'])`. This reduced the raw dataset from **1,675 item-level rows to 603 unique recall events**. It is the same method that FDA uses when presenting the results in the official dashboards. 

### Section 5 — Latency Metric, Right-Censoring Correction, and Backlog Signals
* **Metric Creation:** Computes `Latency_Days` as the day delta between `Center Classification Date` and `Recall Initiation Date` [3], and extracts the calendar `Year` from the initiation date.
* **Right-Censoring Correction:** Recent recalls that haven't been classified yet would otherwise bias latency *downward* for the most recent period (only the fast-resolving cases have a value. Slow ones are still open and effectively invisible). The pipeline computes `Report_Lag` (Report Date − Initiation Date), takes its 90th percentile as a safety margin, and flags an event `Reliable` only if it was initiated early enough before the July 12, 2026 pull date to have plausibly been resolved by now.
* **Backlog-Signal Metrics:** `total_recall_volume` (unique events/year) is computed on the **full** dataset, since a recall's initiation is known immediately and isn't subject to censoring. `median_latency`, `p90_latency`, `pct_over_60`, and `pct_over_90` are computed on the **Reliable subset only**, since all four are latency-derived and would otherwise be skewed toward artificially fast outcomes in recent years.

### Section 6 — Queue Depth (Open Caseload) Time Series
* For each month-end snapshot date, counts events where `Recall Initiation Date <= snapshot < Center Classification Date` — i.e., the number of recalls administratively "open" at that moment.
* This produces a proxy for real-time backlog size, independent of the annual aggregate metrics.

### Section 7 — Volume-vs-Latency Relationship Test (Backlog-vs-Load)
* **Concurrent Volume:** For each event, counts how many other recalls were initiated within a ±45-day window, computed efficiently via `np.searchsorted` on sorted dates.
* **Regression:** Fits `Latency_Days ~ Concurrent_Volume + C(Year)` via OLS, isolating the within-year effect of concurrent caseload on an individual event's latency, net of whatever year it happened in.
* **Detrending:** For visualization, both variables are also detrended by subtracting each year's group mean, so the scatterplot shows the relationship with year-level effects removed.

### Section 8 — Visualization
* Renders interactive Altair charts, all using the shared Lean Six Sigma palette: median vs. P90 latency by year (line), the share of recalls exceeding 60/90-day thresholds by year (bar), the monthly queue depth (area), the annual volume paired with the latency trend (stacked bar + line), and the detrended concurrent-volume-vs-latency scatter with a fitted regression line.

---

## Data Source & Known Limitations

* **Domain Bounds:** Data is sourced from the FDA Data Dashboard [1], filtered strictly to `Product Type = Drugs` and `Classification = Class I` [2]. Findings reflect Class I drug-center behavior specifically and do not model FDA-wide recall processing for foods, medical devices, biologics or other classes of drug recalls. 
* **Analysis Window:** The pipeline restricts analysis to recalls initiated on or after January 1, 2013 through the July 12, 2026 data pull. This is a chosen analysis boundary, not a hard system limitation. The underlying FDA Dashboard's own display boundary starts June 8, 2012 [1].
* **Metric Definition:** `Latency_Days` measures administrative, internal paperwork processing time between a firm's disclosure and formal center classification, per the field definitions in the FDA Enforcement Report API documentation [3]. It does not map product removal speed from physical retail shelves or public communication timing.
* **2026 Is Incomplete:** 2026 shows `NaN` for all latency-derived metrics with only 9 events initiated and a pull date mid-year. There isn't yet a reliable resolved sample at the time of generating this analysis. This is intentional (the right-censoring correction excludes it), not missing data.
* **Small Annual Sample Sizes:** Annual unique-event counts range from 28 to 65. At this scale, a handful of events can meaningfully shift a year's median or P90, and year-to-year metrics carry real sampling noise. The multi-year plateau pattern (below) is more trustworthy than any single year's exact figure.
* **Modeling Choices Not Independently Validated:** The 90th-percentile `Report_Lag` safety margin and the 45-day concurrent-volume window are both arbitrary analyst-chosen thresholds. The results were not stress-tested against alternative window sizes.
* **Causal Scope:** This analysis measures latency as an output metric and tests one specific causal hypothesis (concurrent volume/backlog). It does not model or control for other upstream drivers of processing time — e.g., FDA staffing levels, hiring patterns, budget cycles, or government shutdowns/lapses in appropriations. The regression result below rules out *caseload* as the driver. It does not identify what the actual driver was.

---

## Conclusion

**Yes — there is a meaningful trend, and it runs in the opposite direction "backlog worsening over time" would predict.**

* **Median latency fell sharply and then plateaued.** 141 days (2013) → 109 (2014) → 140 (2015) → 105 (2016) → 76 (2017) → 35 (2018), then held in a narrow 26–35 day band every year from 2019–2025.
* **The tail (P90) shows the same pattern, more dramatically.** 299 days (2013) → up to 358 (2015) → 136 (2017) → 74 (2018) → mostly 48–122 days in 2019–2025, i.e., worst-case processing time dropped by roughly 3–5x.
* **Share of recalls exceeding administrative thresholds collapsed correspondingly.** Recalls taking >60 days: 98.5% (2013) → 65% (2017) → 17% (2018) → single digits to high teens thereafter. Recalls taking >90 days: 80% (2013) → 37% (2017) → 7% (2018) → mostly under 15% thereafter.
* **2026 is excluded** from these figures (right-censored, `NaN`), consistent with the pipeline's methodology, not a data gap.

**Is this real, or an artifact of the data structure / recall volume?**

* It is not a deduplication artifact. All figures above are computed on the 603 unique events, not the 1,675 raw item-rows.
* It is not explained by caseload/backlog pressure. The raw correlation between an individual event's latency and its concurrent 45-day recall volume is weak (r = 0.13). Once year fixed effects are controlled for, the concurrent-volume coefficient is 0.325 days per additional concurrent recall, p = 0.745, 95% CI [−1.64, 2.29]. It is statistically indistinguishable from zero.
* In plain terms: **within any given year, how many other recalls were open at the same time did not predict how long a specific event took to classify.** The improvement is not a queueing phenomenon where lighter caseloads freed up faster processing. It presents as a step-change tied to something that happened per-year (process, staffing, policy, definitional, or system changes), not to how busy the Center was at any given moment.

**Net answer to the driving question:** the FDA's Class I drug-recall classification latency did not slow down over 2013–2026. It dropped substantially, with almost all of the improvement concentrated in 2013–2018, followed by a stable low-latency plateau through 2025. The data rules out concurrent recall volume as the explanation for either the drop or the plateau. It does not identify what did cause it, since staffing, policy, and procedural changes are outside this dataset's scope (see Causal Scope above).

---

## References
1. FDA Advanced Search Portal (data source): https://www.accessdata.fda.gov/scripts/ires/index.cfm#tabNav_advancedSearch
2. FDA Enforcement Report Information and Definitions: https://www.fda.gov/safety/enforcement-reports/enforcement-report-information-and-definitions
3. FDA Enforcement Report API Definitions: https://www.fda.gov/safety/enforcement-reports/enforcement-report-api-definitions
4. FDA Data Dashboard Glossary: https://datadashboard.fda.gov/oii/glossary.htm
