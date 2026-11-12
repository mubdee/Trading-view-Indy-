# Module 03 — Order Block + Fair Value Gap

## Timeframe
OB and FVG detection timeframe is independently selectable: Chart TF or a timeframe >= chart TF (Phase 1). Entry retest timeframe is separately configurable.

## Displacement
Bullish: range > 0; body/ATR(14) >= input (default 1.0); close in upper 25% of range; optionally require structure break. Bearish mirrored. Closed candle only.
For MSS, the displacement candle must fall inside the displacement window (Master §7.3, default 0 = the break candle itself).

## OB creation
After qualifying displacement, search backward from displacement candle for the most recent opposite-colour candle within N bars (default 5). Bullish OB = last bearish candle; bearish OB = last bullish candle. Store origin time, creation time, distal/proximal prices and `sourceBreakId` (the MSS/BOS event ID that the displacement belongs to).

### Hybrid boundaries (approved)
- Bullish OB source candle is bearish: zone bottom = source low; zone top = source open. This includes lower wick + body and excludes upper wick.
- Bearish OB source candle is bullish: zone bottom = source open; zone top = source high. This includes body + upper wick and excludes lower wick.
- Midpoint = (zone top + zone bottom) / 2.
- If source candle is doji/open equals close, skip by default and log reason; optional doji handling must be an input.
- Invalidation: candle close beyond distal boundary plus invalidation buffer (default 0).

## FVG (three-candle)
At close of current candle:
- Bullish if current low > high[2]; zone = [high[2], current low].
- Bearish if current high < low[2]; zone = [current high, low[2]].
- Gap size must meet configurable ATR/tick minimum.
- Optional displacement filter on middle candle.
Do not confirm using unfinished bar.

## Linkage to an MSS setup (approved)
A zone qualifies for a setup's `ZONE_WAIT` only if it was created by that MSS displacement leg:
- OB: source candle found from the MSS displacement candle search; `sourceBreakId = mssId`.
- FVG: middle candle lies in `[idmSweepBar + 1, mssBar]`; because a FVG completes one bar after its middle candle, FVGs completing on `mssBar + 1` are still linked.
Any other active zone does not qualify.

## Zone lifecycle per later candle
1. Check overlap; set first-touch once.
2. Track fill percentage consistently from proximal edge.
3. Mark mitigation at selected threshold: Touch (default) / 50% / full fill.
4. Invalidate on close beyond distal boundary + buffer.
5. Expire after `Zone max age` (default 300 bars).
6. Keep invalidated historical zones optionally with changed style.
7. Process zones existing before current candle before adding new zones, preventing instant false retest on creation candle.

OB/FVG presence is not an entry. Entry trigger is separate (Module 06).
