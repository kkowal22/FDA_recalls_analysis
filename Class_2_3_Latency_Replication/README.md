# Systemic Reform or Targeted Priority? Evaluating Latency and Queue-Depth Behavior Across FDA Recall Classifications

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/kkowal22/FDA_recalls_analysis/blob/main/Class_2_3_Latency_Replication/class23_latency.ipynb)

## Overview
This directory contains a comparative replication analysis examining whether the decade-long administrative latency decline (2013–2019) and tail-risk improvement observed in **Class I FDA drug recalls** replicate across lower-severity **Class II and Class III recalls**. 

While Class I recalls represent immediate, life-threatening risks, Class II and Class III recalls comprise the vast majority of total recall volume. By scaling up the dataset to include these lower-priority tiers, this analysis tests whether FDA operational enhancements reflected agency-wide systemic reforms or targeted prioritization strictly limited to high-severity events.

---

## Core Research Questions

1. **Latency Trend Replication:** Does Class II/III administrative latency (Recall Initiation → Center Classification) follow the same sharp 2013→2019 decline-then-plateau shape, or is the plateau unique to Class I recalls?
2. **Workload Independence & Statistical Power:** Does the volume-latency independence observed in Class I ($r=0.127, p=0.745$) hold for Class II/III, or does workload start to impact processing times when statistical power increases across a significantly larger sample size ($N$)?
3. **Tail-Risk Backlog Reduction:** Does the $p_{90}$ tail-risk improvement ($299 \rightarrow 61$ days) show a comparable magnitude in Class II/III, or is eliminating worst-case backlogs uniquely a Class I achievement?
4. **Queue-Depth Dynamics:** Does the open-caseload time series show a persistently higher queue depth for Class II/III recalls, corroborating or contradicting the displacement hypothesis?

---

## Pipeline Architecture *(In Progress)*

> **Status Note:** Steps 1–5 are completed and validated. Data concatenation, standardization, and imputation (Steps 6 & 7) are currently **In Progress** as schema and null handling logic are refined.

| Step # | Notebook Code Block | Action Executed in Code | Technical / Analytical Rationale |
| :---: | :--- | :--- | :--- |
| **1** | **Environment Setup & Storage Mount** | Mounted Google Drive to Colab (`/content/drive`) to expose cloud directory paths. | Establishes physical path access to raw data storage before initiating file reading operations. |
| **2** | **Libraries Installation & Imports** | Installed `pandera` and imported standard data engineering libraries (`glob`, `os`, `numpy`, `pandas`, `pandera`). | Loads execution dependencies for path matching, tabular manipulation, and schema validation into the active Python kernel. |
| **—** | **DATA PROCESSING** | *(Section Grouping)* | *Core workflow for data standardization, merging, and cleaning.* |
| **3** | **Lean Six Sigma Palette Configuration** | Defined `SIXSIGMA_PALETTE` dictionary establishing standardized hex codes for process visuals. | Configures global visual encoding standards early to ensure consistent chart styling throughout downstream analyses. |
| **4** | **File Path Matching & CSV Ingestion** | Set directory path, retrieved matching CSV files via `glob.glob()`, and read files into `df_list` while appending `source_file` metadata as the trailing column. | Locates all 22 batch files and loads them into memory, preserving record-level lineage back to individual raw source files. |
| **5** | **Pre-Merge Schema Validation Audit** | Normalized column names via `.str.strip().str.lower()`, extracted schema sets across all loaded DataFrames, and printed divergent/missing column reports. | Diagnoses column structural discrepancies (e.g., extra lot number fields in older files) across files prior to executing concatenation. |
| **6** *(In Progress)* | **Concatenation & Null Standardization Audit** | Merged `df_list` into `combined_df` via `pd.concat(ignore_index=True)`, converted empty spaces/placeholders to true `NaN`, and executed initial missing value audit. | Unifies 22 files into a single master structure (`df_clean`) and audits non-standard missing value flags across the unified record set. |
| **7** *(In Progress)* | **Multi-Type Data Imputation & Cleanup** | Applied categorical placeholders (`'Not Recorded'`, `'Unknown'`), converted date fields, filled numeric missing values with medians, and verified remaining nulls. | Systematically addresses missing data across text, date, and numeric field types to finalize a fully cleaned dataset ready for latency analysis. |

---

## Repository Structure

```text
Class_2_3_Latency_Replication/
├── README.md                      # Folder documentation
├── class23_latency.ipynb          # Main execution notebook (ingestion, audit, latency metrics)
└── Raw CSV Files/                  # 22 date-chunked raw CSV files (June 2012 – July 2026)
