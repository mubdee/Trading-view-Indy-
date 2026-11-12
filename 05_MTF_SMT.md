# Module 05 — MTF + SMT

## User-adjustable timeframe architecture
Provide independent inputs for:
- Structure TF
- Liquidity TF
- OB/FVG TF
- SFP/RB TF
- IDM TF
- HTF context TF (default Off)
- Setup TF (default Off)
- Entry TF
Each input supports Chart TF / Off where appropriate / selectable Pine timeframe. No fixed timeframe stack. All defaults = Chart TF except HTF context and Setup = Off.

## Phase 1 timeframe rule (approved)
- Supported: Chart TF, or any TF >= chart TF.
- A TF lower than chart TF disables that module and shows a dashboard warning. `request.security_lower_tf()` support is Phase 2.

## MTF context
Stable default uses prior completed HTF bar/structure for confirmed bias. Never present incomplete HTF state as confirmed. Use the non-repainting pattern `request.security(sym, tf, expr[1], lookahead = barmerge.lookahead_on)` and document update timing (the HTF value changes on the first chart bar after the HTF bar closes).

Bias options: HTF structure; HTF close vs selected swing; neutral on insufficient/contradictory data.
Profile use: Strict requires HTF bias = setup direction (Neutral fails). Balanced allows HTF Neutral. HTF Off skips the check.

## SMT
SMT is relative non-confirmation, not standalone entry.

### Comparison symbol (approved: Claude picks suitable pairs)
Input `SMT symbol mode`: `Auto map` (default) / `Manual`.

Auto map (by chart base+quote, ignoring prefix and broker suffix):
| Chart | Compare | Correlation |
|---|---|---|
| XAUUSD | XAGUSD | Direct |
| EURUSD | GBPUSD | Direct |
| GBPUSD | EURUSD | Direct |
| AUDUSD | NZDUSD | Direct |
| NZDUSD | AUDUSD | Direct |
| USDCHF | EURUSD | Inverse |
| USDJPY | USDCHF | Direct |
| Anything else | none → SMT shows N/A until set manually | — |

Ticker construction in Auto map: keep the chart's exchange prefix and broker suffix (e.g. chart `ICMARKETS:XAUUSD.a` → `ICMARKETS:XAGUSD.a`). If that request returns no data, dashboard shows "SMT: N/A (compare symbol not found)" and suggests Manual mode.

Manual mode: input `Comparison symbol` (symbol picker) + input `Inverse correlation` (default off).

### Detection
- Direct, bullish: chart makes a lower low versus its prior confirmed swing low while the comparison symbol does not make a lower low at a matching swing low within the timestamp tolerance (default 2 bars). Bearish mirrored at swing highs.
- Inverse, bullish: chart makes a lower low while the comparison symbol does **not** make a higher high at a matching swing high within tolerance. Bearish mirrored.
- Require confirmed pivots on both symbols (same pivot L/R); match by timestamp within tolerance.
- If comparison data is `na`, show unavailable and do not count as failed confirmation.

Pine concerns: request limits, suffixes, different sessions, missing bars, pivot alignment and no lookahead.
