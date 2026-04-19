---
title: "What is a signal worth taking"
prereqs: "Returns; The drunkard, the photon, the stock"
arrives_at: "why financial ML cannot use iid labels"
code_ref: "—"
---

# What is a signal worth taking

*What is a lesson worth learning? What is a signal worth taking? We must choose — before we measure. The ML texts do not tell us this. They assume we already know.*

Supervised ML requires labels. What label should be assigned to observations in a financial time series?

The textbook answer — predict tomorrow's return, or predict its sign — fails in specific ways once applied. This lesson explains why naive labeling strategies fall short and introduces the labeling framework AFML uses.

## The naive approach — fixed-horizon returns

Given a daily return series $r_t$, assign each day a binary label: $y_t = 1$ if $r_{t+1} > 0$, else $y_t = 0$. Train a classifier to predict $y_t$ from features available at $t$.

*The "predict tomorrow's close" framing of a thousand tutorials. Several problems follow.*

### The label does not correspond to a decision

A trader's decision is not "will tomorrow close up?" It is: if this position is taken now, should it be closed at a profit target, a stop, or after a time period? The decision depends on the *path* between entry and exit, not only on the terminal price.

Fixed-horizon labels ignore path dependence. A label of $y = 1$ on a day with a 0.3% close-to-close gain is treated identically to $y = 1$ on a day with a 5% intraday peak that reverses before close. The scenarios produce materially different trading outcomes.

### Magnitude is discarded

Reducing a continuous return to $\pm 1$ discards information. A strategy trading on the signal does not treat a 0.01% move and a 5% move equivalently.

### Labels are noisy near zero

In flat markets, small returns flip sign randomly. A day with $r_t = +0.001\%$ receives label 1; a day with $r_t = -0.001\%$ receives label 0. These represent the same market condition. The classifier learns to produce confident predictions on days with no real signal.

### The iid assumption fails

Scikit-learn's default cross-validation shuffles the dataset. For time series, this produces the k-fold leakage covered in [the walk-forward lesson](../backtest/walk-forward.md).

## A partial improvement — continuous return labels

Use the continuous return $y_t = r_{t+1}$ as a regression target. Magnitude and near-zero flip are addressed. Remaining issues:

- Regression on raw returns has very low SNR.
- Path dependence remains unaddressed.
- MSE loss weights outlier days disproportionately.

## The AFML reframe — label events, not bars

*Rather than predict the return at every bar, predict the outcome of specific events where a trading decision would actually be made.*

An event is a moment at which a primary signal fires — an RSI cross, a volatility spike, a news release, a calendar trigger. The labeling question becomes: given the primary signal at time $t$, what would the outcome have been had the trade been taken?

The outcome is determined by a specific exit rule. The canonical choice is the **triple barrier**, defining three exit conditions:

- **Upper barrier** (profit-take): a move of $+\text{pt\_mult} \times \sigma$ above entry.
- **Lower barrier** (stop-loss): a move of $-\text{sl\_mult} \times \sigma$ below entry.
- **Vertical barrier** (time stop): a fixed maximum holding period.

The first barrier hit determines the label: upper yields $+1$, lower yields $-1$, vertical yields $\text{sign}(r_\text{final})$.

## How triple-barrier addresses the failure modes

**Labels correspond to decisions.** The profit-take and stop-loss levels are actions the strategy could have executed. The time barrier reflects the maximum holding period.

**Path dependence is preserved.** The label depends on the entire path between entry and exit.

**Vol scaling makes labels comparable.** Barriers are specified in units of rolling volatility. A 2σ move in 2017 and a 2σ move in 2020 receive the same label, even though their dollar magnitudes may differ by 5×.

**Events are sparse.** A typical strategy produces hundreds to thousands of events per year rather than thousands of bars.

## New problems introduced by triple-barrier

*Every fix creates new problems.*

### Label overlap

If events are close together in time, their label horizons overlap. An event at $t$ has a label depending on prices through $t + 5$. An event at $t + 2$ has a label depending on prices through $t + 7$. The two overlap by three days.

Labels are not iid; they share underlying price realizations. This breaks naive cross-validation. [Purging](../backtest/purging-embargo.md) is the correction.

### Sample non-uniqueness

Overlapping labels share causal observations. Treating them as iid in training over-weights the overlapping regions. Sample weighting by uniqueness (AFML's sequential bootstrap, `packages/afml/src/afml/bootstrap.py`) corrects for this.

### Meta-labels for primary-secondary decomposition

A triple-barrier label describes whether a trade would have been profitable. The operational question is whether to take the trade. Meta-labeling separates the two: a primary model emits direction, a secondary model predicts whether the primary's direction would have been profitable. Covered in the meta-labeling lesson.

## Features for the model

Labeling is half the problem; feature selection is the other half.

- **Stationarity.** Raw price is non-stationary. Features should be stationary or approximately so.
- **Fractional differentiation.** AFML's fracdiff preserves memory while achieving stationarity (`packages/afml/src/afml/fracdiff.py`).
- **Information available at $t$, not $t + 1$.** Features must be computable from data available at $t$.
- **Features for the secondary.** Meta-labeling features include regime tags, trailing primary hit rates, vol regime, calendar effects.

## Summary

- "Predict tomorrow's close" is an inappropriate framing for a tradeable ML model: it ignores decision structure, path dependence, and has near-zero signal-to-noise.
- Labeling events rather than bars aligns ML with the trading decisions that matter.
- Triple-barrier introduces new problems (label overlap, sample non-uniqueness) addressed by the rest of the AFML toolkit (purging, sequential bootstrap, meta-labeling).

## Implemented at

The labeling problem motivates the AFML package:

- `trading/packages/afml/src/afml/labeling.py` — `apply_triple_barrier`, `meta_label`, `rolling_vol`.
- `trading/packages/afml/src/afml/cv.py` — `PurgedKFold`.
- `trading/packages/afml/src/afml/bootstrap.py` — `sequential_bootstrap`, `average_uniqueness`.
- `trading/packages/afml/src/afml/fracdiff.py` — `get_weights_ffd`, `frac_diff_ffd`.
- `trading/packages/afml/src/afml/meta.py` — `MetaLabeler`.

*Each of these exists because the naive labeling approach does not work.*

---

*What is worth learning? We must choose before we measure. Next: three gates.*

**Next:** [Three gates →](triple-barrier.md)
