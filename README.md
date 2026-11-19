# P1 — Ranking agreement between cheap and expensive robustness evaluations

A small, pre-planned re-analysis of the public robustness dataset of Jung, Lukasik and Keuper (ICLR 2023) for NAS-Bench-201 architectures.

**Question:** if only a cheap evaluation is affordable (clean accuracy or FGSM at a small ε), how much of the architecture ranking under strong attacks (APGD-CE, Square) is recovered, especially among the best architectures? And is the loss of agreement under strong attacks a floor effect?

**What is new relative to the dataset paper:** Jung et al. (ICLR 2023) already report global rank correlations between attacks (Kendall τ) and a NAS experiment that searches on FGSM accuracy. This project does not repeat that as a contribution. It asks how much of the robustness gap left by FGSM-guided search is caused by the cheap objective itself rather than by the search, how well cheap evaluations select the best architectures (top-k overlap, regret, absolute accuracy), how much attack cost is needed for that, and whether near-zero accuracies under strong attacks explain the loss of agreement.

**Status:** the analysis plan was fixed on 19 November 2026, before any result was computed: [`docs/analysis_plan.md`](docs/analysis_plan.md). Results will follow in this repository.

## Data and credit

- Dataset: Jung, Lukasik, Keuper, *Neural Architecture Design and Robustness: A Dataset*, ICLR 2023 — http://data.robustness.vision/
- Helper code: https://github.com/steffen-jung/robustness-dataset
- The data are not copied into this repository.

## Author

Mahran Rezk
