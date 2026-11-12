# Module 06 — Entry, Profiles, Alerts, Dashboard

Timeframe: chart TF only.

## Entry sequence (bullish; bearish mirrors)
1. Context: protected low exists; Strict also needs major bias = bullish.
2. IDM swept (or liquidity sweep when IDM is off — `02`).
3. Bullish MSS with displacement after the sweep.
4. Price returns to the setup's own OB or FVG (linked by MSS ID). Zone must still be valid.
5. Trigger on a closed candle:
   - **Rejection Candle** (default): candle overlaps the zone, closes **above the zone top** (proximal) and in the **upper half** of its range.
   - **Internal MSS**: internal bullish BOS/CHoCH while the zone has been touched.
   - **SFP**: bullish SFP confirmed while the zone has been touched.
   - **Any enabled trigger**: any of the above.
6. All required filters pass → one `LONG` marker at bar close, tooltip = checklist. Setup resets with `ENTRY_DONE`.

## Profiles
| Profile | Bias rule | Entry marker |
|---|---|---|
| Strict | Major bias must equal direction; neutral fails | yes |
| Balanced (default) | Any bias; against major bias → label `(CT)` countertrend | yes |
| Context-only | — | no (all context still drawn; reason `CONTEXT_ONLY`) |
The score/checklist is not a probability or win rate.

## Conflict handling
- Long and short trigger on one candle → `CONFLICT`, no entry, both setups keep waiting.
- Invalidated / expired zone cannot qualify. All linked zones gone → reset `ZONES_INVALIDATED`.
- SMT unavailable = N/A, not failed.

## Reason codes
`PROTECTED_BROKEN`, `EXPIRED_MAX_BARS`, `PROTECTED_CHANGED`, `IDM_ACCEPTED_BEYOND`, `RECLAIM_TIMEOUT`, `NO_MSS_LEVEL`, `BREAK_NO_DISPLACEMENT`, `SAME_BAR_AMBIGUOUS`, `NO_ZONE_FROM_MSS`, `ZONES_INVALIDATED`, `ZONE_WAIT_EXPIRED`, `BIAS_NOT_ALIGNED`, `CONTEXT_ONLY`, `CONFLICT`, `ENTRY_DONE`. Debug mode prints them on the chart; the dashboard always shows the last one per side.

## Alerts (`alertcondition`, closed-bar flags → one per event)
Bullish/Bearish BOS, Bullish/Bearish CHoCH, Bullish/Bearish MSS, SSL/BSL sweep, IDM candidate, IDM swept, Bullish/Bearish SFP, New OB, New FVG, Zone mitigated, Bullish/Bearish SMT, Confirmed LONG, Confirmed SHORT.
Recommended alert frequency in TradingView: "Once per bar close".

## Dashboard
Chart TF · profile + trigger · major / internal bias · protected L / H · last liquidity event · bull / bear setup state + reason · active OB / FVG / RB counts · SMT status · PD/PW availability · last entry.
