# Module 05 — SMT Divergence

Timeframe: chart TF only. The comparison symbol is requested on `timeframe.period`, lookahead off, `ignore_invalid_symbol = true`.

## Comparison symbol
- Default: `OANDA:XAGUSD` (silver) for XAUUSD — direct correlation.
- Forex / other: user selects the symbol in the input (e.g. EURUSD ↔ GBPUSD).
- Inverse correlation input (e.g. DXY vs gold): compares the opposite extreme of the comparison symbol.

## Rule (major pivots, same L/R on both symbols)
- **Bullish SMT**: chart confirms a major **lower low** (below previous major low − tolerance) while the comparison symbol's matching pivot low does **not** make a lower low (direct) / its matching pivot high does not make a higher high (inverse).
- **Bearish SMT**: mirrors at highs.
- Matching: the comparison pivot time must be within ± tolerance bars (default 2) of the chart pivot time. The check waits up to `tolerance` bars after the chart pivot is confirmed.
- No matching pivot / comparison data `na` → **N/A**, never counted as failed confirmation.
- Same symbol as chart → N/A.

SMT is context only. It is shown in the dashboard and in the entry checklist; it is **not** a required entry condition in any profile.
