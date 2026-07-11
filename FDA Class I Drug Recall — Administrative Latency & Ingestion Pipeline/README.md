# FDA Class I Drug Recall — Classification Latency Analysis

## Project Overview
This pipeline investigates whether the FDA's drug recall classification process slows down over time or during high-volume recall years. It utilizes publicly available Class I drug recall records pulled from the FDA Data Dashboard covering operations from June 8, 2012, through July 5, 2026.

The core investigation centers on a specific macro-metric: **Does the time elapsed between a firm initiating a recall and the FDA Center formally classifying it ($Recall\ Initiation\ Date \rightarrow Center\ Classification\ Date$) show a meaningful trend over time?** More importantly, is that trend a reflection of real regulatory delays, or simply an artifact of how the raw dataset is structurally organized?

---

## Project Structural Architecture

| Goal (Why) | Means (How) | Characteristics (What) | Target Data (Where) | Workflow (When) | Roles (Who) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Investigate backlogs:** Evaluate if administrative latency grows or spikes during high-volume recall years. | **Automated Pipeline:** Programmatic data ingestion, type-coercion, and structural data validation routines. | **Class I Categorization:** Focuses on the most severe recall tier involving high-risk medical threats. | **FDA Data Dashboard:** Publicly available historical datasets from June 8, 2012, to July 5, 2026. | **Post-Ingestion Gate:** Standardizes variables and executes an operational quality check before building charts. | **Data Scientist / Auditor:** Single-investigator execution handling end-to-end extraction and visualization. |
| **Expose structural skewing:** Trace how uncollapsed item-level data artificiality distorts aggregate metrics. | **Unit-of-Analysis Correction:** De-duplicating SKU clusters down to standalone unique `Event ID` entities. | **SKU-to-Event Mapping:** Resolves many-to-one relationships where single actions map to hundreds of rows. | **Target Dataset:** Isolated itemized product spreadsheets containing repeated administrative event records. | **Prior to Aggregation:** Deduplication runs immediately before calculating annual descriptive metrics. | **Data Architect / Reviewer:** Auditing raw reporting patterns vs. true regulatory definitions. |

---

## Pipeline Architecture

### Section 1 — Environment Setup
* Installs and imports `pandas`, `numpy`, and `altair`. 
* Configures Altair's JSON data transformer (available and commented out by default) to handle memory allocation for datasets exceeding 5,000 rows.

### Section 2 — Data Loading
* Reads the raw CSV input and outputs the total record count prior to execution.
* Establishes a concrete baseline for the downstream data quality audit trail.

### Section 3 — Date Standardization
* Converts `Recall Initiation Date` and `Center Classification Date` strings into proper datetime objects using `pd.to_datetime(..., errors='coerce')`. 
* Preserves raw, pre-conversion copies of both columns to cleanly isolate missing values from corrupted, unparseable text strings.

### Section 4 — Data Quality Audit & Quarantine Gate
Cross-references the raw and converted date columns to classify every row into one of three structural states per field:
1. **Originally Blank:** No data was ever entered.
2. **Unparseable Text:** A value was entered but could not be parsed as a valid date due to a typo or formatting error.
3. **Clean:** Parsed successfully.

* **Strict Operational Gate:** Any record containing an anomaly in either date field is immediately isolated into a quarantine DataFrame. 
* If quarantined rows exist, the script prints an explicit diagnostic sample to the console and halts further execution via a `raise ValueError`. This prevents downstream visualization models from running silently on corrupted data.
* Row deduplication is intentionally bypassed at this stage to preserve the true row-level scope of data quality profiles.

### Section 5 — Latency Metric, Deduplication, and Trend Visualization
* **Metric Creation:** Computes `Latency_Days` as the explicit day delta between classification and initiation events, and extracts the calendar `Year` for annual grouping.
* **Unit-of-Analysis Correction:** The raw source data is logged at the SKU/item level, meaning a single recall action can generate hundreds of near-identical rows if it covers various pack sizes or dosages sharing a single `Event ID`. Computing a median directly on raw rows allows single massive events to dominate an entire year's metric. (Verified in this dataset where an uncorrected 465-row event inflated annual median latency by nearly 7x). The pipeline corrects for this by collapsing the data to one row per unique `Event ID` via `df.drop_duplicates(subset=['Event ID'])`.
* **Annual Aggregation:** Groups deduplicated data by year to calculate total annual unique recall events (volume) and the median processing latency. A median metric is used over the mean to eliminate skewing from extreme administrative outlier delays.
* **Visualization:** Renders an interactive Altair chart layering annual recall volume (vertical bars) against median processing latency (trend line) using dual independent Y-axes, complete with an interactive horizontal brush selector.

---

## Data Source & Known Limitations

* **Domain Bounds:** Data is sourced from the FDA Data Dashboard, filtered strictly to `Product Type = Drugs` and `Classification = Class I`. Findings reflect drug-center behavior specifically and do not model FDA-wide recall processing for foods, medical devices, or biologics.
* **Temporal Truncation:** The dataset's June 8, 2012 start boundary is a fixed limitation of the underlying FDA system architectures (recalls are only displayed once classified on or after that date) and is not an artifact of local pipeline extraction.
* **Metric Definition:** `Latency_Days` measures administrative, internal paperwork processing time between a firm's disclosure and formal center classification. It does not map product removal speed from physical retail shelves or public communication timing.
