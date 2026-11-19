# Analysis plan — P1: ranking agreement between cheap and expensive robustness evaluations

**Author:** Mahran Rezk · **Version:** 1.6 · **Plan fixed:** 19 November 2026 · **Status:** pre-results

**What had been seen when this plan was fixed:** the data were loaded and checked, and the marginal distribution (count, mean, standard deviation, quantiles) of seven columns on CIFAR-10 was inspected: clean, FGSM ε = 0.5, FGSM ε = 1, APGD-CE ε = 1, Square ε = 1, APGD-CE ε = 4 and Square ε = 4. PGD (ε = 1, C4) was added in v1.4 after reading the paper; its distribution had not been inspected, and only one PGD value had been seen (the loading check for architecture 13433). The maxima reported in Table 1 of the paper had also been read. **No agreement metric** (τ, ρ, overlap, regret, floor share, conditional τ) had been computed. The changes in v1.2 to v1.6 (§12) were informed by that descriptive view and by the count of degenerate architectures (§3), and are declared here.

Any change after this date is listed in the change log (§12) with its reason.

## 1. Questions

**Relation to Jung et al. (2023), checked against the paper itself.** The paper already shows global rank agreement between clean and adversarial accuracies for all attacks and ε (Kendall τ, Figure 3; other datasets in Figures 23–24), rank agreement across image datasets (Figure 17), and a NAS experiment that searches on clean or FGSM (ε = 1) accuracy with 300 queries and reports the accuracy of the found architectures under the other attacks (Table 1). It does not report top-k selection quality, regret relative to the optimum, the role of near-zero accuracies, or how much of the remaining robustness gap is caused by the cheap objective itself. Our τ and ρ therefore **reproduce** known results and serve as a check; the contribution we aim at is in questions 0, 2 and 3.

0. **Where does the gap in Table 1 come from? (headline)** In Table 1, searching on FGSM (ε = 1) reaches almost the FGSM optimum (Local Search: 69.10% vs 69.24%), yet the APGD-CE (ε = 1) accuracy of the found architectures stays far below the APGD-CE optimum (23.18% vs 54.02%). With all 6,466 architectures available, we report a **descriptive gap decomposition** into a **proxy gap** (even a perfect search on FGSM over the full space ends here) and a **search gap** (what a 300-query search loses on top of that). The same split is reported for the other cheap evaluations.
1. **Global ranking:** does a cheap evaluation preserve the ranking of all architectures under a stronger or more expensive evaluation?
2. **Selection and cost:** does it find the best architectures (top-k), which is what matters for architecture search, and how much attack cost is needed for that? The cheap evaluations form a cost ladder: no attack (clean), one gradient step (FGSM), 40 gradient steps (PGD).
3. **Explanation:** where global agreement is weak, is this explained by a floor effect (most architectures at near-zero robust accuracy), or is the ranking unstable also among architectures that keep some robustness? Feng et al. (ICLR 2025) report vanishing agreement on this dataset under strong attacks and suggest near-zero accuracies as the cause; we test this rather than assume it.

We measure **ranking agreement** and **ranking transferability**. We do not build or test a predictive model, so we do not use the word "predict".

## 2. Data

- Robustness dataset of Jung, Lukasik and Keuper (ICLR 2023): NAS-Bench-201 architectures evaluated under adversarial attacks and common corruptions. Data: http://data.robustness.vision/ (redirects to a Nextcloud share); helper code: https://github.com/steffen-jung/robustness-dataset
- 6,466 non-isomorphic architectures, i.e. **the complete non-isomorphic NAS-Bench-201 space**, not a sample. One trained network per architecture and dataset.
- Epsilon values in `meta.json` (checked 19 Nov 2026, unit 1/255): APGD-CE, Square, PGD: 0.1, 0.5, 1, 2, 3, 4, 8; FGSM: 0.1, 0.5, 1, 2, 3, 4, 5, 6, 7, 8, 255.
- Loading check passed: architecture 13433 on CIFAR-10 gives clean accuracy 0.893 and PGD (ε = 1) accuracy 0.336, as in the helper's README.
- Accuracies are stored with three decimals, so ties are frequent (§4).
- Cross-check with the paper: on CIFAR-10 the maxima of our columns match the "Optimum" row of Table 1 for FGSM (0.692 vs 69.24%), APGD-CE (0.540 vs 54.02%) and Square (0.736 vs 73.61%), all at ε = 1. For clean accuracy we see 0.946 vs 94.68% in Table 1; this small difference is checked in session 2 and reported.
- Only accuracy files are used.

## 3. Cheap evaluations (C), targets (T) and the architecture set

