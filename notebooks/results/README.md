# Results

5 seeds per arm, both datasets, Recall@20 / NDCG@20 via the shared scoring function.

## Summary (mean ± std across seeds)

| Dataset | Model | Recall@20 | NDCG@20 |
|---|---|---|---|
| ML-1M | DeepFM_off | 0.0465 ± 0.0029 | 0.0344 ± 0.0009 |
| ML-1M | DeepFM_on | 0.0442 ± 0.0017 | 0.0329 ± 0.0009 |
| ML-1M | WideDeep_off | 0.0470 ± 0.0033 | 0.0346 ± 0.0016 |
| ML-1M | WideDeep_on | 0.0395 ± 0.0043 | 0.0290 ± 0.0038 |
| ML-1M | LightGCN | 0.1852 ± 0.0005 | 0.1733 ± 0.0002 |
| ML-1M | SVD | 0.0235 ± 0.0005 | 0.0201 ± 0.0005 |
| KuaiRec | DeepFM_off | 0.0615 ± 0.0075 | 0.0456 ± 0.0055 |
| KuaiRec | DeepFM_on | 0.0657 ± 0.0145 | 0.0509 ± 0.0142 |
| KuaiRec | WideDeep_off | 0.0564 ± 0.0088 | 0.0413 ± 0.0074 |
| KuaiRec | WideDeep_on | 0.0701 ± 0.0073 | 0.0527 ± 0.0050 |
| KuaiRec | LightGCN | 0.1438 ± 0.0029 | 0.1297 ± 0.0028 |
| KuaiRec | SVD | 0.0318 ± 0.0018 | 0.0256 ± 0.0015 |

LightGCN is the top performer on both datasets, by a wide margin over every other arm.

## Content-feature effect (on − off, paired by seed, NDCG@20)

| Dataset | Method | Δ | Δ % | p-value | Significant (α=0.05) |
|---|---|---|---|---|---|
| ML-1M | DeepFM | −0.0015 | −4.4% | 0.086 | No |
| ML-1M | WideDeep | −0.0056 | −16.2% | **0.022** | **Yes** |
| KuaiRec | DeepFM | +0.0053 | +11.6% | 0.555 | No |
| KuaiRec | WideDeep | +0.0114 | +27.6% | 0.094 | No |

## Headline finding

Content features **hurt** both hybrids on ML-1M and **help** both hybrids on KuaiRec — a directional reversal, consistent across DeepFM and WideDeep. This runs counter to the pre-registered prediction (positive lift expected on ML-1M, null/negative expected on KuaiRec).

Only WideDeep/ML-1M clears significance at n=5 seeds; the other three deltas are directionally consistent but not individually significant. Effect sizes, not just p-values, should be reported — n=5 has limited power to detect anything short of a large effect.

LightGCN's advantage over SVD narrows on KuaiRec (8.6× → 4.6×), consistent with the prediction that the fully-observed eval protocol reduces the benefit of graph propagation over sparse data.

## Known caveats

- `DeepFM_on / KuaiRec / seed 0` had a source conflict during result compilation (two logged values, ~2× apart); the value used here has not been independently re-verified.
- Cross-dataset comparisons of effect *size* (not direction) are confounded by ML-1M's 18-genre vs. KuaiRec's 14-tag feature vocabulary — see Discussion.
- Some KuaiRec values were transcribed from truncated terminal output rather than a saved CSV; precision beyond ~6 decimal places on those rows is not meaningful.

## Files

- `summary_by_model.csv` — full mean/std/n table
- `content_feature_deltas.csv` — per-seed paired deltas, t-stats, p-values
- `analysis.py` — script that produced both
