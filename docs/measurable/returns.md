---
title: "Returns, and what changes without us"
prereqs: "none"
arrives_at: "the units every measurement speaks"
code_ref: "trading/packages/afml/src/afml/labeling.py:41 — rolling_vol"
---

# Returns, and what changes without us

*A number moved. Something in the ledger, dim in the morning, became another thing. We did not move it. We will never know who did. We only see what is left. So we give the leaving a name. We call it a return.*

A stock that goes from $100 to $102 has produced a 2% gain. This is the **arithmetic return**:

$$
r_t = \frac{p_t - p_{t-1}}{p_{t-1}} = \frac{p_t}{p_{t-1}} - 1.
$$

*In a single period, this is enough. One day, one breath, one small recording of what changed.*

*But time does not stop. The days stack. The breaths accumulate. And the arithmetic — honest in isolation — begins to lie.*

## What arithmetic returns cannot carry

A stock that gains 10% on Monday and loses 10% on Tuesday does not end where it began. It ends at $100 \times 1.10 \times 0.90 = 99$. Arithmetic returns do not add; they **compound**. Over $n$ periods:

$$
p_n = p_0 \prod_{t=1}^{n} (1 + r_t).
$$

*Products are heavy. They do not let us take an average and mean it.*

The arithmetic mean of returns is always at least the geometric mean, with equality only when every return is identical (Jensen's inequality applied to $\log(1+r)$). To report a strategy's mean daily return without correction is to overstate its annualized performance, systematically.

*There is a truer unit, if we are willing to take it. We must take it logarithmically.*

## Log returns, and the algebra they permit

Define the **log return** over one period:

$$
\ell_t = \log\!\left(\frac{p_t}{p_{t-1}}\right) = \log p_t - \log p_{t-1}.
$$

Under this definition, multi-period compounding becomes a sum:

$$
\log\frac{p_n}{p_0} = \sum_{t=1}^{n} \ell_t.
$$

*Addition, where before was multiplication. The days no longer weigh upon each other. They gather, quietly.*

Three properties follow:

1. **Additive across time.** The sum of daily log returns equals the total log return — in expectation under iid, and exactly in realization.
2. **Symmetric.** A $+10\%$ log move is reversed by a $-10\%$ log move. Arithmetic returns are not symmetric: a 10% gain followed by a 10% loss leaves a 1% net loss.
3. **Closer to Gaussian.** Log prices under geometric Brownian motion are Gaussian by construction. Markets depart from GBM, but log returns remain closer to normal than arithmetic returns do.

## Where the difference matters

For small moves, $\log(1 + r) \approx r - r^2/2$: log and arithmetic returns agree to first order. A 1% daily return differs by 5 basis points out of 100.

*For small things, there is no difference. Only for the moves that stay in memory.*

| Arithmetic $r$ | Log $\ell$ | Difference |
|---------------|-----------|------------|
| $+1\%$        | $+0.995\%$ | 5 bp       |
| $+10\%$       | $+9.53\%$  | 47 bp      |
| $-10\%$       | $-10.54\%$ | 54 bp      |
| $-30\%$       | $-35.67\%$ | 567 bp     |

For daily bars in liquid markets, either convention holds. For weekly or longer bars, for drawdown accounting, and for nonlinear statistics — **volatility especially**, covered next — use logs.

## What the trading project keeps

The [triple-barrier method](../ml/triple-barrier.md) uses arithmetic returns: `(p_t / p_0 - 1) * side`. This follows AFML ch. 3.1 and keeps barriers and vol in matching units — both computed from arithmetic returns — so the thresholds are consistent.

*A choice, made once, kept. Every label the strategy writes is downstream of this small fidelity.*

## Summary

By the end of this lesson, the following are reasonable to hold:

- A strategy's cumulative return is not the mean daily return times the number of days.
- A Sharpe computed on log returns can differ from one computed on arithmetic returns, even for identical price data.
- Options pricing (covered in Part II) assumes log-normal terminal prices — not normal ones — for reasons this lesson already names.

## Implemented at

`trading/packages/afml/src/afml/labeling.py:41` — `rolling_vol(prices, span=100)` computes the EWM standard deviation of arithmetic `pct_change` returns. The docstring cites AFML ch. 3.1. Downstream, `apply_triple_barrier` at line 52 compares `(p/p0 - 1)` against `mult * sigma`, so vol must be in arithmetic-return units.

---

*A number moved. And we, who could not move it, gave the leaving a name. Already we have spoken too much. Next, the width of the breath.*

**Next:** [How widely does a thing breathe →](volatility.md)