| ID | Measure | Reason |
|---|---|---|
| C1 | clean accuracy | no adversarial attack required; precomputed clean test accuracy is in the dataset |
| C2 | FGSM, ε = 0.5 | cheapest attack |
| C3 | FGSM, ε = 1 | setting used for NAS in Jung et al. (Table 1) |
| C4 | PGD, ε = 1 | intermediate cost; in this dataset PGD is the iterative least-likely-class attack (targeted, 40 steps), so it is weaker than untargeted PGD |
| T1 | APGD-CE, ε = 1 | strong white-box attack, same ε |
| T2 | Square, ε = 1 | black-box, score-based attack that does not rely on gradients and can help diagnose gradient masking |
| T3 | APGD-CE, ε = 4 | stronger white-box attack |
| T4 | Square, ε = 4 | stronger black-box attack |

4 × 4 = 16 pairs per dataset, 48 in total. All pairs are reported in one table.

**Nominal attack-budget ladder** (budgets per image, from the paper): clean 0; FGSM 1 gradient step; PGD 40 gradient steps; APGD-CE 100 iterations; Square up to 5,000 queries (forward passes only). The attacks differ in method, target and step rule, not only in the number of steps, so this is not a controlled cost curve or a run-time comparison.

**Descriptive note (v1.2):** on CIFAR-10, Square at ε = 1 lowers mean accuracy less than FGSM at ε = 1 (0.626 vs 0.494). For T2, "expensive" (5,000 queries) does not mean "strong". T2 is kept as specified. The targets differ in strength, and computational cost is not assumed to equal attack strength; the question is whether a cheap evaluation preserves the ranking under targets of different strength. APGD-CE at ε = 4 has median and upper quartile 0.000, so T3 is expected to be floor-limited.

**Degenerate architectures (v1.2).** Architectures whose clean accuracy is below twice the chance level (CIFAR-10: < 0.20; CIFAR-100: < 0.02; ImageNet16-120: < 2/120 ≈ 0.0167) are excluded from the main analysis. Reason: such networks would never be selected in architecture search, and a network that predicts (almost) the same class for every input keeps about chance-level accuracy under any attack, so it can look "robust" when it is only useless. The number excluded is reported per dataset (CIFAR-10: 5 architectures, all with clean accuracy 0.100), and the full analysis is repeated on all 6,466 architectures as a sensitivity analysis. Exclusion depends on clean accuracy only; architectures that are weak under attack are not excluded.

## 4. Metrics

Let C_i and T_i be the accuracy of architecture i under the cheap evaluation and under the target.

| Metric | Definition |
|---|---|
| Kendall τ-b | rank agreement over all architectures in the analysis set (same coefficient as Jung et al.; corrects for ties) |
| Spearman ρ | rank correlation (for comparison with CRoZe) |
| overlap@k | share of the top-k% architectures by C that are also in the top-k% by T; k = 1%, 5%, 10% of the analysis set |
| top-1 regret | Regret = max_i T_i − T_j, where j = argmax_i C_i; in percentage points. If several architectures share the maximum of C, T_j is their mean |
| top-5% mean target accuracy | mean T of the top-5% selected by C (tie-aware weights, below) and of the top-5% by T itself; absolute values in % |
| top-5% mean clean accuracy | mean clean accuracy of the top-5% selected by C, to show the trade-off with clean accuracy |
| proxy gap (T1 and T2, ε = 1) | max_i T_i − T_j with j = argmax_i C_i, computed on **all 6,466 architectures without exclusion**, the same space as Table 1. If several architectures share the maximum of C, T_j is their mean target accuracy (same rule as top-1 regret); for C3 this is the gap that remains even for a search that finds the FGSM optimum exactly |
| search gap (T1 and T2, ε = 1) | T_j (above) minus the published mean APGD-CE or Square accuracy of the architectures found by each NAS method in Table 1 of Jung et al. (C1 and C3 only, since Table 1 searched on clean and FGSM). **Descriptive:** it compares one architecture with a published mean over 100 runs, so it can be negative (a search may find architectures close to the FGSM optimum that are better under APGD-CE) and it is not a pure measure of search efficiency |
| recorded-zero share | share of architectures whose recorded target accuracy is 0.000; with three decimals this includes small values rounded to zero |
| Q0 identities | for Q0, the IDs of the cheap-best and the target-best architectures, and the rank of each under the other evaluation (descriptive) |
| floor share | share of architectures with target accuracy below 1% |
| conditional τ-b | Kendall τ-b among architectures with target accuracy ≥ 1% (diagnostic only, see §6) |

Top-1 regret and the proxy gap are based on a single maximum; bootstrap intervals are unreliable for such statistics, so they are reported as point values. The top-5% quantities carry the bootstrap intervals.

