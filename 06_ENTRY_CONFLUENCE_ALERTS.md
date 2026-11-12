# Module 06 — Entry, Confluence, Alerts and UI

## Display requirement
Show every structure/context event and entry signals. Separate visual styles:
- Structure labels/lines
- Liquidity pools/sweeps
- IDM candidate/swept/MSS states
- OB/FVG zones
- SFP/RB
- SMT
- Entry marker

## Entry sequence proposal
Bullish:
1. Major structure bias (chart TF) is bullish, or neutral if the selected profile allows it.
2. Qualified bullish IDM is swept.
3. Bullish MSS occurs after IDM sweep with displacement.
4. Price returns to active bullish OB or FVG zone on the chart timeframe.
5. Trigger occurs on the chart timeframe: selectable `Rejection Candle`, `Internal MSS`, `SFP`, or `Any enabled trigger`.
6. If all required filters pass, emit one confirmed bullish entry marker at bar close.
Bearish sequence mirrors.
If IDM module is disabled, user may select an alternate liquidity-sweep-to-MSS path. Do not bypass IDM silently while it is enabled.

## Profiles
- Strict: major structure bias aligned (neutral = fail) + IDM sweep + MSS/displacement + OB/FVG retest + trigger.
- Balanced: IDM sweep + MSS + either OB or FVG retest + one enabled trigger; major structure bias may be aligned or neutral.
- Context-only: draw all events; no entry marker.
All profile logic must be documented in the dashboard.

## Conflict handling
- Setup against major structure bias = label countertrend.
- Both directional triggers same candle = conflict/no entry.
- SMT unavailable = N/A, not failed.
- Invalidated/expired zone cannot qualify an entry.
- A confluence score is a checklist/weight, not probability or win rate.

## Alerts
`alertcondition()` for BOS, CHoCH, MSS, liquidity sweep, IDM candidate/swept, SFP, new OB/FVG, mitigation, SMT, confirmed entry. One alert per event; bar-close by default.

## UI
Grouped inputs, module toggles, style controls, object caps, dashboard showing chart timeframe, bias, structure, liquidity, IDM state, SMT status, zone state and entry checklist. Debug mode must show rejection reasons.
