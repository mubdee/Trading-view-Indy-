# Module 05 — SMT (single timeframe)

## Timeframe
Chart TF only. Multi-timeframe context (HTF bias, Setup TF, Entry TF) has been removed from the specification in v1.2.

## SMT
User-selectable comparison symbol, requested on the chart timeframe (`request.security(symbol, timeframe.period, ...)`, lookahead off). SMT is relative non-confirmation, not standalone entry.
Bullish proposal: chart symbol makes a lower low versus prior confirmed swing low while comparison symbol does not make a corresponding lower low within timestamp tolerance. Bearish mirrored at swing highs.
Require confirmed pivots on both symbols; match timestamps within tolerance. If comparison data is `na`, show unavailable and do not count as failed confirmation.

Pine concerns: request limits, suffixes, different sessions, missing bars, pivot alignment and no lookahead.
