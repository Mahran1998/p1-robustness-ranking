# P1 — Ranking agreement between cheap and expensive robustness evaluations

A small re-analysis of the public robustness dataset of Jung, Lukasik and Keuper (ICLR 2023) for the 6,466 non-isomorphic NAS-Bench-201 architectures. The analysis plan was fixed in advance; see the timeline below.

## Question

If only a cheap evaluation is affordable (clean accuracy, FGSM, or a 40-step PGD attack), how much of the architecture ranking under strong attacks (APGD-CE, Square) does it preserve? Above all, does it select the best architectures (top-k)? And is the loss of agreement under strong attacks a floor effect?

## Relation to the dataset paper

Jung et al. already report global rank correlations between attacks (Kendall τ, their Fig. 3 and Fig. 23–24) and a NAS experiment that searches on FGSM accuracy. Our τ values reproduce known results and serve as a check.

The analysis focuses on:

- top-k selection: tie-aware overlap@k, top-5% mean accuracy, and top-1 regret;
- the clean-accuracy cost of selecting robust architectures;
- floor diagnostics.

That global agreement and top-k selection can differ is known in NAS (Ning et al., NeurIPS 2021). Here it is examined for cheap versus strong attack evaluations.

## Design (plan v1.6, [`docs/analysis_plan.md`](docs/analysis_plan.md))

- **Pairs:** 4 cheap evaluations (clean, FGSM ε = 0.5, FGSM ε = 1, PGD ε = 1) × 4 targets (APGD-CE ε = 1, Square ε = 1, APGD-CE ε = 4, Square ε = 4) × 3 datasets = **48 pairs**. ε is in units of 1/255.
- **Development and confirmation:** the code was developed on CIFAR-10 and frozen (commit `a2cf07f`). It was then run on CIFAR-100 and ImageNet16-120 with only `DATASET` and `N_CLASSES` changed.
- **Analysis set:** architectures with clean accuracy below twice the chance level are excluded (5, 5 and 7 architectures). All 6,466 architectures are analysed again as a sensitivity analysis.
- **Verdicts:**
  - **F** if more than 50% of architectures have target accuracy below 1% (floor-limited; descriptive only).
  - Otherwise a selection verdict from the lower bound of overlap@5%: PASS ≥ 0.50, WARN 0.25–0.50, FAIL < 0.25.
  - Outcome **A** = PASS/WARN with τ lower bound ≥ 0.5; **B** = PASS/WARN with weaker τ; **C** = FAIL.
- **Uncertainty:** paired bootstrap over architectures (B = 1000, seed 0). This measures sensitivity to the composition of the architecture set only, not seed or test-set uncertainty.

## Results (48 pairs)

Outcome and overlap@5% (point estimate):

| Target | Cheap | CIFAR-10 | CIFAR-100 | ImageNet16-120 |
|---|---|---|---|---|
| APGD-CE 1 | clean | C (0.00) | C (0.00) | F (0.00) |
| APGD-CE 1 | FGSM 0.5 | C (0.00) | C (0.29) | F (0.43) |
| APGD-CE 1 | FGSM 1 | C (0.00) | B (0.41) | F (0.80) |
| APGD-CE 1 | PGD 1 | A (0.75) | A (0.95) | F (0.98) |
| Square 1 | clean | C (0.25) | C (0.00) | C (0.00) |
| Square 1 | FGSM 0.5 | A (0.43) | B (0.47) | B (0.41) |
| Square 1 | FGSM 1 | A, borderline (0.29) | C (0.25) | B (0.66) |
| Square 1 | PGD 1 | C (0.25) | C (0.21) | A (0.75) |
| APGD-CE 4 | all four | F | F | F |
| Square 4 | clean | C (0.03) | C (0.00) | F (0.01) |
| Square 4 | FGSM 0.5 | C (0.11) | C (0.19) | F (0.25) |
| Square 4 | FGSM 1 | C (0.10) | B, borderline (0.29) | F (0.52) |
| Square 4 | PGD 1 | A (0.42) | A (0.66) | F (0.63) |

Counts (A / B / C / F): CIFAR-10 4 / 0 / 8 / 4 · CIFAR-100 2 / 3 / 7 / 4 · ImageNet16-120 1 / 2 / 1 / 12. All 48 outcomes are unchanged when all 6,466 architectures are used.

