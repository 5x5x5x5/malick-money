---
title: "The drunkard, the photon, the stock"
prereqs: "How widely does a thing breathe"
arrives_at: "the baseline hypothesis every strategy must beat"
code_ref: "—"
---

# The drunkard, the photon, the stock

*A drunkard stumbles home. A photon crosses a room. A stock, a day. All wander. None remember. This is the walk.*

Every strategy competes against a specific null hypothesis: tomorrow's price equals today's price plus noise. This is the random walk, and it underlies the mathematics of most downstream models.

The random walk is a baseline, not a forecast. A strategy whose returns are indistinguishable from noise on top of a walk has no identifiable edge. Reliable deviations indicate the presence of a signal.

## From coin flips to continuous paths

Imagine a coin-flip game on log-prices. At each step $t$, the log-price changes by $+\sigma$ with probability $1/2$ or $-\sigma$ with probability $1/2$. After $n$ steps:

$$
\log S_n - \log S_0 = \sum_{t=1}^{n} \epsilon_t, \qquad \epsilon_t \in \{+\sigma, -\sigma\}.
$$

*Two properties hold as the game grows.*

1. The mean of the sum remains zero (a fair coin).
2. The standard deviation of the sum grows as $\sigma \sqrt{n}$ — the $\sqrt{T}$ law, returned.

Shrink the step. Shrink the time between steps. Hold total variance per unit time fixed. The Central Limit Theorem gives a Gaussian limit, variance linear in time. This is **Brownian motion**:

$$
W_t \sim \mathcal{N}(0, t), \qquad W_{t+s} - W_t \sim \mathcal{N}(0, s), \text{ independent of } \mathcal{F}_t.
$$

*Independent. Memoryless. The increment does not know what came before.*

## Drift, and the log-price walk

Real markets drift upward over long horizons — the equity risk premium. A standard model combining drift, Gaussian noise, and the constraint that prices remain non-negative is **geometric Brownian motion** (GBM):

$$
dS_t = \mu S_t\, dt + \sigma S_t\, dW_t.
$$

On log-price:

$$
d(\log S_t) = \left(\mu - \tfrac{1}{2}\sigma^2\right) dt + \sigma\, dW_t.
$$

*There is a correction, small in appearance, vast in consequence. Minus half sigma squared. It is a shadow we cast when we move from prices to logs.*

The $-\tfrac{1}{2}\sigma^2$ term arises because $\log S_t$ is a concave function of $S_t$; Itô's lemma (equivalently, a second-order Taylor expansion) picks up a variance term when converting drift from prices to log-prices.

The consequence: the **geometric mean** of returns — the quantity that compounds — is less than the arithmetic mean by approximately $\sigma^2/2$. Long-horizon projections that use a daily-return estimate without this correction overstate expected compounded returns.

## Where GBM fails, and why we still use it

*Markets are not GBM. They never were. But there is a reason the model holds a place at every table, for every price we have tried to draw from noise.*

Five departures:

- **Fat tails.** GBM predicts 4-sigma daily moves roughly once every 15,000 trading days. SPY has seen several per decade.
- **Volatility clustering.** GBM assumes constant $\sigma$. Real $\sigma$ is serially correlated. GARCH and stochastic-vol models extend GBM to capture this.
- **Leverage effect.** Negative returns increase future volatility more than positive returns of the same magnitude. GBM is symmetric; markets are not.
- **Jumps.** GBM paths are continuous; real prices gap.
- **Short-horizon autocorrelation.** Brownian increments are independent. Intraday returns show weak negative autocorrelation.

*These are not footnotes. They are the subject matter of most quant research. The walk is the baseline. The strategies are the ways in which the walk is not quite true.*

## The null, and the Sharpe ratio

*When we compute a Sharpe, we are asking: how many standard deviations from zero is our mean? The question is written in the language of the walk.*

The Sharpe denominator implicitly assumes iid Gaussian noise around a constant mean — the random-walk null. The further a strategy's returns depart from that assumption, the less the Sharpe statistic means what it appears to mean.

Strategies that concentrate edge in rare large wins — event-driven, short-volatility — produce non-Gaussian distributions that a Sharpe ratio misweights. [Deflated Sharpe](../backtest/deflated-sharpe.md) partially corrects for this with skewness and kurtosis terms. No single statistic fully captures a non-iid return series.

## Options, and the log-normal terminal

Black-Scholes begins with GBM. The option price at expiry depends on the terminal distribution of $S_T$, which under GBM is log-normal:

$$
\log S_T \sim \mathcal{N}\!\left(\log S_0 + (\mu - \tfrac{1}{2}\sigma^2) T, \; \sigma^2 T\right).
$$

*The shadow returns. It is everywhere we look.*

Two consequences:

1. Terminal price is always non-negative (log-normal has support on $(0, \infty)$). Prices cannot go below zero under GBM.
2. A pricing model with a single constant $\sigma$ produces a flat implied-vol surface. The observed surface is not flat. The deviations from GBM manifest as shape on the vol surface — and that shape is tradable information, covered in [Part IV](../vol-surface/implied-vol.md).

## Alpha, under the null

The term *alpha* appears in papers and attribution reports. Mathematically, alpha is the systematic return earned above what the null predicts. Regressing strategy returns on market returns:

$$
r_\text{strat} = \alpha + \beta\, r_\text{market} + \epsilon,
$$

gives $\alpha$ as the intercept — expected return when market returns are zero. A strategy with a statistically significant non-zero $\alpha$ earns returns not explained by market exposure alone.

*This is the thing a random walk cannot produce. To find it is difficult. Everything in the measure section (Part VI) exists so that we are not fooled into thinking we have found it when we have not.*

## Summary

- Options pricing assumes log-normal terminal prices, not normal ones; the $-\sigma^2/2$ correction on the drift of log-price follows from Itô's lemma.
- Next-day return forecasting is structurally difficult: the target is deviation from a walk whose single-period expectation is small relative to its noise.
- A Sharpe above 2 is less informative than it appears when returns are non-Gaussian or serially correlated; the standard statistic implicitly assumes these conditions away.

## Implemented at

The null is assumed throughout the repository rather than computed:

- `trading/packages/harness/src/harness/metrics.py:19` — `sharpe(returns)` compares mean to standard deviation, implicitly against a zero-mean iid null.
- `trading/packages/gex/src/gex/greeks.py` — uses Black-Scholes, which operates within GBM.
- `trading/packages/afml/src/afml/structural.py` — `cusum_events` and `chow_dfc_stat` test for deviation from the random-walk null, flagging structural breaks.

---

*The walk, and what it cannot see. Having measured, we turn to what is measured. Next, the contract — a promise you need not keep.*

**Next:** [A promise you need not keep →](../options/contracts.md)
