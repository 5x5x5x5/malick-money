---
title: "Two questions, once"
prereqs: "Three gates"
arrives_at: "direction vs sizing decomposition — the capstone strategy"
code_ref: "trading/packages/afml/src/afml/meta.py — MetaLabeler; trading/strategies/meta_rsi2.py"
---

# Two questions, once

*Direction is one question. Size is another. We asked them together, for a long time. It is better to ask separately.*

Meta-labeling is an underused technique in financial ML. It decomposes the trading decision into two questions, trains separate models for each, and uses the second to filter the first.

## The decomposition

**Question 1 (primary):** What direction? Long, short, or flat.

**Question 2 (secondary):** Should we take this signal, and at what size? A probability multiplied by a sizing map.

Every trading strategy answers both questions, usually implicitly. Meta-labeling makes them explicit: the primary model emits direction only, and the secondary model decides whether the primary's direction is worth acting on.

*The two questions are answered using different information. Direction is often determined by simple rules. Sizing requires conditional reasoning — given a primary signal, what is the probability that this specific signal succeeds? One model, two tasks: harder. Two models, one each: easier.*

## A mediocre primary, capable secondary

*A concrete illustration.*

Suppose the primary (RSI(2) on SPY) has a 46% hit rate across all signals — below coin-flip.

Now build a secondary classifier trained to predict, given features at signal time (volatility regime, trailing performance, time of day), whether the primary's signal will hit the profit-take barrier. Apply the secondary as a filter: take only signals where $P(\text{take}) > 0.55$. The filtered signals have a 56% hit rate.

The primary is unchanged. Sharpe improves from 0.73 to 2.72 in the stability sweep.

*The secondary did not identify a better direction signal. It identified a better way to use the existing signal. Conditional hit rates are often substantially higher than marginal hit rates. Most primary signals succeed in some conditions and fail in others. The secondary identifies the favorable conditions.*

## The secondary's label

The secondary trains on the binarized version of the triple-barrier direction labels, not on the labels themselves:

$$
\text{meta}_i = \begin{cases} 1 & \text{if label}_i = +1 \\ 0 & \text{otherwise} \end{cases}
$$

This binarization is produced by `meta_label` in `afml.labeling`.

*The side information is already encoded in the triple-barrier label. A profitable short has label $+1$ via the side adjustment, so `meta = 1`. The meta-label is direction-neutral: it distinguishes successful from unsuccessful trades.*

## Features for the secondary

Useful features discriminate between conditions where the primary succeeds and fails:

- **Regime tags.** The `long_gamma` / `short_gamma` / `vol_inverted` tag from the GEX classifier.
- **Trailing primary performance.** The primary's hit rate over the last 20 signals.
- **Entry conditions.** Volatility percentile, recent trend, vol-of-vol.
- **Calendar features.** Time-of-day, day-of-week, proximity to macro events.
- **Price-path features.** Short-window trailing return, recent absolute move size, distance from rolling high.

The capstone strategy uses a small feature set (6 columns) to demonstrate the pattern. Production meta-labelers typically use 20-50 features.

*A structural warning. Secondary features must be computable at primary-signal time. A feature that looks beyond the primary signal introduces leakage regardless of its source.*

## Classifier choice

