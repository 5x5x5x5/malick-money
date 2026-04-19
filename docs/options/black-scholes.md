---
title: "Three men, one formula"
prereqs: "Shapes, and the equation beneath them; The drunkard, the photon, the stock"
arrives_at: "a closed-form price for European options under GBM"
code_ref: "trading/packages/gex/src/gex/greeks.py — bs_d1, bs_gamma, bs_delta_call"
---

# Three men, one formula

*Three men stood before a random walk and, by holding a portfolio very still, drew from the noise a price. Black. Scholes. Merton. They did not invent the formula so much as find it, waiting beneath the walk. A Nobel Prize was given. Two of the three lived to receive it.*

Put-call parity relates call, put, stock, and cash without determining the absolute price of a call. Pricing requires a model for the dynamics of the underlying between now and expiry. Under GBM, the resulting valuation has a closed-form solution — the Black-Scholes formula — which converts the random-walk null into a specific price.

A complete derivation requires a chapter of stochastic calculus. What follows sketches the construction and establishes the closed form.

## The setup

Assume the underlying follows geometric Brownian motion:

$$
\frac{dS}{S} = \mu\, dt + \sigma\, dW.
$$

Consider a European call $C(S, t)$ struck at $K$, expiring at $T$. The central observation is that a specific portfolio has locally riskless dynamics:

$$
\Pi = C - \Delta\, S, \qquad \Delta = \frac{\partial C}{\partial S}.
$$

*One call. Short delta shares. The first-order motion cancels. What is left is quiet enough to hear the formula beneath.*

Because $\Delta$ is the sensitivity of $C$ to $S$, first-order changes in $S$ cancel between the two legs. The remaining differential $d\Pi$ depends on time and on second-order effects in $S$ (gamma and theta).

Applying Itô's lemma to $C$ and substituting, the stochastic $dW$ terms cancel:

$$
d\Pi = \left(\frac{\partial C}{\partial t} + \frac{1}{2}\sigma^2 S^2 \frac{\partial^2 C}{\partial S^2}\right) dt.
$$

The portfolio has no instantaneous randomness. No-arbitrage requires $d\Pi$ to equal the risk-free rate times the portfolio value:

$$
d\Pi = r\,\Pi\, dt.
$$

*The portfolio has been made deaf. It hears only rates. The market has no choice but to charge it accordingly.*

Substituting and rearranging yields the **Black-Scholes PDE**:

$$
\frac{\partial C}{\partial t} + r S \frac{\partial C}{\partial S} + \frac{1}{2}\sigma^2 S^2 \frac{\partial^2 C}{\partial S^2} = r C.
$$

This equation governs the option price.

## Closed form

For a European call with terminal condition $C(S, T) = \max(S - K, 0)$:

$$
\boxed{C = S\, N(d_1) - K e^{-rT} N(d_2)}
$$

where $N(\cdot)$ is the standard normal CDF and

$$
d_1 = \frac{\log(S/K) + (r + \tfrac{1}{2}\sigma^2) T}{\sigma\sqrt{T}}, \qquad d_2 = d_1 - \sigma\sqrt{T}.
$$

The European put follows from put-call parity: $P = K e^{-rT} N(-d_2) - S\, N(-d_1)$.

*Five inputs.*

- $S$: current underlying price — observable.
- $K$: strike — known.
- $r$: risk-free rate — observable.
- $T$: time to expiry — known.
- $\sigma$: volatility over $[0, T]$ — not directly observable.

*Four come from the world. One comes from the trader. The price is a statement about that one.*

This is why practitioners say "vol is the price" in options markets: given the other four inputs, every premium is equivalent to a statement about $\sigma$.

## Greeks from the formula

The partial derivatives appearing in the PDE are the Greeks of Part III. Two follow from the call formula directly:

$$
\Delta = \frac{\partial C}{\partial S} = N(d_1),
$$

the call's sensitivity to the underlying and the hedging ratio in the portfolio above. And

