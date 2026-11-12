# Module 02 — Liquidity, Sweeps and IDM

## Timeframe
Liquidity source timeframe is independently selectable: Chart TF or a timeframe >= chart TF (Phase 1). Prior-day/week levels use their native daily/weekly reference and must not be fabricated from partial sessions. PDH/PDL auto-off when chart TF >= 1D; PWH/PWL auto-off when chart TF >= 1W (dashboard notice).

## Liquidity sources
Independent toggles: confirmed major/internal swing highs/lows; EQH/EQL; previous day high/low; previous week high/low; optional session highs/lows.

## Pool creation
1. Confirmed swing high creates candidate BSL; swing low creates candidate SSL.
2. Merge nearby highs/lows into equal-level pools within configurable tolerance.
3. Store price, source, timestamps, touch count, active/taken state.
4. Expire after `Liquidity pool max age` (default 500 bars) or consumption.

## Sweep (closed-bar default)
Bullish reversal context: candle low penetrates active SSL by sweep buffer (default 0) and closes back above the liquidity level. Bearish: high penetrates BSL and closes back below. Store level ID, penetration, reclaim distance and timestamp. A close accepted beyond level is breakout/acceptance, not a sweep.
A sweep is the parent event of a possible SFP; SFP confirmation rules are in Module 04 and store the parent sweep ID.

## IDM approved definition
IDM = confirmed internal swing liquidity swept before MSS.

Bullish sequence:
1. Confirm an internal swing low using the configured internal pivot L/R.
2. It must be above the active protected major swing low (Master §7.1). If no protected major low exists, do not create a qualified IDM candidate.
3. Mark candidate only at pivot confirmation time; do not backdate availability.
4. Price low must trade below IDM level by at least sweep buffer.
5. Default reclaim: the sweep candle closes back above IDM. Optional input may allow reclaim within N subsequent bars, disabled by default.
6. Mark `IDM_SWEPT` once; store level ID and sweep timestamp.
7. Wait for bullish MSS after the sweep (Master §7.3): break of the most recent confirmed internal swing high that existed before the sweep (input: Internal / Major), selected break mode, displacement within the displacement window (default 0 = break candle itself).
8. If price closes below protected major swing low before MSS, invalidate candidate/setup.
9. Expire candidate after `IDM candidate max bars` (default 100) or when protected major swing low is replaced/invalidated.

Bearish sequence mirrors:
- internal swing high below protected major high;
- wick trades above IDM and reclaim close below;
- bearish MSS after sweep (break of most recent confirmed internal swing low before the sweep);
- close above protected major high invalidates.

## IDM state values
`NONE`, `CANDIDATE`, `SWEPT`, `MSS_CONFIRMED`, `INVALIDATED`, `EXPIRED`.
IDM candidate/sweep is context, never an entry by itself.

## Per-bar order
Update confirmed pivots -> create liquidity pools and IDM candidates -> snapshot prior active levels -> detect sweeps/reclaims -> update IDM state -> detect subsequent MSS -> pass qualified event to OB/FVG and entry modules.

## Edge cases
- If IDM sweep and MSS appear on same candle, reject as sequence ambiguous (reason `SAME_BAR_AMBIGUOUS`).
- If MSS occurs before IDM sweep, do not qualify the IDM sequence.
- If multiple internal swings qualify, select the most recent unconsumed candidate by default and expose other candidates in debug.
