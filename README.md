# FDA Class I Drug Recall Analysis
This repository explores **Class I FDA drug recall data (2012–2026)** through a series of independent, question-driven analyses. Each subfolder tackles a distinct question using the same underlying recall dataset, with its own pipeline, notebook(s), and write-up.
## Structure
Each analysis lives in its own subfolder and is self-contained:
## Analyses
| Folder | Question | Status |
|---|---|---|
| [`FDA Class I Drug Recall Classification (2013–2026): A Decade-Long Decline, Not a Backlog`](./FDA%20Class%20I%20Drug%20Recall%20Classification%20%282013%E2%80%932026%29%3A%20A%20Decade-Long%20Decline%2C%20Not%20a%20Backlog) | Is FDA classification latency (Recall Initiation → Center Classification) trending over time, and is any trend explained by recall-volume backlog? | ✅ Complete |
| *More coming soon* | — | 🔜 Planned |
## Data Source
All analyses draw from the [FDA Data Dashboard's advanced search portal](https://www.accessdata.fda.gov/scripts/ires/index.cfm#tabNav_advancedSearch), covering recalls from June 8, 2012 onward. Individual analyses may restrict this window further — check each subfolder's README for its exact date range and any filtering applied.
## How to Use This Repo
- Browse the table above to find an analysis of interest.
- Each subfolder's own README documents its specific question, methods, and conclusions in detail.
- Shared/reusable ingestion logic (where applicable) is noted in each analysis's `src/` folder.
## Roadmap
This repo will grow to include additional angles on the same dataset — e.g., recall severity trends, manufacturer/company patterns, geographic distribution, and regulatory bottlenecks. New analyses will be added as new subfolders with an entry in the table above.