**Observations.** These are descriptive. Plan §9.4 calls a conclusion general only if the same A, B or C repeats on all three datasets, and **one pair** meets that condition: clean accuracy → Square ε = 1 (C on all three).

- **Clean accuracy** gets C in all 7 pairs where a selection verdict is possible. Under APGD-CE ε = 1, its top 5% has a lower mean robust accuracy than a random top 5% on all three datasets.
- **PGD ε = 1** gets A in 5 of its 7 assessable pairs. The exceptions are Square ε = 1 on CIFAR-10 and CIFAR-100.
- **FGSM** outcomes depend on the dataset.
- **APGD-CE ε = 4** is floor-limited on all three datasets. The number of floor-limited pairs is 4, 4 and 12.
- **Floor effect** (criterion of plan §6, applied to each floor-limited pair): supported in 2 pairs, not supported in 10, not assessable in 8.
- **Clean-accuracy cost.** The mean clean accuracy of the top 5% selected by PGD is 83.6% / 46.9% / 17.4%, against 93.8% / 71.3% / 45.4% when selected by clean accuracy.

Exploratory checks, not in the plan:

- For APGD-CE ε = 4 on CIFAR-10, exact accuracies from the confusion matrices reproduce the τ values of Fig. 3, while the stored 3-decimal values do not (CIFAR-10 notebook, cell 19).
- Colour readings of Fig. 23 (CIFAR-100) agree with our τ within about 0.01 for 12 of 16 values.
- On ImageNet16-120 (Fig. 24) the readings differ by 0.01–0.06. This is consistent with the storage precision of small accuracies, but it has not been tested.

## Timeline and transparency

- The analysis plan (v1.6) was fixed on 19 November 2026.
- Question 0 of the plan (the proxy gap, a descriptive computation fully specified in the plan) was computed on the evening of 19 November, shortly before the plan was committed (20:34 CET, commit `307f4dd`). The plan text was not changed afterwards.
- All agreement analyses (τ, ρ, overlap@k, bootstrap, verdicts; steps A3–A6) were run after that commit.
- The plan was therefore pre-specified before the main analyses, but its public timestamp is later than Q0. The commit message of `307f4dd` ("before results") should be read in that sense.

## Limitations

- One search space (NAS-Bench-201), standard-trained networks only, and one trained network per architecture. Seed and test-set uncertainty cannot be estimated.
- On ImageNet16-120, 3 of the 4 targets are floor-limited, so the selection question can be assessed there only for Square ε = 1.
- Accuracies are stored with 3 decimals. This affects ties and near-zero values, most of all on ImageNet16-120.
- The exclusion rule removes only fully degenerate networks. Near-degenerate networks (clean accuracy around 10–17% on ImageNet16-120) remain and can rank high under attack.
- Two verdicts are borderline: lower bounds of 0.258 and 0.252 against the threshold of 0.25.

## Repository

- `docs/analysis_plan.md`: the analysis plan, v1.6.
- `notebooks/P1_robustness_ranking.ipynb`: CIFAR-10 development notebook, frozen. Cell 19 is exploratory.
- `notebooks/P1_robustness_ranking_cifar100.ipynb` and `notebooks/P1_robustness_ranking_ImageNet16_120.ipynb`: confirmation runs. Cells 20–22 (download, integrity check, side-by-side verdicts) are outside the frozen analysis.
- `results/`: for each dataset tag (`cifar10`, `cifar100`, `ImageNet16-120`, each also with `_all` for the sensitivity analysis):
  - `a4_point_*`: point estimates.
  - `a4_bootstrap_*`: bootstrap resamples.
  - `a4_summary_*`: estimates with intervals.
  - `a5_verdicts_*`: verdicts.
  - `a5_overlap_at_k_*.png`: overlap@k against k.

**To reproduce:** run a notebook in Google Colab after downloading the accuracy files (see "Data and credit"). For the sensitivity analysis, run cell 18 before cells 11–17.

## Data and credit

- Dataset: S. Jung, J. Lukasik, M. Keuper, *Neural Architecture Design and Robustness: A Dataset*, ICLR 2023. http://data.robustness.vision/
- Helper code: https://github.com/steffen-jung/robustness-dataset
- The data are not copied into this repository.
- Related work: X. Ning et al., *Evaluating Efficient Performance Estimators of Neural Architectures*, NeurIPS 2021. Y. Feng et al., *Zero-Cost Proxy for Adversarial Robustness Evaluation*, ICLR 2025.

## Author

Mahran Rezk
