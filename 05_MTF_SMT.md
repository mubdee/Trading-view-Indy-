# Module 05 — MTF + SMT

## User-adjustable timeframe architecture
Provide independent inputs for:
- Structure TF
- Liquidity TF
- OB/FVG TF
- SFP/RB TF
- IDM TF
- HTF context TF
- Setup TF
- Entry TF
Each input supports Chart TF / Off where appropriate / selectable Pine timeframe. No fixed timeframe stack.

## MTF context
Stable default uses prior completed HTF bar/structure for confirmed bias. Never present incomplete HTF state as confirmed. Document exact `request.security()` gaps/lookahead behavior and update timing. If a selected TF is lower than chart TF, use `request.security_lower_tf()` with explicit array handling where feasible, otherwise disable that selection with a visible warning. Do not silently treat lower-timeframe data as complete.

Bias options: HTF structure; HTF close vs selected swing; neutral on insufficient/contradictory data.

## SMT
User-selectable comparison symbol. SMT is relative non-confirmation, not standalone entry.
Bullish proposal: chart symbol makes a lower low versus prior confirmed swing low while comparison symbol does not make a corresponding lower low within timestamp tolerance. Bearish mirrored at swing highs.
Require confirmed pivots on both symbols; match timestamps within tolerance. If comparison data is `na`, show unavailable and do not count as failed confirmation.

Pine concerns: request limits, suffixes, different sessions, missing bars, pivot alignment and no lookahead.