**Ties in top-k selection (v1.3, tie-aware).** Each architecture gets a membership weight in the top-k of a ranking: 1 if its value is strictly above the k-th value, r/g if it belongs to the group of g architectures tied at the k-th value with r places left, and 0 otherwise. Then

overlap@k = (Σ_i w_i^C · w_i^T) / k,

which equals the **expected overlap under uniform random tie-breaking**, computed exactly and without random draws. The same formula is used for the point estimate and inside every bootstrap resample, so ties are treated identically in both, and the result does not depend on the order of architecture IDs. (Checked against 4,000 random tie-breaking draws on simulated tied data: identical to four decimals.) The number of architectures tied at the k-th position is reported for every pair.

## 5. Uncertainty

| Source | Measurable with these data? | What is needed for an exact estimate | How it is reported |
|---|---|---|---|
| (a) architecture composition | yes | available | bootstrap intervals (below) |
| (b) test-set sampling | no | per-image correctness for every architecture and attack; the accuracy files contain one value per architecture, and the confidence and confusion-matrix files appear to be per class, not per image | not estimated; stated as a limitation |
| (c) training seed | no | several trained networks per architecture | not estimated; stated as a limitation |

**Meaning of the bootstrap (v1.2).** Because the analysis uses the complete non-isomorphic space, the point estimates are the exact values for NAS-Bench-201 under these trained networks and this test set. The bootstrap (B = 1000 resamples of architectures with replacement, seed 0, 95% percentile intervals) therefore does **not** measure uncertainty about NAS-Bench-201 itself. It measures how sensitive a result is to the composition of the architecture set, i.e. whether it is carried by many architectures or by a few. The main remaining uncertainties, (b) and (c), cannot be estimated here. Conclusions apply to NAS-Bench-201 and do not transfer automatically to other search spaces.

## 6. Floor diagnostics

- A target is **floor-limited** if its floor share is above 50%.
- **Conditional τ-b** is computed only if at least 100 architectures have target accuracy ≥ 1%; otherwise it is reported as "not assessable". Selecting on the target restricts the range, which usually lowers correlation; an increase relative to global τ is therefore informative, a decrease is not.
- **Floor effect is called "supported"** only if the target is floor-limited **and** the conditional τ-b is assessable **and** the lower bound of its 95% interval is ≥ 0.5. Weak global τ alone is never called a floor effect.
- **Floor-limited targets get no selection verdict (v1.3).** When most architectures are near zero, ranking them separates tiny differences around zero, often within rounding and ties. For these targets, τ, overlap@k, regret and the top-5% mean target accuracy are reported **descriptively**, together with the floor diagnostics above. We do not conclude from them that a cheap evaluation can select architectures with useful robustness.

## 7. Decision rules

Selection verdict for each (C, T) pair, based on the **lower bound** of the 95% bootstrap interval of overlap@5%:

| Verdict | Lower bound of overlap@5% |
|---|---|
| PASS | ≥ 0.50 |
| WARN | 0.25 to < 0.50 |
| FAIL | < 0.25 |

"Global τ is good" means: lower bound of the 95% interval of Kendall τ-b ≥ 0.5.

| Outcome | Condition | Interpretation |
|---|---|---|
| F — floor-limited target | floor share > 50% | descriptive only, no selection verdict (§6); floor effect reported as supported, not supported or not assessable |
| A — agreement | not floor-limited, overlap@5% PASS or WARN, global τ good | the cheap evaluation preserves both the global ranking and the top |
| B — top-k preserved despite weak global agreement | not floor-limited, overlap@5% PASS or WARN, global τ not good | the top can still be selected, although the global ranking is not preserved |
| C — top-k not preserved | not floor-limited, overlap@5% FAIL | the cheap evaluation is not adequate for selecting the best architectures |

F is decided first; A, B and C then cover all remaining cases. The top-5% mean target accuracy is reported for every pair, so that a verdict is always read together with the absolute accuracy it refers to. Results for k = 1% and 10% are reported as sensitivity; the verdict uses k = 5% only.

## 8. Baselines

Descriptive reference points, not significance tests:

- **Random ranking:** the cheap ranking is replaced by an independent random ranking, 1000 permutations (seed 0), and compared with the target ranking; expected τ ≈ 0 and overlap@k ≈ k.
- **Clean accuracy (C1):** FGSM (C2, C3) adds value only if it beats C1 for the same target. Reported as the paired bootstrap difference (same resample for both).

## 9. Development and confirmation

1. Code is developed and results are inspected on **CIFAR-10 only**.
2. The notebook is committed to this repository before any other dataset is analysed; the commit hash is recorded.
3. The same code runs **unchanged** on CIFAR-100 and ImageNet16-120. Changes after freezing are bug fixes only and are logged.
4. If datasets disagree, the disagreement is reported as a finding. A conclusion is called general only if its outcome (A, B or C) repeats on all three datasets.

