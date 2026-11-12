# Module 07 — Test and Repaint Audit

## Required bar-by-bar tests
1. Pivot not confirmed until right bars close.
2. Break emits once per level.
3. Wick-only break follows selected mode.
4. Both-side break is ambiguous.
5. Sweep + reclaim emits once.
6. Close accepted beyond liquidity = breakout, not sweep.
7. IDM candidate only after internal pivot confirmation and only on valid side of protected swing.
8. IDM sweep before MSS qualifies; MSS before IDM does not.
9. IDM sweep and MSS same candle is rejected by default.
10. Protected swing invalidation cancels IDM.
11. OB Hybrid boundaries match approved formula.
12. Doji source candle is skipped by default.
13. FVG formulas correct on closed candle.
14. OB selects nearest opposite candle in lookback.
15. Zone lifecycle persists correctly.
16. SFP requires close reclaim.
17. RB forms only after rejection confirmation.
18. MTF uses completed HTF data.
19. SMT matches confirmed pivots by time; missing data handled.
20. Alerts do not duplicate on realtime updates.
21. Reload and Bar Replay produce same confirmed historical events.
22. Different tick sizes/digits/symbol suffixes supported.
23. Lower-TF selection is handled safely or rejected with warning.
24. Object limits respected; no signal on insufficient data.

## Repaint audit
- No unoffset `lookahead_on`.
- No negative offset to imply early signal.
- Distinguish pivot location from confirmation time.
- Intrabar preview separate from confirmed signals.
- Confirmed history stable after reload.
- Alert frequency documented.

## Acceptance package
Pine v6 source; compile status (only if actually tested in TradingView); test pass/fail/not-tested table; known limitations; exact defaults.
