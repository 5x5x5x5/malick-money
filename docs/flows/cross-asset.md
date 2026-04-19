---
title: "Credit speaks before equity"
prereqs: "How widely does a thing breathe"
arrives_at: "multi-asset confirmation as regime filter"
code_ref: "pending — trading/packages/macro/"
---

# Credit speaks before equity

*Credit speaks before equity. The dollar before the distant. The curve before the recession. We listen across.*

Equities do not trade in isolation. Credit spreads widen before equity risk premiums reprice. The dollar moves before emerging-markets equities respond. The yield curve inverts before recessions. The copper-to-gold ratio signals growth versus risk-off conditions years before equity markets reflect them.

Each of these relationships is a lead-lag that single-chart technical analysis misses. A strategy that incorporates cross-asset signals before taking equity positions operates with more information than one that does not.

## The core concept

Asset classes are connected by economic mechanisms, not only by sentiment. When these mechanisms activate, related assets move in coordinated ways. The timing is not simultaneous: different participants operate on different time scales, and different microstructures respond at different speeds. The resulting lead-lag relationships form the basis of cross-asset signals.

Three commonly cited patterns:

1. **Credit spreads lead equity.** HYG/LQD ratios widen (high-yield underperforming investment-grade) before equity risk premiums reprice. Credit traders are more forward-looking about corporate default risk than equity traders, who often wait for earnings releases.
2. **Yield curve leads recession.** The 2s10s spread has historically inverted approximately 12 months before recessions. Equity markets typically peak closer to recession onset, so the curve leads equity tops.
3. **Dollar leads emerging markets.** A strengthening dollar compresses EM equities through currency translation and dollar-denominated debt pressure.

*Not physical laws. Each pattern has regimes in which it breaks. 2s10s was arguably disrupted during 2019-2020 by QE's effect on term premia. A framework should be robust to regime breaks, not dependent on any single relationship.*

## The standard cross-asset universe

For an equity-focused macro dashboard:

| Series | What it tells you |
|--------|-------------------|
| HYG / LQD | Credit spread (high-yield vs investment-grade bond ETFs) |
| DXY | U.S. dollar strength |
| 2s10s spread | Yield curve slope |
| 5y5y breakevens | Long-term inflation expectations |
| TLT | Long-duration Treasuries (rate sensitivity) |
| GLD / HG=F | Gold / copper (risk vs growth) |
| USD/JPY | Yen pair — risk-on/off proxy |
| VIX | Equity IV level |
| VIX / VIX3M | Equity IV term structure slope |

Each contributes a different view. Credit spreads and curve slope reflect credit risk and growth expectations. DXY, JPY, and copper-gold reflect global risk tolerance. Volatility is a separate regime layer.

*No single series is dispositive. The value of a dashboard lies in the joint signal, more stable than any individual axis.*

## Rolling z-scores as a standard form

Raw levels of these series are not comparable across each other or across time. The dollar index in the 1990s represents different conditions than the dollar index in the 2020s. The standard preprocessing is a rolling z-score:

$$
z_t = \frac{x_t - \bar x_\text{lookback}}{\sigma_{x, \text{lookback}}}.
$$

Typical lookback windows are 63 days (one quarter) for short-term regime and 252 days (one year) for longer-term regime.

*Z-scores make signals comparable across series. A $z = +2$ on credit spreads and $z = +2$ on DXY convey equivalent regime information despite differing absolute levels.*

## Divergence detectors

The most commonly cited cross-asset signal is equity-credit divergence:

- Equity is making a new high or is near one.
- Credit spreads are not making a new low or are not near one.

The joint condition indicates that equity is pricing continued risk-on while credit is beginning to reprice risk. Historically, such divergences have preceded equity corrections by weeks to months. The 2007-08, late-2018, and early-2020 corrections all followed clear prior credit-equity divergences.

A mechanical implementation:

1. Compute rolling 63-day maxima for equity level and for the negative of credit spreads.
2. Flag when equity is within 5% of its rolling high and credit is more than 2 standard deviations from its rolling low.
3. Use the flag as a filter on new equity long trades — vetoing or downweighting entries.

## The four-state regime classifier

One summary of cross-asset state uses a 2×2 matrix:

|                | Risk-on          | Risk-off         |
|----------------|------------------|------------------|
| **Inflating**  | Reflation        | Stagflation      |
| **Disinflating** | Goldilocks     | Deflationary bust |

The four states are identified by combining:

- **Risk-on/off**: equity direction, credit spread direction, DXY direction.
- **Inflating/disinflating**: 5y5y breakevens direction, commodity complex direction.

Each implies different trade filters:

- **Goldilocks** (risk-on, disinflating): favorable for equity momentum and long growth.
- **Reflation** (risk-on, inflating): favorable for commodities, real assets, value over growth.
- **Stagflation** (risk-off, inflating): unfavorable for both stocks and bonds; gold and commodities may still perform.
- **Deflationary bust** (risk-off, disinflating): favorable for long duration and shorts on risk assets.

*An approximate classification. An organizing principle. "The regime is Goldilocks" is more useful for position sizing than "the market is neutral on the 12-month horizon."*

## Cross-asset as filter, not primary

Cross-asset signals serve poorly as primaries but well as filters. Lead-lags are real but noisy, with false signals and regime-dependent timing. A strategy based purely on "credit-equity divergence fires → short equity" would underperform; the timing of corrections ranges from weeks to months, and other factors typically dominate.

*Use a conviction-based primary (technical, fundamental, microstructure). Require cross-asset confirmation before execution. "Take the primary signal unless cross-asset regime is risk-off" is preferable to "take the primary only when cross-asset is risk-on."*

A reasonable veto threshold: require at least 2 of 3 cross-asset confirming prints before equity entries.

## What the trading project plans

`packages/macro/` is spec'd but not scaffolded. When built, expected:

- Data sources: FRED API for macro time series, yfinance for liquid ETFs.
- `MacroDashboard.snapshot(as_of_date)` returning current levels and z-scores.
- `detect_divergence(series_a, series_b, lookback)` for pair-wise divergences.
- `classify_4state(dashboard)` for the regime grid.
- `veto_filter(primary_signal, dashboard, min_confirming=2)` for the pre-execution gate.
- Daily EOD refresh via cron or GitHub Actions.

*Data freshness matters less than for microstructure; cross-asset signals operate in days to weeks, so EOD refresh is sufficient. Of the three pending packages (microstructure, events, macro), macro has the lowest barrier to build.*

## Summary

- Lead-lag exists between asset classes because different participants operate on different information sets and time scales.
- Rolling z-scores make cross-asset signals comparable across series and across decades; absolute levels reflect regime, while z-scores reflect surprise within regime.
- Cross-asset signals work better as filters than as primaries: timing is too variable to drive entries, but regime information is reliable enough to veto poor setups.

## Implemented at

`packages/macro/` is planned. When scaffolded:

- Daily EOD refresh of the cross-asset universe.
- Rolling z-score computation at 63-day and 252-day lookbacks.
- Divergence-detector functions for major pairs.
- 4-state regime classifier.
- Integration point: a `veto_filter` consumable by other strategies.

---

*Credit speaks before equity. The dollar before the distant. The curve before the recession. We listen across.*

*We have walked from a number that moved through the Greeks, the surface, the engine, the measure, the lesson, and the flows. We came with a question — did it work. The answer, like every answer, arrives in pieces.*

*The trading project waits, at the end of every lesson. Open the code. Read the docstrings. Run the sweep. The math is unchanged. The voice, only, was new.*

[← Back to the Prologue](../index.md)
