# Sniper Flow Profile (Pine Script v6)

Intraday overlay indicator for TradingView that combines a **volume profile**, a **time profile (TPO)**, **tick flow** and **delta flow** to spot trend initiation and give retest ("sniper") entries.

File: [`SniperFlowProfile.pine`](SniperFlowProfile.pine)

## Install
1. TradingView → Pine Editor → New → paste the file → **Add to chart**.
2. Use an **intraday** chart (1m–15m works best, 5m is a good default).
3. Optional: for tick-like precision on a 1m chart, turn off *Auto intrabar timeframe* and set `1S` / `5S` (needs a plan with seconds data).

## What's on the chart
| Element | Meaning |
|---|---|
| Right-side histogram | Session profile built from intrabar data. Modes: Volume (buy/sell split), Delta, or TPO (time). POC row in yellow, value-area rows brighter. |
| Yellow line / blue band | Developing POC and value area (VAH/VAL). |
| Cyan dots | TPO POC: the price with the most time spent. |
| Faded lines | Prior session POC / VAH / VAL. |
| Magenta | Session VWAP ±1σ. |
| Orange | Initial Balance high/low. |
| `INIT ▲/▼ nn` | Trend initiation, with its 0–100 score. Hover to see each check. |
| `🎯 LONG / SHORT` | Sniper entry with entry, stop, T1 and T2 lines. Hover for details. |
| ◆ | Absorption: heavy volume and small range, with the aggressor failing to move price. |
| `Δdiv` | CVD divergence at a new session high/low (exhaustion warning). |

## Logic
**Order flow per bar** (from lower-timeframe intrabars):
- *Delta*: buy volume minus sell volume. Classified by tick rule (default), candle body, or close location.
- *Tick Δ*: upticks minus downticks across the intrabar prints. *Tick activity z* measures a surge in the number of prints.
- *CVD*: delta summed over the session. *RVOL*: bar volume divided by its average.

**Trend initiation** fires when the close breaks a balance edge (developing VAH/VAL, IB high/low, or prior VAH/VAL), the score is at least the minimum, delta agrees with the direction, and price is on the right side of VWAP. Score weights:

| Check | Points |
|---|---|
| Beyond value | 25 |
| VWAP side | 10 |
| RVOL | 15 |
| Delta % | 15 |
| Delta momentum | 10 |
| Tick Δ | 10 |
| Tick surge | 5 |
| IB break | 10 |

The trend state resets when price is accepted back through both the POC and VWAP, and at the start of each session.

**Sniper entry**: within *N* bars of an initiation, price pulls back into the broken edge, VWAP, dPOC or dVAH/dVAL (within an ATR-based tolerance) and closes back in the trend direction. The bar must also show buyers stepping back in (positive delta) or sellers being absorbed (negative delta but a close near the high). Shorts mirror this. The stop goes beyond the level plus an ATR buffer. Targets are set at R multiples, and the stop can move to break-even after T1.

The dashboard shows the live state, scores, flow readings, the levels, the active trade and the chart's own hit rate.

## Alerts
Bull/Bear initiation, Sniper LONG/SHORT and CVD divergence are available through `alertcondition`. Entry and initiation also send detailed `alert()` messages (choose *Any alert() function call*).

## Limits
- TradingView does not expose raw ticks to Pine. "Tick" values come from lower-timeframe prints, so the finer the intrabar timeframe, the closer they get to real ticks.
- Intrabar history is limited. Older bars fall back to chart-bar data, and the dashboard shows when this happens.
- Signals are confirmed on bar close and do not repaint. The developing levels update live, as any developing profile does.
- This is a decision-support tool, not financial advice. Forward-test it on your instrument before trading real money.