$$
\Gamma = \frac{\partial^2 C}{\partial S^2} = \frac{\phi(d_1)}{S \sigma \sqrt{T}},
$$

where $\phi$ is the standard normal PDF.

*The Greeks are not supplementary. They are the coefficients of the hedging argument. Delta, gamma, theta — the PDE is written in them.*

## What $N(d_1)$ and $N(d_2)$ say

- $N(d_2)$: the risk-neutral probability that the call finishes in the money.
- $N(d_1)$: a probability under a different measure — the risk-neutral probability of finishing ITM using the underlying as numéraire.

The gap between $N(d_1)$ and $N(d_2)$ widens with $\sigma\sqrt{T}$ and reflects the convexity of the call payoff. For small $\sigma\sqrt{T}$, $N(d_1) \approx \Delta$; the sensitivity to $S$ tracks the probability of exercise closely. For long-dated options, the distinction matters.

*Delta and probability-of-exercise are cousins, not twins. Treat them as twins for short-dated options. For LEAPS, remember the cousinhood.*

## What Black-Scholes requires, and what it misses

The derivation relies on a specific set of assumptions. Each is violated empirically to some degree:

1. **GBM with constant $\sigma$.** Real volatility is stochastic, mean-reverting, spiky around events.
2. **Continuous hedging.** The riskless-portfolio argument requires instantaneous adjustment; practice is discrete and incurs slippage.
3. **No jumps.** Real markets gap.
4. **No transaction costs.** Continuous hedging with non-zero costs is infeasible.
5. **Known, constant risk-free rate.** Acceptable in low-rate regimes; more significant during rate-volatile periods.
6. **European exercise.** American options carry an additional early-exercise premium.

*Wrong in specific, understood ways. And yet: the market uses it.*

Options are not quoted at Black-Scholes prices directly; they are quoted at market prices that, inverted through the formula, yield an **implied volatility**. Two participants who disagree on $\sigma$ can still agree on Black-Scholes as the shared vocabulary for the disagreement. The model's inaccuracies are encoded entirely in the implied-volatility surface, covered in Part IV. The shape of that surface — term structure and skew — is the market's correction to the model.

## Vol as the price

*If $\sigma$ is the only unobservable, then asking for an option's value is asking what $\sigma$ the market assigns to the contract's life. The question is invertible. There is a unique $\sigma$ that reproduces any quote. That $\sigma$ is the implied volatility.*

Options markets are volatility markets expressed in premium units.

The trading project does not recompute option premiums from first principles in the GEX pipeline. It consumes quoted premiums, pairs them with quoted implied volatilities (or inverts premiums to recover them), and passes $\sigma$ into the Greeks formulas.

## Summary

- $\sigma$ is the only unobservable input to Black-Scholes; practitioners describe vol as the price.
- The Black-Scholes formula and the Greeks are related: $\Delta$ and $\Gamma$ are coefficients in the hedging argument that produced the PDE.
- The six Black-Scholes assumptions and the specific regime in which each breaks (constant $\sigma$, continuous hedging, no jumps, no transaction costs, known rate, European exercise).

## Implemented at

`trading/packages/gex/src/gex/greeks.py`:

- `bs_d1(spot, strike, rate, sigma, tenor_years)` at line 16 — computes $d_1$ vectorized.
- `bs_gamma` at line 31 — returns $\phi(d_1) / (S\sigma\sqrt{T})$; identical for calls and puts.
- `bs_delta_call` at line 52 — returns $N(d_1)$; used in 25-delta skew computation in [Part IV](../vol-surface/skew.md).

The module header states: "Assumes no dividends, continuous compounding, European exercise. SPX/QQQ are European-style, so this is correct for the primary use case. Single names are American — treat as approximation (error is small for gamma)." The six assumptions, encoded.

---

*A formula found in noise. A price drawn from a walk. Next, the first of the Greeks: how much does the door move.*

**Next:** [How much does the door move →](../greeks/delta.md)
