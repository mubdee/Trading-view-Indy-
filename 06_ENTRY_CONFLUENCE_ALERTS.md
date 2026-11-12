# Module 06 — Entry, Confluence, Alerts and UI
(Single source for Module 06. `06_CONFLUENCE_ALERTS_UI.md` was retired in v1.2.)

## Display requirement
Show every structure/context event and entry signals. Separate visual styles:
- Structure labels/lines (BOS / CHoCH / Fake BOS / MSS)
- Liquidity pools/sweeps
- IDM candidate/swept/MSS states
- OB/FVG zones
- SFP/RB
- MTF/SMT
- Entry marker

## Entry sequence (follows Master §8 lifecycle)
Bullish:
1. `CONTEXT_FOUND`: protected low exists; HTF filter passes for the profile (or HTF Off).
2. Qualified bullish IDM is swept (`IDM_SWEPT`).
3. Bullish MSS occurs after IDM sweep with displacement (`MSS_CONFIRMED`).
4. `ZONE_WAIT`: price returns to a bullish OB or FVG **linked to that MSS** (`sourceBreakId`, Module 03).
5. Entry trigger fires inside that zone (input `Entry trigger`):
   - `Rejection Candle` (default)
   - `Internal MSS`
   - `SFP`
   - `Any enabled trigger`
6. If all required filters pass, emit one confirmed bullish entry marker at bar close, then reset to NONE with reason `ENTRY_DONE` (one entry per setup).
Bearish sequence mirrors.
If IDM module is disabled, user may select an alternate liquidity-sweep-to-MSS path. Do not bypass IDM silently while it is enabled.

### Rejection Candle (approved)
- Bullish: `low <= zone top` (touches the zone) AND `close > zone top` (closes above the proximal edge) AND `close >= low + 0.5 × (high − low)` (closes in the upper half of its range).
- Bearish: `high >= zone bottom` AND `close < zone bottom` AND `close <= high − 0.5 × (high − low)`.
- Zero-range candle never qualifies.

## Profiles
- Strict: HTF alignment required (HTF Neutral = fail) + IDM sweep + MSS/displacement + linked OB/FVG retest + entry trigger.
- Balanced: IDM sweep + MSS + linked OB or FVG retest + one enabled trigger; HTF is a filter if enabled and HTF Neutral is allowed.
- Context-only: draw all events; no entry marker.
All profile logic must be documented in the dashboard.

## Conflict handling
- HTF vs LTF conflict = label countertrend.
- Both directional triggers same candle = conflict/no entry.
- SMT unavailable = N/A, not failed.
- Invalidated/expired zone cannot qualify an entry (setup resets with `ZONE_INVALIDATED` / `ZONE_EXPIRED`).
- A confluence score is a checklist/weight, not probability or win rate.

## Alerts
`alertcondition()` for BOS, CHoCH, Fake BOS, MSS, liquidity sweep, IDM candidate/swept, SFP, new OB/FVG, mitigation, SMT, confirmed entry. One alert per event; bar-close by default.

## UI
Grouped inputs, module/timeframe toggles, style controls, object caps (500/500/500), dashboard showing timeframe selections and disabled-module warnings, bias, structure, liquidity, IDM state, SMT status (incl. compare symbol and Direct/Inverse), zone state and entry checklist. Debug mode must show rejection reasons.
