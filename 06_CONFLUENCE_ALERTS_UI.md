# Module 06 — Confluence, Alerts and UI

Profiles:
- Strict: HTF bias + liquidity sweep + MSS/displacement + OB/FVG retest + LTF trigger.
- Balanced: structure + one liquidity event + one zone + one confirmation.
- Context-only: draw events; no entry marker.

Score is a checklist/weight, not probability or win rate. Show passed/failed/not available modules. If HTF and LTF conflict, label countertrend. If both directions trigger same bar, conflict/no signal. If SMT data unavailable, show N/A. Disable IDM until its definition is approved.

Alerts via `alertcondition()` for BOS, CHoCH, MSS, sweeps, SFP, new OB/FVG, mitigation, SMT and confirmed setups. Alerts only once per event and bar-close by default.

UI: grouped inputs, module toggles, style controls, max boxes/labels/lines, dashboard for HTF bias/structure/liquidity/SMT/IDM/checklist, debug mode with rejection reasons.
