---
title: "What a label reaches for"
prereqs: "You cannot stand in tomorrow"
arrives_at: "leak-free CV on overlapping labels"
code_ref: "trading/packages/afml/src/afml/cv.py — PurgedKFold"
---

# What a label reaches for

*A label reaches forward. It does not mean to. It was written from a future it could not help containing. We must teach our models not to see.*

Walk-forward ensures test data follows training data in wall-clock time. This is necessary but not sufficient. A more subtle form of leakage occurs within a single fold and silently inflates k-fold results on overlapping labels.

This lesson covers the López de Prado AFML ch. 7 corrections: purging and embargo.

## The problem — label horizons

Consider a triple-barrier label. A signal at time $t$ receives a label based on what happens between $t$ and $t + 5$ (or until the first barrier is hit). The label at $t$ is a function of prices up to $t + 5$.

Now consider k-fold CV. Suppose the test fold contains observation $t_\text{test}$ and the train fold contains observation $t_\text{train}$ with $t_\text{train} < t_\text{test} < t_\text{train} + 5$. The training observation's label was computed using prices through $t_\text{train} + 5$, which includes the test observation's window.

*The label reached forward. The model learned from a future it should not have seen.*

The result: the model performs better on the test fold than it would in live trading. The Sharpe is inflated. The bias is systematic. Naive k-fold on triple-barrier labels typically overstates Sharpe by 10-30% in standard setups.

## Purging

The AFML correction: drop training observations whose label horizons overlap the test fold.

For each observation $i$, define the label horizon $t_1[i]$ — the time at which its label is realized. In triple-barrier labeling, this is `exit_idx`, which may be upper barrier, lower barrier, or vertical. An observation $i$ in the train fold is purged if:

$$
[i, t_1[i]] \cap [\text{test fold boundaries}] \neq \emptyset.
$$

*Any train observation whose label horizon reaches into the test fold is dropped.*

Purging reduces training data by an amount that depends on label horizon and fold size. For vertical of 5 and fold sizes of 50, approximately 10% of train observations are purged.

## Embargo

Purging addresses overlap on the "before" side of the test fold. The "after" side involves serial correlation.

Even when no label horizon crosses the test-fold boundary, consecutive observations tend to be correlated. A training observation at $t = t_\text{test,end} + 1$ correlates with test observations through short-horizon serial dependence.

The correction: embargo a buffer of samples after the test fold, excluding them from training.

*Embargo size is typically 1% to 2% of total samples. On 5,000 daily observations, a 1% embargo removes 50 samples after each test fold.*

The embargo applies whether or not labels themselves overlap. It addresses serial-correlation leakage independently of purging.

## The combined procedure

The full AFML purged-k-fold with embargo:

1. Partition samples into $k$ equal-size contiguous folds (without shuffling).
2. For each test fold:
   a. Drop training observations whose label horizon overlaps the test window (purging).
   b. Drop training observations within a small buffer after the test fold (embargo).
   c. Train on what remains.
   d. Evaluate on the test fold.
3. Aggregate results.

*Contiguous, non-shuffled folds preserve time order within each fold. Shuffled folds would defeat the purge.*

## Purged k-fold versus walk-forward

Purged k-fold is a model-selection tool: hyperparameter tuning, feature selection, threshold calibration. It provides an out-of-sample estimate that does not leak through label horizons or serial correlation.

Walk-forward is a final-evaluation tool: how would this strategy have performed rolling forward through time?

The AFML protocol uses:

- **Inner loop (purged k-fold)** for hyperparameter selection on held-out purged folds within each walk-forward training window.
- **Outer loop (walk-forward)** for evaluating the selected model on strictly-future test windows.

This is the pattern in the trading project: `harness.backtest.WalkForward` outer, `afml.cv.PurgedKFold` inner.

## Embargo as a percentage

The embargo in `PurgedKFold` is specified as a fraction of sample size (`embargo_pct=0.01`) rather than an absolute count.

*A percentage scales cleanly. Serial correlation is roughly constant in calendar time. As sample rate grows, absolute embargo grows.*

## Cost of correctness

Purging has costs:

- Reduced training data. For horizon 5 and fold sizes 50, approximately 10%. For horizon 50 and fold sizes 100, approximately 50%.
- Less stable early folds. Purging disproportionately affects observations near fold boundaries.
- Increased computational cost. The purge mask is computed per fold.

*The question is not whether to afford the purge. It is whether to afford the bias that follows from skipping it.*

## Summary

- Naive k-fold on triple-barrier labels systematically overstates Sharpe because training labels computed from future prices leak across fold boundaries.
- Purging drops train samples whose label horizons overlap the test fold; embargo drops train samples in a buffer after the test fold to prevent serial-correlation leakage.
- Purged k-fold does not replace walk-forward; they solve different leakage problems and combine into a nested CV protocol.

## Implemented at

`trading/packages/afml/src/afml/cv.py:18` — `PurgedKFold(n_splits, t1, embargo_pct=0.01)` extends `sklearn.model_selection.BaseCrossValidator`. The `split()` method lines 39-75:

```python
train_mask = np.ones(n, dtype=bool)
train_mask[test_idx] = False
overlap = (indices <= test_end) & (self.t1 >= test_start)
train_mask &= ~overlap
if embargo > 0:
    embargo_mask = (indices > test_end) & (indices <= test_end + embargo)
    train_mask &= ~embargo_mask
yield indices[train_mask], test_idx
```

— the overlap line is the purge; the embargo block is the serial-correlation buffer. Because it subclasses `BaseCrossValidator`, it drops into any sklearn `GridSearchCV` or `cross_val_score` call.

---

*What a label reaches for, it must not see. Next: and if we tried too many times before we called it real.*

**Next:** [You tried a hundred →](deflated-sharpe.md)
