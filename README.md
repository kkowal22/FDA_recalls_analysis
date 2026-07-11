# FDA Class I Drug Recall Analysis

This repository explores **Class I FDA drug recall data (2012–2026)** through a series of independent, question-driven analyses. Each subfolder tackles a distinct question using the same underlying recall dataset, with its own pipeline, notebook(s), and write-up.

## Structure

Each analysis lives in its own subfolder and is self-contained:
## Analyses

| Folder | Question | Status |
|---|---|---|
| [`FDA Class I Drug Recall — Administrative Latency & Ingestion Pipeline`](./FDA%20Class%20I%20Drug%20Recall%20%E2%80%94%20Administrative%20Latency%20%26%20Ingestion%20Pipeline) | How long does it take recalls to move through FDA administrative stages? | ✅ Complete |
| *More coming soon* | — | 🔜 Planned |

## Data Source

All analyses draw from the [FDA's public Class I drug recall enforcement data](https://open.fda.gov/apis/drug/enforcement/), covering 2012–2026.

## How to Use This Repo

- Browse the table above to find an analysis of interest.
- Each subfolder's own README documents its specific question, methods, and conclusions in detail.
- Shared/reusable ingestion logic (where applicable) is noted in each analysis's `src/` folder.

## Roadmap

This repo will grow to include additional angles on the same dataset — e.g., recall severity trends, manufacturer/company patterns, geographic distribution, and regulatory bottlenecks. New analyses will be added as new subfolders with an entry in the table above.
