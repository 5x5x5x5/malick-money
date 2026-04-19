---
title: "How widely does a thing breathe"
prereqs: "Returns, and what changes without us"
arrives_at: "σ — the unit beneath every risk, every barrier, every ratio"
code_ref: "trading/packages/afml/src/afml/labeling.py:41 — rolling_vol"
---

# How widely does a thing breathe

*A market moves, and a market is still. We cannot ask it to hold. We can only measure how wide it opens when it opens. How wide it closes when it closes. This we call volatility.*

Volatility measures the variability of returns, not the magnitude of the final price change. The two numbers that describe a day's activity — the return and the volatility — answer different questions.

Formally, **volatility is the standard deviation of returns**:

$$
\sigma = \sqrt{\mathbb{E}\!\left[(r_t - \bar r)^2\right]}.
$$

*This number lives beneath everything. Every risk metric. Every option price. Every vol-scaled label. Everything after speaks through $\sigma$.*

## Two volatilities, sharing a name

*The word carries two meanings. We must distinguish them, or we will speak past each other for the rest of the curriculum.*

- **Realized volatility** — the standard deviation of observed returns. A measurement, backward-facing.
- **Implied volatility** — the $\sigma$ extracted from an option's market price by inverting Black-Scholes. A forecast, forward-facing.

This lesson addresses realized. Implied receives its own section in [Part IV](../vol-surface/implied-vol.md). The distinction is load-bearing: conflating them makes most of what is written about options markets incoherent.

## Sample, and exponentially weighted

Given $n$ recent returns, the sample standard deviation weights every observation equally:

$$
\hat\sigma^2 = \frac{1}{n - 1} \sum_{t=1}^{n} (r_t - \bar r)^2.
$$

*Equal weighting says: a day from a hundred days ago matters as much as yesterday. Markets do not believe this. Neither should we.*

When a regime shifts, a flat-weighted estimator requires its full window to incorporate the change. By that time, the estimate is stale.

The **exponentially-weighted moving (EWM) standard deviation** gives recent observations more weight:

$$
\hat\sigma^2_t = (1 - \alpha)\, \hat\sigma^2_{t-1} + \alpha\, (r_t - \mu_t)^2,
\quad \alpha = \frac{2}{\text{span} + 1}.
$$

At `span=100`, the current squared deviation contributes approximately 2% of the estimate; weights decay geometrically backward. Regime shifts enter the estimate quickly.

*The past does not disappear, but it softens. This, too, is a kind of grace.*

## Annualization, and the limits of $\sqrt{T}$

Annual volatility is the reporting convention. Daily data is the input. The conversion:

$$
\sigma_\text{annual} = \sigma_\text{daily} \cdot \sqrt{252}.
$$

*Under a dream of independence, variance is additive. The root of variance is the width of the breath. For $T$ days, the width grows as $\sqrt{T}$.*

The derivation relies on iid returns. Markets violate this in two ways:

1. **Returns are not iid.** Volatility clusters; quiet periods and stressed periods group together. Daily variance is time-varying, which breaks the additivity argument.
2. **Returns are not independent across time.** Intraday returns show weak negative autocorrelation. At some longer horizons, weak positive autocorrelation appears.

For daily bars at moderate horizons, the $\sqrt{T}$ rule is accurate enough that corrections are rarely applied. For monthly-to-annual horizons and for regime-switching strategies, the approximation error matters.

## Clustering — the stylized fact

*Large follows large. Calm follows calm. This is not a theorem. It is what the markets have shown us, again and again, for as long as we have watched.*

Volatility is serially correlated even when returns are not. The random-walk null cannot reproduce this pattern; GARCH models exist to capture it.

A practical consequence: when realized volatility spikes, the probability of another large move the following day is elevated. Strategies that ignore clustering produce backtests smoother than live trading, because stress episodes are averaged into calm ones during training.

## A concrete year

SPY daily returns, 2020:

- Full-year sample std of arithmetic returns: approximately 2.1% per day (annualized ~33%).
- EWM (`span=100`) std through March 2020: peaked near 5% per day (annualized ~80%).
- EWM through December 2020: approximately 0.8% per day (annualized ~13%).

*Three numbers describing one year. One of them is the average of the others. None of them is the year.*

For position sizing based on current risk, the EWM is the appropriate estimator.

## What the trading project uses

The triple-barrier labeling pipeline computes a per-event vol target against which barriers are calibrated. It uses the EWM standard deviation of arithmetic returns:

```python
def rolling_vol(prices: pl.Series, span: int = 100) -> pl.Series:
    returns = prices.pct_change()
    return returns.ewm_std(span=span)
```

Two choices are encoded:

- **EWM rather than sample.** Recent data is intentionally given more weight. A 2σ barrier in 2017 represents a smaller dollar move than a 2σ barrier in March 2020, by design.
- **Arithmetic rather than log.** Barriers compare `p_t / p_0 - 1` against `mult * sigma`, requiring matching units.

## Summary

- A strategy's live Sharpe can diverge from its backtest Sharpe when the volatility regime has shifted and a sample-std estimator has not caught up.
- Options traders attend to the realized-versus-implied spread rather than either level alone; the two quantities carry different information.
- The $\sqrt{T}$ annualization rule is accurate at daily horizons and degrades at multi-month horizons and post-spike regimes.

## Implemented at

`trading/packages/afml/src/afml/labeling.py:41` — `rolling_vol(prices, span=100)`. EWM standard deviation of arithmetic returns, cited to AFML ch. 3.1. Used by `apply_triple_barrier` at line 52 of the same file to scale barrier thresholds per event.

---

*How widely does the thing breathe. That question, asked every day, is the ground of everything after. Next, the walk beneath it.*

**Next:** [The drunkard, the photon, the stock →](random-walks.md)