## 10. What this analysis does not do

- No p-values or significance tests: this is estimation, not hypothesis testing.
- No choice of ε, k, thresholds or exclusions after seeing agreement results.
- No estimate of test-set or training-seed uncertainty.
- No conclusions for adversarially trained networks (the networks in this dataset are standard-trained) or for other search spaces.
- No claim of novelty: the literature check (8 works, 2023–2026) was not systematic.
- Rounding: accuracies are stored with three decimals, which affects values near zero (recorded-zero share), the number of ties and the floor diagnostics.

## 11. Planned outputs

One table with all pairs (point estimate, interval, verdict, outcome, ties at the boundary, number of excluded degenerate architectures), the same table for the sensitivity analysis on all 6,466 architectures, one figure of overlap@k against k, and a limitations section.

## 12. Change log

| Date | Version | Change | Reason |
|---|---|---|---|
| 19 Nov 2026 | 1.1 | Plan drafted | — |
| 19 Nov 2026 | 1.2 | Bootstrap described as architecture-composition sensitivity, not uncertainty about the space | the analysis uses the complete space (external review) |
| 19 Nov 2026 | 1.2 | Outcome B renamed; "floor effect" only with evidence (zero share, floor share, conditional τ-b with n ≥ 100) | weak τ alone does not show a floor effect (external review) |
| 19 Nov 2026 | 1.2 | Random tie-breaking for top-k; ties at the boundary reported | results must not depend on ID order (external review); 3-decimal accuracies produce many ties |
| 19 Nov 2026 | 1.2 | Test-set noise simulation removed | not recoverable from aggregate accuracies (external review) |
| 19 Nov 2026 | 1.2 | Exclusion of degenerate (near-chance) architectures, with a sensitivity analysis on all architectures | descriptive view: minimum clean accuracy 0.100 on CIFAR-10, and APGD-CE ε = 4 maximum 0.106, close to chance |
| 19 Nov 2026 | 1.2 | Regret formula, wording for C1 and the random baseline, three-question framing | clarity (external review) |
| 19 Nov 2026 | 1.3 | Exact tie-aware overlap (expected overlap under random tie-breaking), used identically in point estimates and bootstrap; tie rule for regret | ties were treated differently in point estimate and bootstrap in v1.2 (external review) |
| 19 Nov 2026 | 1.3 | Floor-limited targets: outcome F, descriptive only, no selection verdict; top-5% mean target accuracy added for all pairs | overlap around zero is not evidence of selecting useful robustness (external review) |
| 19 Nov 2026 | 1.3 | Count of excluded degenerate architectures on CIFAR-10 (5, all at clean accuracy 0.100); cost not equated with strength | descriptive count; clarity |
| 19 Nov 2026 | 1.4 | Relation to Jung et al. stated; τ and ρ marked as reproduction; headline question 0 (proxy gap vs search gap, using Table 1) | reading the paper showed that global τ, cross-dataset τ and FGSM-based NAS are already covered |
| 19 Nov 2026 | 1.4 | C4 = PGD (ε = 1) added as an intermediate-cost evaluation; nominal attack budgets reported | cost ladder for the selection question |
| 19 Nov 2026 | 1.4 | Mean clean accuracy of the selected top-5%; no bootstrap for max-based statistics; cross-check of maxima against Table 1 | trade-off with clean accuracy; correct use of the bootstrap; data check |
| 19 Nov 2026 | 1.5 | Proxy gap on the full space without exclusion; search gap described as descriptive and possibly negative; "nominal attack-budget ladder"; metrics table formatting fixed | comparability with Table 1 and precise wording (external review) |
| 19 Nov 2026 | 1.6 | Tie rule for the proxy gap; proxy and search gap also for Square (T2, reported in Table 1); Q0 identities and ranks; "recorded-zero share"; Square wording; list of inspected columns; rounding limitation | precise definitions and transparent record of what was seen (external review) |

## References

- S. Jung, J. Lukasik, M. Keuper. *Neural Architecture Design and Robustness: A Dataset.* ICLR 2023.
- J. Lukasik, M. Moeller, M. Keuper. *An Evaluation of Zero-Cost Proxies – from Neural Architecture Performance to Model Robustness.* IJCV 2024.
- Y. Feng, Y. Ou, J. Fan, Y. Sun. *Zero-Cost Proxy for Adversarial Robustness Evaluation.* ICLR 2025.
- H. Ha et al. *Generalizable Lightweight Proxy for Robust NAS against Diverse Perturbations (CRoZe).* NeurIPS 2023.
