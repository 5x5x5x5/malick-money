---
title: "Did it work, and what does 'work' mean"
prereqs: "Returns; Volatility"
arrives_at: "the metrics every strategy report lists, and their failure modes"
code_ref: "trading/packages/harness/src/harness/metrics.py"
---

# Did it work, and what does 'work' mean

*Did it work? Say it aloud. Define 'did'. Define 'work'. Define 'it'. We have tools for each, and each tool lies a little. This lesson is a catalogue of the lies, as much as the tools.*

A strategy produces a return series. The question of whether it worked requires a small number of summary statistics. The metrics catalogue below is the minimum viable set — the quantities every backtest report should include. Each has a definition, a measurement target, and a failure mode.

## Sharpe ratio

$$
\text{Sharpe} = \frac{\bar r - r_f}{\sigma_r} \sqrt{A}
$$

where $\bar r$ is mean per-period excess return, $\sigma_r$ its standard deviation, $r_f$ the risk-free rate per period, and $A$ the annualization factor (252 for daily U.S. equity, 52 for weekly, 12 for monthly).

*How many standard deviations does the annualized mean return exceed zero? That is the question Sharpe answers. Nothing more.*

Approximate reference levels:

| Sharpe | Typical interpretation |
|--------|------------------------|
| < 0 | Losing money |
| 0 – 0.5 | Poor; barely differs from noise |
| 0.5 – 1.0 | Modest; viable with scale |
| 1.0 – 1.5 | Good; tradeable book |
| 1.5 – 2.5 | Very good; institutional quality |
| > 2.5 | Exceptional; warrants scrutiny for look-ahead |

Backtest Sharpes tend to exceed live Sharpes. [Deflated Sharpe](deflated-sharpe.md) later in this part quantifies the gap.

### The failure modes of Sharpe

1. **Symmetric volatility penalty.** Sharpe penalizes upside volatility as heavily as downside. Two distributions with the same spread can have different utility.
2. **No tail sensitivity.** Sharpe uses only the first and second moments. Two strategies with identical Sharpes can have very different distributions.
3. **iid assumption.** The formula assumes returns are independent across time. Momentum has positive autocorrelation; mean-reversion has negative. Sharpe misreports risk-adjusted return under both.
4. **Bias under non-normality.** For strategies with negatively skewed returns, observed Sharpe in quiet periods overstates long-run Sharpe because the correcting tail event has not yet occurred.
5. **Multiple testing.** Given many strategies, the best-Sharpe one is likely the luckiest rather than the best.

*Five failures, and Sharpe still appears on every tearsheet. We must know where it lies, and where it does not.*

## Sortino ratio

A partial remedy for the symmetric-volatility issue. Only downside deviation enters the denominator:

$$
\text{Sortino} = \frac{\bar r - \tau}{\sigma_\text{down}} \sqrt{A}, \qquad \sigma_\text{down} = \sqrt{\mathbb{E}\!\left[\min(r - \tau, 0)^2\right]}.
$$

$\tau$ is a target return (usually zero). Upside volatility no longer penalizes the metric. Sortino is always at least equal to Sharpe under symmetric distributions, and strictly greater under positive skew.

## Max drawdown

$$
\text{MaxDD} = \min_{t \le T}\left(\frac{V_t - \max_{s \le t} V_s}{\max_{s \le t} V_s}\right).
$$

Peak-to-trough worst loss, expressed as a non-positive fraction. A value of −0.15 corresponds to a 15% drawdown.

*This is the number investors actually remember. A Sharpe of 2 is impressive. A 40% drawdown is what drives the allocation decision.*

Max drawdown should be paired with recovery time — the duration required to regain the prior peak — for a complete loss profile.

## Turnover

Mean absolute position change per period:

$$
\text{Turnover} = \mathbb{E}\!\left[|p_t - p_{t-1}|\right].
$$

A strategy that holds a position indefinitely has turnover 0. A strategy that flips fully each day has turnover 2. Turnover does not appear in Sharpe, but it controls costs.

*Two strategies with identical gross Sharpe but different turnover have different net Sharpe. The lower-turnover strategy often outperforms live. The backtest did not lie; it priced the wrong version.*

## Capacity

A strategy is capacity-limited when scaling its notional size materially degrades its returns through slippage. Two mechanisms:

1. **Market impact.** Orders move prices against themselves.
2. **Liquidity.** Some edges exist only in illiquid market segments.

The capacity estimate in `metrics.py:capacity_estimate` is a stub. The intended method: determine the notional position at which slippage erodes 50% of realized Sharpe.

## Deflated Sharpe (preview)

Selecting the best-Sharpe strategy from 100 trials is cherry-picking. Bailey & López de Prado's Deflated Sharpe Ratio corrects:

$$
\text{DSR} = \mathbb{P}(\text{true SR} > 0 \mid \text{observed SR}, N \text{ trials}).
$$

The probability that the true Sharpe exceeds zero, given the observed statistic and the number of trials. Covered in its own lesson.

## Selecting metrics

A standard report includes:

- Sharpe for the risk-adjusted headline.
- Sortino for a downside-only view.
- Max DD and recovery time for worst-case characterization.
- Turnover and net-of-costs Sharpe for the live-trading projection.
- Higher moments (skewness, kurtosis) when distributions deviate from Gaussian.
- DSR when hyperparameter sweeps are involved.

*Reporting only Sharpe is not incorrect. It is incomplete.*

## Annualization

The $\sqrt{A}$ factor derives from the additive-variance argument in Part I:

- Daily equity returns: $A = 252$.
- Weekly: $A = 52$.
- Monthly: $A = 12$.
- Already-annualized: $A = 1$.
- Crypto (continuous trading): $A = 365$.

Incorrect annualization is a common source of error. The metrics functions default to $A = 252$ and accept overrides.

## Summary

- A Sharpe of 2.0 with 40% max DD differs materially from Sharpe 1.5 with 15% max DD for allocation purposes.
- Turnover controls net-of-costs returns even though it does not appear in the Sharpe formula.
- The five failure modes of Sharpe (symmetric vol penalty, no tail sensitivity, iid assumption, non-normal bias, multiple testing) motivate the rest of Part VI.

## Implemented at

`trading/packages/harness/src/harness/metrics.py`:

- Line 19: `sharpe(returns, annualization=252, risk_free=0)`.
- Line 38: `sortino(returns, annualization=252, target=0)`.
- Line 57: `max_drawdown(equity)`.
- Line 71: `turnover(positions)`.
- Line 83: `deflated_sharpe(returns, num_trials, annualization=252)` — stub, `NotImplementedError`.
- Line 104: `capacity_estimate(...)` — stub.
- Line 120: `expected_max_sharpe(num_trials, std)` — helper used by DSR.

*The stubs are the curriculum's open items. DSR is the highest-value one to implement — it changes what a strategy report can honestly claim.*

---

*Did it work. Say it aloud. Next: how to ask the question while respecting time.*

**Next:** [You cannot stand in tomorrow →](walk-forward.md)
