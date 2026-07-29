# FDA Class I Drug Recall Analysis
This repository explores **Class I FDA drug recall data (2012–2026)** through a series of independent, question-driven analyses. Each subfolder tackles a distinct question using the same underlying recall dataset, with its own pipeline, notebook(s), and write-up.
## Structure
Each analysis lives in its own subfolder and is self-contained:
## Analyses
| Folder | Question | Status |
|---|---|---|
| [`FDA Class I Drug Recall Classification (2013–2026): A Decade-Long Decline, Not a Backlog`](https://github.com/kkowal22/FDA_recalls_analysis/tree/main/FDA%20Class%20I%20Drug%20Recall%20%E2%80%94%20Administrative%20Latency%20%26%20Ingestion%20PipelineFDA%20Class%20I%20Drug%20Recall%20Classification%20(2013%E2%80%932026)%3A%20A%20Decade-Long%20Decline%2C%20Not%20a%20Backlog) | Is FDA classification latency (Recall Initiation → Center Classification) trending over time, and is any trend explained by recall-volume backlog? | ✅ Complete |
[`Class_2_3_Latency_Replication`](https://github.com/kkowal22/FDA_recalls_analysis/tree/main/Class_2_3_Latency_Replication) | Does the Class I administrative latency decline and tail-risk improvement replicate across lower-severity Class II and III recalls? | 🔄 In Progress |
| *More coming soon* | — | 🔜 Planned |
## Data Source
All analyses draw from the [FDA Data Dashboard's advanced search portal](https://www.accessdata.fda.gov/scripts/ires/index.cfm#tabNav_advancedSearch), covering recalls from June 8, 2012 onward. Individual analyses may restrict this window further — check each subfolder's README for its exact date range and any filtering applied.
## How to Use This Repo
- Browse the table above to find an analysis of interest.
- Each subfolder's own README documents its specific question, methods, and conclusions in detail.
- Shared/reusable ingestion logic (where applicable) is noted in each analysis's `src/` folder.
## Roadmap
This repo will grow to include additional angles on the same dataset — e.g., recall severity trends, manufacturer/company patterns, geographic distribution, and regulatory bottlenecks. New analyses will be added as new subfolders with an entry in the table above.
