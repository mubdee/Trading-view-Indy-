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

### Added in v1.2
25. From Neutral bias: wick-only break emits Fake BOS once, bias stays Neutral, level stays active.
26. From Neutral bias: body-close break emits BOS and sets bias; a later body break of a level that earlier produced a Fake BOS emits a real BOS.
27. SFP: sweep then later body close through trigger level confirms SFP at that close.
28. SFP: wick-only break through trigger level does not confirm SFP.
29. SFP: wick break followed by a strong-volume candle confirms SFP at the volume candle's close; a normal-volume follow-up does not.
30. SFP: volume `na`/zero disables the volume path and shows N/A.
31. SFP: close beyond sweep extreme before confirmation invalidates; no confirmation inside window expires.
32. MSS breaks the most recent confirmed internal swing that existed before the IDM sweep (and Major when selected).
33. Displacement window 0: a non-displacement break candle does not confirm MSS.
34. ZONE_WAIT ignores active zones not linked to the MSS `sourceBreakId`.
35. Rejection Candle formula matches Module 06 exactly; zero-range candle rejected.
36. One entry per setup; setup resets to NONE with `ENTRY_DONE`.
37. Strict profile rejects HTF Neutral; Balanced accepts it.
38. Module TF lower than chart TF is disabled with dashboard warning.
39. PDH/PDL off on chart TF >= 1D; PWH/PWL off on chart TF >= 1W.
40. SMT auto map keeps prefix/suffix; inverse correlation compares opposite swing side.

## Repaint audit
- No unoffset `lookahead_on`.
- No negative offset to imply early signal.
- Distinguish pivot location from confirmation time.
- Intrabar preview separate from confirmed signals.
- Confirmed history stable after reload.
- Alert frequency documented.

## Acceptance package
Pine v6 source; compile status (only if actually tested in TradingView); test pass/fail/not-tested table; known limitations; exact defaults.
