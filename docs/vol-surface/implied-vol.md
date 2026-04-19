---
title: "The market does not know tomorrow"
prereqs: "Three men, one formula"
arrives_at: "the market's forecast of σ, and the premium that persists above realized"
code_ref: "—"
---

# The market does not know tomorrow

*The market does not know tomorrow. And yet it speaks, as if it did. We listen. We write down what it says. We call it implied.*

Black-Scholes takes five inputs and produces a price. Four — $S$, $K$, $r$, $T$ — are observable or contractual. The fifth, $\sigma$, is not. The market resolves this by letting observed premiums define $\sigma$: inverting the formula against market prices yields the **implied volatility** (IV).

This lesson covers what IV measures, how it differs from realized volatility, and the variance risk premium that persists between them.

## Inverting Black-Scholes

Given a market premium $C^\text{mkt}$ for a call with known $S$, $K$, $r$, $T$, the task is to find $\sigma$ such that:

$$
C_\text{BS}(S, K, r, T, \sigma) = C^\text{mkt}.
$$

The Black-Scholes formula is monotonic in $\sigma$. The inversion has a unique solution. Numerical solvers find it in microseconds. The resulting $\sigma$ is the option's implied volatility.

Every option on the chain has its own. If Black-Scholes were fully accurate, every option on a given underlying would share a single IV. In practice, IVs vary across strikes (skew, smile) and expiries (term structure). These deviations are the subject of the next two lessons.

## IV is a forecast, not a measurement

*An option's implied volatility is not a reading of what happened. It is a reading of what the market expects, now, for the contract's remaining life.*

In practice it combines three components:

1. The market's estimate of future realized volatility.
2. A risk premium for bearing volatility risk — sellers require compensation for tail events.
3. Supply-demand imbalances specific to the option (buyer demand for protection, dealer inventory).

These cannot be separated from a single IV quote. Comparing IV to subsequent realized volatility across many observations allows estimation of the average gap.

## The variance risk premium

*IV tends to exceed subsequent realized volatility, on average. Especially for short-dated equity-index options. The spread has a name.*

| Approximate figures, SPX 30-day, decade-average |                   |
|-------------------------------------------------|-------------------|
| Mean VIX (implied)                              | ~18%              |
| Mean realized 30-day vol                        | ~14%              |
| Mean spread (premium)                           | ~4 percentage points |

The spread is not constant. It compresses near zero (and occasionally inverts) in quiet regimes, and widens to 10 points or more around events. The unconditional average has remained meaningfully positive across multi-decade samples.

*Selling options is economically equivalent to selling insurance. The seller takes a small premium to accept convex losses in tail events. Risk-averse agents pay more than actuarial fairness would require. Over time, the insurance costs more than its expected payoff — and sellers, on average, profit.*

Short-volatility strategies — naked puts, strangles, variance swaps — produce positive unconditional returns, subject to occasional tail losses that erase years of accumulated gains. Long-Term Capital in 1998. Volmageddon in February 2018. COVID in March 2020. Each had short-vol books at the center.

*The VRP is real. It is not free. Its price is the occasional catastrophe, rare but certain on a long enough horizon.*

## VIX — a model-free IV

The CBOE Volatility Index reports a 30-day implied-volatility estimate on the S&P 500 **without assuming Black-Scholes**. The construction:

$$
\text{VIX}^2 = \frac{2 e^{rT}}{T} \sum_{i} \frac{\Delta K_i}{K_i^2} Q(K_i) - \frac{1}{T}\left(\frac{F}{K_0} - 1\right)^2.
$$

Here $Q(K_i)$ is the OTM option price at strike $K_i$ (puts below forward, calls above), $\Delta K_i$ is the strike spacing, $F$ is the forward price, $K_0$ is the first strike below $F$. The formula comes from the static replication of a variance swap via a continuum of OTM options — a mathematical identity stating that the fair strike of a variance swap equals a specific weighted integral of OTM option prices.

Two points:

1. No Black-Scholes is required. The formula uses observed option prices directly and makes no assumption about the dynamics of $S$. VIX is model-free in this sense (subject to the variance-swap replication formula being exact — requiring a continuum of strikes and no jumps).
2. VIX is reported as a volatility, not a variance. A VIX of 20 corresponds to 20% annualized volatility.

*VIX is computed in real time from a specific set of SPX options. The two nearest maturities, weighted to bracket 30 days. Analogous indices for 9-day (VIX9D) and 3-month (VIX3M), used in the next lesson to measure slope.*

## Realized volatility estimators

If IV is a forecast, realized volatility is the measurement. The standard estimate is the EWM or rolling std of log returns, annualized by $\sqrt{252}$.

Three subtleties:

- **Sampling frequency.** Daily close-to-close returns miss intraday volatility. Higher-frequency estimators (Parkinson using high-low, Garman-Klass using OHLC, realized variance from intraday returns) use more of the data.
- **Jumps.** A close-to-close estimator combines continuous volatility and jumps. Bipower variation separates them.
- **Microstructure noise.** Very high-frequency returns carry bid-ask-bounce noise; realized variance overstates volatility when sampled too finely without correction.

For comparison to IV, the standard choice is daily close-to-close log returns annualized by $\sqrt{252}$.

## Summary

*Two uses of the word volatility. Two directions in time. One spread between them.*

- The word "volatility" ambiguously references two distinct quantities — a measurement (realized) and a forecast (implied). Conflation makes options commentary incoherent.
- Short-volatility strategies have positive expected returns and severe tail properties: they harvest a real premium that compensates for real tail risk.
- VIX is constructed model-free from option prices via variance-swap replication, and is a statement about the options market rather than about Black-Scholes.

## Implemented at

The GEX pipeline consumes IV as input rather than recomputing it from premiums. Data sources in `trading/packages/gex/src/gex/data/options.py` provide per-contract IV from the upstream chain feed. The VIX series is pulled via `data/vix.py::fetch_vix_history`, and the regime classifier in `regime.py:classify_regime` uses VIX/VIX3M and VIX9D/VIX slopes as inversion triggers. Those slopes are the next lesson.

---

*The market does not know. But it speaks. Next: the shape of what it says, across horizons.*

**Next:** [Across horizons →](term-structure.md)
