---
title: "You tried a hundred"
prereqs: "Sharpe, Sortino, max drawdown; Walk-forward vs k-fold"
arrives_at: "the multiple-testing-corrected Sharpe"
code_ref: "trading/packages/harness/src/harness/metrics.py — deflated_sharpe"
---

# You tried a hundred

*You tried a hundred. The best was lucky. How do we know? The math knows. It has always known. We only need to ask.*

A Sharpe of 1.5 on a single backtest is a meaningful result. A Sharpe of 1.5 as the best of 100 backtests is less informative than noise. The difference is multiple testing, and the Deflated Sharpe Ratio (DSR) is the statistical correction that turns a raw Sharpe into an honest headline.

## The multiple-testing trap

Trying 100 strategy variants — different parameters, features, or signals — and selecting the best-Sharpe result without adjustment produces biased estimates. If the true Sharpe of every strategy is zero, the expected Sharpe of the best is substantially positive.

With 100 iid noise trials of 252 daily returns each, the expected maximum sample Sharpe is roughly 0.8. With 1,000 trials, approximately 1.0. With 10,000 trials — easily reached by automated hyperparameter grids — more than 1.2.

*These Sharpes can be generated from random data by trying enough trials. The best-of-N sample is not a sample from the underlying distribution. It is a sample from the maximum of N draws, which has a fundamentally different (higher) mean.*

## Expected maximum Sharpe under the null

Bailey & López de Prado derived an approximation for the expected maximum Sharpe across $N$ iid trials, assuming true Sharpe zero with cross-trial standard deviation $\sigma_{SR}$:

$$
\mathbb{E}[\max \text{SR}] \approx \sigma_{SR} \left[ (1 - \gamma) \Phi^{-1}\!\left(1 - \tfrac{1}{N}\right) + \gamma\, \Phi^{-1}\!\left(1 - \tfrac{1}{N e}\right) \right]
$$

where $\gamma \approx 0.5772$ is Euler-Mascheroni, $\Phi^{-1}$ is the inverse standard normal CDF, and $\sigma_{SR}$ is the standard deviation of true Sharpe ratios across trials.

$\mathbb{E}[\max \text{SR}]$ grows rapidly in $N$. For $\sigma_{SR} = 0.5$:

| $N$ | $\mathbb{E}[\max \text{SR}]$ (approximate) |
|-----|--------------------------------------------|
| 1 | 0.00 |
| 10 | 0.66 |
| 100 | 1.22 |
| 1,000 | 1.59 |
| 10,000 | 1.92 |

*After a thousand trials, the expected best-case under pure noise is 1.59. A reported Sharpe of 1.5 from a grid search of this size falls within the null's expected range.*

## Non-normality correction

A second correction. Sharpe's sampling distribution assumes Gaussian returns; real returns have skew and kurtosis. The denominator of the Sharpe test statistic understates variance for non-Gaussian distributions, inflating the reported statistic.

The correction uses sample skewness $\hat\gamma_3$ and excess kurtosis $\hat\gamma_4$:

$$
\sigma(SR) \approx \sqrt{\frac{1 - \hat\gamma_3 \cdot SR + \frac{\hat\gamma_4}{4} \cdot SR^2}{T - 1}}.
$$

Negative skew and fat tails both widen the confidence interval on the Sharpe.

## The DSR

Given the observed Sharpe $\widehat{SR}$, number of trials $N$, cross-trial std $\sigma_{SR}$, sample skew $\hat\gamma_3$, excess kurtosis $\hat\gamma_4$, and sample size $T$, the DSR is:

$$
\text{DSR} = \Phi\!\left( \frac{(\widehat{SR} - \mathbb{E}[\max \text{SR}]) \sqrt{T - 1}}{\sqrt{1 - \hat\gamma_3 \widehat{SR} + \frac{\hat\gamma_4}{4} \widehat{SR}^2}} \right).
$$

