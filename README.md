# BAPGT

**Balance-Aware Prototypical Graph Transformer for Signed Bipartite Link-Sign Prediction**

This repository accompanies the manuscript *Balance-Aware Prototypical Graph Transformer for Signed Bipartite Link-Sign Prediction*.

BAPGT predicts the polarity of a candidate interaction in a signed bipartite graph. It represents observed positive and negative interactions as edge tokens and reasons over a bounded structural context retrieved for each query. The model is evaluated on rating, check-in, and buyer–seller feedback datasets.

## Overview

For a candidate link, BAPGT creates an **unknown-sign query token** and retrieves relevant observed interactions without constructing a complete line graph. It combines:

- **Balanced-path bias (BPB):** polarity evidence from odd-length signed paths between the query endpoints.
- **Butterfly-aware bias (BAB):** compatibility evidence from signed bipartite 4-cycles among context interactions.
- **Sparse Graph Transformer:** query-conditioned attention guided by the two structural biases.
- **Multi-prototype sign decoder (PSD):** multiple learnable prototypes for each sign class.
- **Balanced-path consistency (BPC):** prediction-level structural supervision filtered by the strength and agreement of path evidence.

The query edge and its ground-truth sign are excluded from query-conditioned input construction. For training queries, the query edge is removed from that query's context; validation and test queries use the observed training graph only.

## Datasets

The following statistics describe the **preprocessed graphs used in the manuscript**. Original dataset downloads may have different sizes.

| Dataset | Left nodes | Right nodes | Interactions | Positive ratio |
| --- | ---: | ---: | ---: | ---: |
| MovieLens-1M | 6,040 | 3,952 | 1,000,209 | 0.575 |
| Amazon-Book | 35,736 | 38,121 | 1,960,674 | 0.806 |
| Gowalla | 37,000 | 11,500 | 3,500,000 | 0.612 |
| ML-10M | 69,878 | 10,677 | 10,000,054 | 0.589 |
| Bonanza | 7,919 | 1,973 | 36,543 | 0.980 |

For MovieLens-1M, Amazon-Book, and ML-10M, ratings **≥ 4** are positive and ratings **< 4** are negative. This includes half-star ratings in ML-10M: a rating of 3.5 is negative. Bonanza uses its original feedback signs. In Gowalla, observed check-ins are positive; negatives are sampled from user–location pairs absent from the complete raw interaction log. These sampled pairs are **unobserved interactions, not explicit negative feedback**. Original data sources and further preprocessing details are given in Section 4.1.1 of the manuscript.

## Environment and default settings

The experiments in the manuscript used **PyTorch 2.3.2** and one **NVIDIA A100 GPU**.

| Hyperparameter | Value |
| --- | ---: |
| Optimizer | Adam |
| Learning rate | 0.005 |
| Weight decay | 0.00001 |
| Embedding dimension | 32 |
| Batch size | 2,048 |
| Transformer layers / attention heads | 3 / 4 |
| Prototypes per sign class | 4 |
| Maximum signed-path length | 5 |
| BPC loss weight | 0.1 |
| Early-stopping patience | 20 epochs |
| Default incident-edge / context-token budgets | 64 / 256 |
| Default retained-path / sampled-butterfly budgets | 64 / 64 |

Other tuned settings are described in Appendix A of the accompanying supplementary material.

## Reproducing the experiments

1. Apply the sign-construction rules above to the original datasets.
2. Create five independently generated, matched **85% / 5% / 10%** training, validation, and test splits.
3. Build the observed signed graph from training interactions only. Exclude each training query from its own retrieved context.
4. Train on each split and select the checkpoint by validation AUC, with a patience of 20 epochs.
5. Evaluate on the held-out test set using **AUC, Binary-F1, Macro-F1, and Micro-F1** and aggregate results over the five splits.

When measuring efficiency, construct test-query contexts once, then reuse them for the cached forward pass. End-to-end inference includes both context construction and the forward pass; the training time per epoch reported in the manuscript excludes one-time context construction.

**Repository-specific installation and execution commands:** add the exact commands after uploading the implementation and its dependency file. This README does not assume unverified script names or command-line options.

## Main results

BAPGT achieves the following mean results in Table 2 of the manuscript:

| Dataset | AUC | Macro-F1 |
| --- | ---: | ---: |
| MovieLens-1M | 0.811 | 0.769 |
| Amazon-Book | 0.774 | 0.724 |
| Gowalla | 0.799 | 0.774 |
| ML-10M | 0.844 | 0.804 |
| Bonanza | 0.687 | 0.668 |

## Citation

Please cite the manuscript if you use this work. Replace this entry with the final publication details when available.

```bibtex
@unpublished{wang2026bapgt,
  title  = {Balance-Aware Prototypical Graph Transformer for Signed Bipartite Link-Sign Prediction},
  author = {Wang, Xiaosong and Wang, Qingyong and Gu, Lichuan},
  year   = {2026},
  note   = {Manuscript}
}
```
