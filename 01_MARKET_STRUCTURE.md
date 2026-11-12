# Module 01 — SMC Market Structure: bar-by-bar

## Inputs
- Structure timeframe: Chart TF or user-selected timeframe >= chart TF (Phase 1; see Master §4.1)
- Major pivot L/R: default 3/3
- Internal pivot L/R: default 1/1
- Break method: CLOSE (default) or WICK
- Break buffer: ticks or ATR fraction, default 0
- Separate major/internal structure toggles
- Show Fake BOS: on

## Pivot confirmation
A candidate pivot high/low becomes known only after the configured right-side bars close. Never treat it as available on its original candle. A high above prior confirmed high = HH; below = LH. A low above prior confirmed low = HL; below = LL. Within tolerance = EQH/EQL.

## Each closed candle
1. Update ATR and candle metrics.
2. Confirm pivots whose right-side window has now completed.
3. Snapshot structure levels that existed before this candle.
4. Test break using selected CLOSE/WICK mode and buffer (except while bias is Neutral — see below).
5. Classify using prior bias: continuation = BOS; break against prior bias = CHoCH.
6. Update bias after classification; mark level consumed to prevent duplicate events.
7. If both sides break in one candle, mark ambiguous/expansion and do not issue arbitrary directional signal.
8. Keep internal and major structure independent (each has its own bias and its own Neutral start).
9. MSS requires a qualifying IDM sweep followed by displacement and a structure break (Master §7.3); it is not a label for every BOS/CHoCH.

## First break from Neutral (approved)
While a structure's bias is Neutral, the break mode input is overridden:
- **Wick-only break** (`high > level + buffer` and `close <= level + buffer`; bearish mirrored): emit `FAKE_BOS` once for that level. Bias stays Neutral. The level is **not** consumed, so a later close beyond it can still produce a real BOS.
- **Body break** (`close > level + buffer`; bearish mirrored): emit `BOS` and set bias to the break direction. Level consumed.
- Both sides in one candle: ambiguous, no event, bias stays Neutral.
Once bias is Bullish/Bearish, the normal break mode input applies.

## Protected major swing
Defined in Master §7.1. Updated when a major BOS/CHoCH occurs (protected = the major swing that started the breaking leg) and when a newer major swing is confirmed while no break exists yet.

## Edge cases
Gap beyond level; wick-only break; pivot confirmation and break same bar; equal levels; insufficient history; selected structure timeframe lower than chart timeframe (module disabled with warning).
