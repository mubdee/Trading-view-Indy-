# Module 06 — Entry, Confluence, Alerts and UI

## Display requirement
Show every structure/context event and entry signals. Separate visual styles:
- Structure labels/lines
- Liquidity pools/sweeps
- IDM candidate/swept/MSS states
- OB/FVG zones
- SFP/RB
- MTF/SMT
- Entry marker

## Entry sequence proposal
Bullish:
1. Optional HTF context is bullish or neutral according to selected profile.
2. Qualified bullish IDM is swept.
3. Bullish MSS occurs after IDM sweep with displacement.
4. Price returns to active bullish OB or FVG zone on configured retest timeframe.
5. LTF trigger occurs: selectable `Rejection Candle`, `Internal MSS`, `SFP`, or `Any enabled trigger`.
6. If all required filters pass, emit one confirmed bullish entry marker at bar close.
Bearish sequence mirrors.
If IDM module is disabled, user may select an alternate liquidity-sweep-to-MSS path. Do not bypass IDM silently while it is enabled.

## Profiles
- Strict: HTF alignment + IDM sweep + MSS/displacement + OB/FVG retest + LTF trigger.
- Balanced: IDM sweep + MSS + either OB or FVG retest + one enabled trigger; HTF is a filter if enabled.
- Context-only: draw all events; no entry marker.
All profile logic must be documented in the dashboard.

## Conflict handling
- HTF vs LTF conflict = label countertrend.
- Both directional triggers same candle = conflict/no entry.
- SMT unavailable = N/A, not failed.
- Invalidated/expired zone cannot qualify an entry.
- A confluence score is a checklist/weight, not probability or win rate.

## Alerts
`alertcondition()` for BOS, CHoCH, MSS, liquidity sweep, IDM candidate/swept, SFP, new OB/FVG, mitigation, SMT, confirmed entry. One alert per event; bar-close by default.

## UI
Grouped inputs, module/timeframe toggles, style controls, object caps, dashboard showing timeframe selections, bias, structure, liquidity, IDM state, SMT status, zone state and entry checklist. Debug mode must show rejection reasons.