Gradient boosting (XGBoost, LightGBM, scikit-learn's `GradientBoostingClassifier`) is the usual starting point:

- Non-linear, handling regime interactions naturally.
- Robust to feature scale, no preprocessing needed.
- Tolerant of noisy features via tree depth and learning rate.
- Fast to train, suitable for walk-forward retraining.

Neural networks (small MLPs) are viable alternatives but typically do not outperform boosted trees on financial tabular data with limited samples.

Conservative defaults — small tree depth (3), moderate number of trees (200), lower learning rate (0.05-0.1) — are appropriate.

## Evaluating the secondary

**Avoid raw accuracy.** The meta-label distribution is typically imbalanced.

**Precision and recall on the minority class.** The cases of interest are those where the secondary predicts "take." Precision is the fraction of "take" predictions that were profitable; recall is the fraction of profitable signals identified.

**Calibration.** When soft sizing maps probability to size, the probabilities must be well-calibrated. A classifier reporting 70% confidence should be correct 70% of the time. Gradient boosting is generally reasonably calibrated; when not, Platt scaling or isotonic regression is the correction.

**Final evaluation via Sharpe lift.** The secondary's value is the improvement in the full strategy's Sharpe relative to the primary alone.

## The sizing function

The secondary outputs a probability $P(\text{take}) \in [0, 1]$. Mapping this probability to a position size is more consequential than it appears.

Several valid shapes:

- **Hard threshold**: `size = 1 if p >= threshold else 0`. Simplest; discards probability magnitude.
- **Thresholded soft**: `size = p if p >= threshold else 0`. Retains gradient information; amplifies calibration errors.
- **Linear ramp**: `size = clip((p - p_lo) / (p_hi - p_lo), 0, 1)`. Smooth; tolerant to small calibration biases.
- **Kelly-style**: `size = clip(2p - 1, 0, 1)`. Theoretically optimal under log-utility with accurate probabilities; penalizes miscalibration.

The capstone strategy uses a linear ramp from 0.45 to 0.65. The stability sweep demonstrated robustness: lift remains stable across neighboring ramps and seeds.

## Why decomposition wins

*Before meta-labeling, the standard approach was to train one model to predict return or direction and scale position size by model confidence. This underperformed. A few reasons.*

- **Loss functions conflate direction and magnitude.** A model minimizing MSE optimizes a combination; improvements in one can harm the other.
- **Label imbalance.** Direction labels are noisy; magnitude labels noisier. Training on both directs signal into the noise-heavy regression target.
- **Calibration.** Multi-task outputs are difficult to calibrate jointly.

*Meta-labeling decouples. Primary has one loss. Secondary has another. Each trained for its own task. The decomposition produces a stronger whole than either alone would yield.*

## Pre-deployment checklist

- [ ] The primary emits clean directional signals with sufficient density.
- [ ] Triple-barrier labels with reasonable `pt_mult`, `sl_mult`, `vertical_bars`.
- [ ] Meta-label binarization applied (`afml.labeling.meta_label`).
- [ ] Secondary features are strictly available at entry time.
- [ ] Inner purged k-fold for hyperparameter selection.
- [ ] Outer walk-forward with label-horizon embargo between train and test folds.
- [ ] A stability sweep over hyperparameters and seeds.
- [ ] DSR computed to discount for multiple-testing bias (pending harness implementation).
- [ ] Sizing function chosen with awareness of calibration sensitivity.

The capstone strategy passes all items except DSR, which awaits the harness stub.

## Summary

- The primary-secondary decomposition often improves Sharpe more than improvements to the primary alone: conditional hit rates live in feature space the primary cannot access.
- The secondary's training label is binary (did the primary's bet succeed?), not directional. Direction is already determined.
- Soft sizing amplifies calibration errors: uncalibrated probabilities translate into inappropriate position sizes.

## Implemented at

- `trading/packages/afml/src/afml/meta.py:25` — `MetaLabeler(model, threshold=0.5)`. Wraps any sklearn-compatible binary classifier.
- `trading/packages/afml/src/afml/labeling.py:126` — `meta_label(triple_barrier_out, sides)` binarizes.
- `trading/strategies/meta_rsi2.py` — the capstone: RSI(2) primary, 6-feature secondary, `GradientBoostingClassifier(n_estimators=200, max_depth=3)`, linear-ramp sizer, walk-forward with label-horizon embargo.
- `trading/scripts/sweep_meta_rsi2.py` — hyperparameter × seed stability probe (25 cells, 25/25 beat primary).

---

*Two questions. Separate models. The whole stronger than either. Next: below the minute, a tape of sparks.*

**Next:** [A tape of sparks →](../flows/microstructure.md)
