# Module 07 — Tests and Repaint Audit

## Build status
| Item | Status |
|---|---|
| Syntax parse (open-source `pynescript` parser) | ✅ Parsed |
| Compile in TradingView Pine Editor | ⬜ **Not tested** — must be done by the user |
| Visual / bar-replay tests below | ⬜ Not tested |

## Bar-by-bar tests (all ⬜ not tested)
1. Pivot is not used before its R right-side bars close.
2. A break emits once per level.
3. Wick-only break follows the selected mode once a bias exists.
4. From neutral: wick-only = fake BOS (bias stays neutral); body close = real BOS.
5. Both-side break is ambiguous, no directional event.
6. Sweep + reclaim emits once; close accepted beyond = breakout, not sweep.
7. IDM candidate only after internal pivot confirmation and only on the valid side of the protected swing.
8. IDM sweep before MSS qualifies; MSS before IDM does not.
9. IDM sweep and MSS on the same candle is rejected by default.
10. MSS break without displacement resets the setup.
11. Protected swing break / change cancels IDM.
12. OB Hybrid boundaries match `[low, open]` / `[open, high]`.
13. Doji source candle skips the OB by default.
14. FVG formulas correct on closed candle 3.
15. OB uses the nearest opposite candle within the lookback.
16. Zone lifecycle (touch, mitigation, invalidation, expiry) persists correctly; no retest on the creation candle.
17. SFP needs a body close beyond the sweep candle extreme; wick-only needs the high-volume next candle.
18. RB forms only after SFP confirmation, with the selected boundary.
19. Every module reads chart TF only; the only `request.security()` is SMT on `timeframe.period`.
20. PDH/PDL/PWH/PWL use only completed periods; auto-disabled on chart TF ≥ D / ≥ W.
21. SMT matches confirmed pivots by time; missing data → N/A.
22. Entry fires once per setup; long + short on one candle = conflict.
23. Alerts do not duplicate on realtime ticks.
24. Reload and Bar Replay produce the same confirmed history.
25. Different tick sizes / digits / symbol suffixes work.
26. Object limits respected; no signal before ATR is available.

## Repaint audit (code review)
- All state changes are inside `if barstate.isconfirmed` → realtime ticks never change state.
- No `lookahead_on`; no negative `offset`.
- Pivot labels sit on the pivot candle but are created on the confirmation candle (documented in tooltips).
- SFP / SMT labels: SFP on the confirmation candle; SMT on the chart pivot candle, created only after both pivots are confirmed.
- Alerts are flags from closed candles → set alert frequency to "Once per bar close".

## Known limitations
- Drawing objects older than ~5000 bars are clamped to 5000 bars back.
- Volume rule depends on the broker's (tick) volume.
- Only the newest IDM candidate per side is tracked (older ones are dropped, visible in debug).
