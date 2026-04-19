---
title: "Time takes, fear rises"
prereqs: "The hinge, and its curvature"
arrives_at: "the remaining first-order sensitivities — time, implied vol, rates"
code_ref: "trading/packages/gex/src/gex/greeks.py"
---

# Time takes, fear rises

*Time takes. It always takes. We can no more ask it to give back the hour than we can ask the tide to stay. Theta is the name we put on this small, daily subtraction.*

*Fear rises. Fear falls. Rates, too — though we tell ourselves they will not. These are the other sensitivities.*

Delta and gamma receive the most attention because they track the underlying. The Black-Scholes PDE has additional partial derivatives — $\Theta$ (time) and one implicit in $\sigma$ (vega) — equally important for managing a hedging book. Rho (rate sensitivity) is typically secondary for short-dated equity options but completes the set.

## Theta — time decay

$$
\Theta = \frac{\partial V}{\partial t}.
$$

Theta measures how much an option's value changes per unit of elapsed time, with other inputs held constant. For long options, $\Theta < 0$: value declines as time passes, with $S$ and $\sigma$ unchanged.

*Time value is the premium for future uncertainty. Less future, less uncertainty. An ATM call with 30 days to go carries more time value than one with 2 days. The difference is collected as theta, a day at a time.*

For a European call under Black-Scholes:

$$
\Theta_\text{call} = -\frac{S \phi(d_1) \sigma}{2\sqrt{T}} - r K e^{-rT} N(d_2).
$$

The first term is the gamma-scalping-cost term — large for ATM short-dated options. The second is a smaller rates term. The first dominates most practical positions.

Theta is conventionally reported in dollars per day. Traders describe positions by their net daily theta: a book earning \$50k per day in theta is the sum of theta across positions.

### The theta-gamma trade-off

The previous lesson showed a delta-hedged long option earns $\tfrac{1}{2}\Gamma (dS)^2$ from motion and loses $|\Theta| dt$ from time; break-even is when realized variance equals implied variance. Both $\Theta$ and $\Gamma$ peak for ATM short-dated options — the same region.

*Short options is long theta, short gamma. The quiet days accrue. Each move costs. This is the whole story of short-volatility strategies.*

The characterization of an iron condor as "selling time" is literally correct: the position captures theta while bearing gamma risk in the tails. When the underlying remains within the condor's wings, theta accrues; when it breaks through, gamma cost dominates.

## Vega — sensitivity to implied volatility

$$
\mathcal{V} = \frac{\partial V}{\partial \sigma}.
$$

Vega measures how much an option's value changes per unit change in **implied** volatility — the $\sigma$ recovered by inverting Black-Scholes against a market premium, not the realized volatility of the underlying.

For a European call or put (vega is identical):

$$
\mathcal{V} = S \phi(d_1) \sqrt{T}.
$$

Units: "dollars per 1 vol point." A vega of \$0.30 means a 1-point rise in IV (20% to 21%) changes the option's value by \$0.30.

### Where vega is large

$\phi(d_1) \sqrt{T}$ implies:

- ATM options have the highest vega.
- Long-dated options have higher vega than short-dated ones — the $\sqrt{T}$ factor grows with expiry. A 1-year option has approximately $\sqrt{365/30} \approx 3.5\times$ the vega of a 30-day option.
- Very deep ITM or OTM options have low vega.

### Vol hedging

*A delta-hedged book is not vol-hedged. If implied vol rises by a point across the board, every long option gains by its vega and every short option loses by its vega.*

A vol-hedged book has net vega near zero, typically achieved by offsetting long and short options at similar strikes and expiries. Delta neutralizes first-order moves in the underlying; vega hedging neutralizes first-order moves in implied volatility. A properly managed market-maker book controls both.

### Vega and vol-surface risk

Vega as defined assumes a parallel shift in implied volatility — the entire surface moves uniformly. In practice, IV evolves regime by regime. The short end moves differently from the long end (term-structure changes); OTM puts move differently from OTM calls (skew changes).

*A single vega number hides much. Real vol books quote vega bucketed by expiry and delta, and hedge the buckets separately.*

## Rho — sensitivity to rates

$$
\rho = \frac{\partial V}{\partial r}.
$$

For a European call:

$$
\rho_\text{call} = K T e^{-rT} N(d_2).
$$

The $T$ in the numerator means rho scales with time to expiry. For a put, rho is negative. Higher rates raise the forward price of the underlying, increasing call value and decreasing put value.

For most short-dated equity options, rho is small enough to ignore. A 1% rate move typically changes a 30-day option's price by a few cents. For long-dated options (LEAPS), rho becomes non-negligible; for rate-vol-linked products, rho is central.

*Rho rarely appears in daily options-trading discussions. It matters when rates are moving — the 2022 Fed cycle, for example — or when expiries are long.*

## Second-order Greeks

Each first-order Greek has its own sensitivity to other inputs. Most are negligible; a few are material in specific regimes.

### Charm: $\partial \Delta / \partial t$

How delta changes with time, at fixed $S$. For short-dated near-the-money options, charm is small early in the day and grows as expiry approaches. ATM deltas drift across an expiry session even with the underlying flat; this contributes to end-of-day hedging dynamics on 0DTE and expiry-Friday sessions.

### Vanna: $\partial \Delta / \partial \sigma$

How delta changes with implied volatility. When IV spikes sharply on a news event, a delta-hedged book is suddenly mis-hedged at first order; vol and delta hedging interact through vanna.

### Volga (vol-of-vol): $\partial \mathcal{V} / \partial \sigma$

How vega changes with vol. For strangles and risk reversals, volga determines whether the position is long or short vol convexity. Dispersion trading, variance swaps, and some VIX-futures strategies are volga trades.

*A heuristic. Manage delta and vega continuously. Manage gamma and theta per trade. Rho when rates move. Second-order Greeks when the surface reshapes.*

## Summary

- Short-option positions are long theta and short gamma; the decay captured is compensation for the risk taken.
- Vega is parallel-IV-shift sensitivity; vol-surface risk (term-structure and skew changes) is not captured by a single vega number.
- Rho is secondary for short-dated equity options and material for LEAPS and rate-volatile regimes.

## Implemented at

`trading/packages/gex/src/gex/greeks.py` — the current module exports `bs_d1`, `bs_gamma`, and `bs_delta_call`. Theta, vega, and rho are not computed because the GEX pipeline's regime classification operates on gamma alone. Future strategies requiring these sensitivities (a vega-hedged overlay, a theta-driven expiry play) would extend the same module using the existing Black-Scholes scaffolding.

---

*Time takes. Fear rises. Rates — we forget them, until they move. Next: what the market says about tomorrow.*

**Next:** [The market does not know tomorrow →](../vol-surface/implied-vol.md)
