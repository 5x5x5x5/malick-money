---
title: "A tape of sparks"
prereqs: "A promise you need not keep; How widely does a thing breathe"
arrives_at: "the signal layer below OHLCV"
code_ref: "pending — trading/packages/microstructure/"
---

# A tape of sparks

*Below the minute. Below the second. A tape of sparks. Each one a meeting. Each one a memory. The price does not glide. It ticks.*

The curriculum to this point has assumed the analysis operates on bars — daily, hourly, or 5-minute. Microstructure is the activity within a bar. At this scale, prices move in discrete ticks rather than continuously. Each tick represents a transaction between a buyer and a seller at a specific price in a specific order book, potentially triggering a chain of further transactions.

*The view at this scale differs from the bar chart. We must learn to see it, if we want to trade it.*

Microstructure is the domain of proprietary trading firms. Retail participation is rare because the required data is expensive, latency matters, and backtest fidelity is difficult.

## The order book

At any moment, a security's market state is summarized by the order book — a list of bids (buy orders at specific prices) and asks (sell orders at specific prices).

```
         size   price
ask 3:   2000   101.05
ask 2:   3500   101.03
ask 1:   1200   101.01   ← best ask (lowest sell price)
--------------- spread = $0.03
bid 1:    800   100.98   ← best bid (highest buy price)
bid 2:   1800   100.96
bid 3:   2500   100.94
```

The best bid and best ask (Level 1) are what most quote feeds report. The depth behind them (Level 2) contains additional information. Which side has more size? Where do orders thin out?

*A 100,000-share buy order submitted into a book with only 50,000 of ask depth traverses multiple price levels, producing a spike visible on the bar chart as "the close ran up." Depth information identifies this in advance.*

## Trade classification — Lee-Ready

When a trade prints at price $P$, which side was the aggressor — the buyer or the seller? Equivalently, was the bid hit or the ask lifted? The raw tick feed does not carry this tag.

The Lee-Ready algorithm infers direction from price location and timing:

1. If the trade price is above the midpoint, classify as a buy.
2. If below the midpoint, classify as a sell.
3. If at the midpoint exactly, apply the tick rule: compare with the previous trade price. Higher previous trade implies buy; lower implies sell. When equal, carry forward the previous classification.

Lee-Ready (1991) is the standard classification method. Accuracy is high for active stocks (above 90% for liquid names) and degrades for illiquid names or fast markets where quotes lag trades.

## Volume footprint

Given trade-direction classifications, aggregate volume at each price level into bid-volume and ask-volume components:

```
price     bid-vol    ask-vol
101.00     100        500      ← ask-heavy at this level
100.98     200        450
100.96     300        250
100.94     450        200
100.92     500        150      ← bid-heavy at this level
```

The pattern across a bar shows where buying and selling were aggressive.

*A bar that closes up with bid-heavy volume at the bottom and ask-heavy at the top tells a different story than a bar that closes up with ask-heavy throughout. The first suggests absorbed short-covering. The second suggests continued demand. Reading them well is a skill.*

## Cumulative delta

Summing signed volume (+ask_volume, −bid_volume) across a period produces cumulative delta. Rising cumulative delta indicates more aggressive buying than selling.

The useful signal appears when cumulative delta diverges from price. If price makes a new intraday high but cumulative delta does not, the move is thin — more short-covering than genuine demand — and is likely to fade.

*Divergence is among the most-cited microstructure signals. Its reliability varies by regime: it tends to work in ranging or late-trend markets and fails in early-trend or sudden-regime-change breakouts. Not free. A meaningful input to a broader signal set.*

## Market Profile (TPO)

Time-price opportunity charts show, for a given session, the set of price levels traded during each 30-minute period. Over a full session, each price level receives a count of the 30-minute slots during which it was traded.

From the count distribution:

- **Point of Control (POC)**: the price level traded for the most time.
- **Value Area High (VAH)** and **Low (VAL)**: the upper and lower bounds of the band (typically 70%) around the POC.
- **Single prints**: price levels traded in only one 30-minute period, often at session extremes.

*Market Profile originates in Peter Steidlmayer's 1980s work. The central intuition: prices that traded heavily in the past tend to act as magnets in the future. The prior session's POC attracts today. Single-print extremes fill quickly when revisited.*

## Sources of durable edge

Microstructure strategies retain durable edge for three reasons:

1. **Data costs.** Clean L2 tick data is expensive (Databento starts at thousands per month for SPX; direct exchange feeds are five to six figures annually). Most retail participants are excluded.
2. **Latency.** A signal from the current order book is available only to infrastructure that can produce it quickly. Co-location is necessary for the sharpest strategies.
3. **Simulation difficulty.** Backtesting requires simulating order interaction with the book — queue position, fill probability, slippage, adverse selection. Midpoint-fill simulators systematically overstate quality.

*The first two barriers shrink over time. The third persists.*

## What the trading project plans

`packages/microstructure/` is spec'd but not scaffolded. The rough outline:

- Data source: Databento MBP-10 or IBKR historical via `ib_insync`.
- Lee-Ready tick classification.
- Volume footprint in N-volume bars.
- Cumulative delta, stacked-imbalance detector.
- TPO profile: VAH, VAL, POC, single prints.
- Signals: delta/price divergence, POC rejection.
- Fill simulator using L2 snapshots.

## Retail-accessible microstructure

Lower-cost data sources provide enough microstructure information for some strategies:

- **1-minute OHLCV with volume**: Yahoo, IEX, Alpaca free tier. Too coarse for full microstructure work, but volume-at-price can be approximated.
- **Level 1 tick data**: tick-level trades without book depth. Lee-Ready is usable via tick-rule fallback.
- **IBKR delayed L2**: 15-minute delayed Level 2 for most U.S. equities, free. Insufficient for live trading but useful for building and sanity-checking strategy logic.

*Strategies built on these substrates will not compete with proprietary firms. They can still exhibit meaningful alpha patterns in less-liquid markets where the barrier against institutional competition is smaller.*

## Summary

- Microstructure edge is durable despite public awareness: data cost, latency requirements, and simulation fidelity are structural barriers.
- The Lee-Ready algorithm is the standard trade-classification method and fails in illiquid names and fast markets with lagging quotes.
- Volume footprint (price-level bid vs ask volume) and TPO (price-level time counts) are different aggregations of the same underlying tick data.

## Implemented at

`packages/microstructure/` is planned, not yet scaffolded. When built, the expected components:

- A `ChainSource`-like abstraction over L2 feeds (Databento, IBKR).
- `classify_lee_ready(ticks)` producing buy/sell tags.
- `volume_footprint(ticks, bar_rule="volume", bar_size=10000)`.
- `tpo_profile(ticks)` producing VAH, VAL, POC, single prints.
- `L2FillSimulator` for honest backtests.

---

*A tape of sparks, each one a meeting. Next: flows that do not choose, that cannot opt out.*

**Next:** [The geometry of mandate →](event-driven.md)
