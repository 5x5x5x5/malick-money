---
title: "The geometry of mandate"
prereqs: "—"
arrives_at: "mechanical inefficiencies from forced flows"
code_ref: "pending — trading/packages/events/"
---

# The geometry of mandate

*Some flows are forced. The index adds, and the fund must buy. Not because it wants to. Because the mandate requires it. The geometry of mandate is not an opinion. It is a shape in the data, waiting to be read.*

The central argument for event-driven strategies is that the inefficiency is structural, not behavioral. Behavioral edges erode as participants learn to trade them. Mechanical edges — forced buyers and sellers acting under fixed rules — do not erode in the same way. The rules are insensitive to being traded, and the forced participants cannot opt out.

Joel Greenblatt's 1990s work on special situations framed this case for a generation of value investors. The same logic applies to systematic implementations today.

## Why mechanical inefficiencies persist

*The standard argument for why any inefficiency eventually disappears: traders notice it, participate, and additional demand eliminates the edge. This requires two things — a counterparty willing to take the inefficient side, and that counterparty having a choice.*

Mechanical inefficiencies violate the second condition. When an index adds a stock, index funds must buy. The buying is not motivated by a view on fundamental value but by a mandate. When a company spins off a subsidiary, many mutual funds must sell the spinco because their mandate restricts ownership below certain market caps.

*The forced side cannot alpha-seek. The premium, therefore, does not disappear.*

The inefficiencies are smaller than in Greenblatt's era — more capital competes — but remain non-zero and persist across market cycles in ways behavioral edges do not.

## Post-earnings announcement drift (PEAD)

The canonical academic anomaly: stocks with positive earnings surprises drift upward for weeks after the announcement; stocks with negative surprises drift downward. The persistence appears inconsistent with efficient-markets intuition.

**Standardized Unexpected Earnings (SUE)** is the standard surprise metric:

$$
\text{SUE}_i = \frac{\text{EPS}_i - \mathbb{E}[\text{EPS}_i]}{\sigma_{\text{EPS}_i}}
$$

where $\text{EPS}_i$ is the actual reported earnings per share, and $\mathbb{E}[\text{EPS}_i]$ and $\sigma_{\text{EPS}_i}$ are the consensus estimate and its dispersion.

Implementation: each earnings season, rank companies by SUE, go long the top decile, short the bottom decile, hold 60 days, rebalance. Academic studies report approximately 4-8 percentage points of annualized alpha after costs.

*The mechanism. Analysts do not update models instantly. A large SUE surprise triggers revisions over weeks. Some upgrade immediately; others wait for confirmation. The staggered repricing produces the drift. Institutional constraints (sector weights, position limits) also delay repricing.*

PEAD has decayed since the 1990s — tighter institutional execution, faster information aggregation, and PEAD-targeted funds have compressed the effect. The anomaly remains, smaller than historical papers report.

## Index rebalancing

S&P 500 reconstitution, Russell rebalancing, NASDAQ-100 quarterly adjustments follow published methodology and are announced in advance. Index funds buy additions in the days leading to the effective date and sell deletions.

**Long additions** 5 trading days before effective, close at effective + 3. Additions typically rally into the effective date as passive demand materializes, then give back some of the gain.

**Short deletions** on the same schedule. Deleted names often have fundamental weakness; the additional selling amplifies the move.

Detailed mechanics vary by index:

- **S&P 500**: committee-driven, announced approximately 5 business days before effective.
- **Russell**: rule-based, annual reconstitution in late June. Very high volume, transparent methodology.
- **NASDAQ-100**: rule-based, quarterly reweighting.

*Russell reconstitution is a single annual event with very high volume and well-studied patterns. S&P additions and deletions occur several times per year with lower volume but similar predictability.*

## Spin-offs

A parent company distributes shares of a subsidiary ("spinco") to existing shareholders. The parent goes ex-spinco on a specific date; the spinco begins trading on or shortly after.

Greenblatt documented that spinoffs often outperform their parents during the first 12-24 months following separation:

- **Neglect.** Spinoffs are often small, underfollowed, lacking institutional interest.
- **Forced selling.** Some institutional holders must sell spinoffs due to mandate restrictions.
- **Management focus.** Freed from parent-company constraints, spinoff management operates more effectively.

A systematic strategy: long spinoff and parent at separation, hold 12-24 months, rebalance as new spinoffs occur. Implementation requires reading SEC filings.

## Tax-loss reversal

U.S. individual investors sell securities at a loss in December to offset capital gains. This tax-loss selling pushes weak stocks further down in November and December. Mechanical selling stops in January, and fundamentally strong stocks that were beaten down often recover.

*The January Effect literature documents this pattern, particularly in small caps with higher fractions of tax-sensitive individual holders. The effect has compressed but has not disappeared.*

Implementation: in late November or early December, rank stocks by year-to-date return, select the worst decile, filter for positive fundamental signals, hold through February.

## Merger arbitrage

After an M&A announcement, the target trades below the deal price; the spread reflects the market's probability estimate of deal completion. Merger arbitrageurs take long positions in the target and (for stock-swap deals) short positions in the acquirer, collecting the spread on deal close.

*Merger arb is its own discipline. Deal-by-deal analysis, regulatory assessment, term negotiation. Less purely systematic than PEAD or index rebalancing.*

## What the trading project plans

`packages/events/` is spec'd but not scaffolded. Intended architecture:

- One module per event type (`pead.py`, `index_rebalance.py`, `spinoffs.py`, `tax_loss.py`).
- Shared `signal()` / `size()` interface.
- Integration with `harness` for walk-forward, cost modeling, metrics.

Unlike the GEX pipeline or microstructure, event-driven strategies are mostly buildable on retail-accessible data. The cost is in reading fundamentals, not in paying for sub-millisecond feeds.

## Capacity and crowding

Event-driven strategies have high capacity relative to purely technical strategies. PEAD alone involves approximately 500 earnings per quarter in the S&P 1500. Index rebalance flows are in the billions. Spinoff universes are smaller but the individual names are often mid- to large-cap.

*High entry cost. High capacity. A sparser competitive field. The reason most systematic firms do not do this work is also the reason it continues to work.*

## Summary

- Structural inefficiencies persist in ways behavioral ones do not; the forced side cannot respond to competitive pressure.
- The main event categories (PEAD, index rebalancing, spin-offs, tax-loss reversal) each have a specific structural source of harvestable premium.
- Event-driven strategies tend to have higher capacity and lower crowding than comparable-Sharpe technical strategies.

## Implemented at

`packages/events/` is planned. Expected structure:

- `pead.py` — SUE ranking, long top / short bottom decile, 60-day hold.
- `index_rebalance.py` — rebalance-calendar parser, long additions / short deletions around effective dates.
- `spinoffs.py` — EDGAR Form 10 parsing, long spinco + parent at separation.
- `tax_loss.py` — year-end scan of underperforming fundamentally-solid names.

---

*Some flows are forced. We do not oppose them. We stand on the other side, and collect. Next: credit speaks before equity. We listen across.*

**Next:** [Credit speaks before equity →](cross-asset.md)