*A probability in $(0, 1)$. The probability that the true Sharpe exceeds zero, conditional on the observed statistic, the number of trials, and the non-normality of returns.*

Values near 1 indicate the observed Sharpe substantially exceeds what the null would produce. Values near 0.5 are indistinguishable from chance. A common threshold: a strategy is "statistically meaningful" when $\text{DSR} > 0.95$.

## Worked interpretation

Suppose 200 strategy variants were run, and the best had:

- $\widehat{SR} = 1.8$ (annualized, $T = 1{,}260$ daily observations = 5 years).
- Sample skewness $\hat\gamma_3 = -0.5$ (mild negative, typical for short-vol).
- Sample excess kurtosis $\hat\gamma_4 = 2.0$.
- Cross-trial $\sigma_{SR} = 0.4$.

Compute $\mathbb{E}[\max \text{SR}]$ for $N = 200$, $\sigma_{SR} = 0.4$: approximately $1.24$.

Denominator: $\sqrt{1 - (-0.5)(1.8) + (2.0/4)(1.8)^2} = \sqrt{1 + 0.9 + 1.62} = \sqrt{3.52} \approx 1.88$.

Plug in: $(1.8 - 1.24) \cdot \sqrt{1259} / 1.88 \approx 0.56 \cdot 35.5 / 1.88 \approx 10.6$.

$\Phi(10.6) \approx 1.0$. DSR effectively 1. The observed Sharpe is substantially above the multiple-testing null.

*Change one input. Let $\widehat{SR} = 1.3$ instead of 1.8.*

Denominator: $\sqrt{1 + 0.65 + 0.85} = \sqrt{2.5} \approx 1.58$.

Plug: $(1.3 - 1.24) \cdot 35.5 / 1.58 \approx 1.35$.

$\Phi(1.35) \approx 0.91$. DSR = 0.91 — below the 0.95 threshold. Plausibly within the null's expected range.

*Raw Sharpe changes of 0.5 swing DSR from probably-real to probably-noise at N = 200 trials.*

## What counts as a trial

$N$ should be the total number of distinct strategy variants considered, not only those retained for the final report:

- Every hyperparameter combination in any grid search.
- Every signal tried and discarded.
- Every feature set tried and discarded.
- Every manual parameter adjustment made after viewing intermediate results.

The last category is difficult to track. Each time backtest output is examined and a change is made, a trial has been consumed. Pre-registering the parameter space makes $N$ a known quantity.

*DSR with overstated $N$ is conservative. DSR with understated $N$ is optimistic. Err toward conservative.*

## Harness implementation

`harness.metrics.deflated_sharpe` is a `NotImplementedError` stub. The module includes partial machinery — `expected_max_sharpe(num_trials, std_across_trials)` at line 120 — but the full DSR with the non-normality correction is pending.

*When implemented, `scripts/sweep_meta_rsi2.py` should report DSR at the tested $N$. The difference between "25/25 cells beat primary" and "25/25 cells beat the multiple-testing-adjusted null" separates the current sweep from a deployment-grade stability check.*

## Summary

- Reporting a Sharpe without disclosing the number of strategies tried is misleading; multiple-testing bias is substantial at realistic grid-search sizes.
- The two corrections that convert a raw Sharpe into a DSR: multiple testing ($\mathbb{E}[\max \text{SR}]$) and non-normality (adjusted variance via skewness and kurtosis).
- Trial counting is operationally difficult; every discarded variant counts, including manual adjustments. Pre-registration is the cleanest remedy.

## Implemented at

`trading/packages/harness/src/harness/metrics.py`:

- Line 83: `deflated_sharpe(returns, num_trials, annualization)` — `NotImplementedError` stub.
- Line 120: `expected_max_sharpe(num_trials, std_across_trials)` — Bailey-López de Prado helper, implemented.

---

*You tried a hundred. The math knows. The last thing before we learn how to learn from the noise itself.*

**Next:** [What is a signal worth taking →](../ml/labeling-problem.md)
