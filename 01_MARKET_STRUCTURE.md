# Module 01 — SMC Market Structure: bar-by-bar

## Inputs
- Timeframe: chart TF only (no timeframe input)
- Major pivot L/R: default 3/3
- Internal pivot L/R: default 1/1
- Break method: CLOSE or WICK
- Break buffer: ticks or ATR fraction
- Separate major/internal structure toggles

## Pivot confirmation
A candidate pivot high/low becomes known only after the configured right-side bars close. Never treat it as available on its original candle. A high above prior confirmed high = HH; below = LH. A low above prior confirmed low = HL; below = LL. Within tolerance = EQH/EQL.

## Each closed candle
1. Update ATR and candle metrics.
2. Confirm pivots whose right-side window has now completed.
3. Snapshot structure levels that existed before this candle.
4. Test break using selected CLOSE/WICK mode and buffer.
5. Classify using prior bias: continuation = BOS; break against prior bias = CHoCH. Define first break from neutral explicitly.
6. Update bias after classification; mark level consumed to prevent duplicate events.
7. If both sides break in one candle, mark ambiguous/expansion and do not issue arbitrary directional signal.
8. Keep internal and major structure independent.
9. MSS requires a qualifying liquidity/IDM event followed by displacement and structure break; it is not a label for every BOS/CHoCH.

## Edge cases
Gap beyond level; wick-only break; pivot confirmation and break same bar; equal levels; insufficient history.
