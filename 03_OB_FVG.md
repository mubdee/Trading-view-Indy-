# Module 03 — Order Block + Fair Value Gap

## Timeframe
Chart TF only. OB/FVG detection and entry retest both use the chart timeframe.

## Displacement
Bullish proposal: range > 0; body/ATR(14) >= input (default 1.0); close in upper 25% of range; optionally require structure break. Bearish mirrored. Closed candle only.

## OB creation
After qualifying displacement, search backward from displacement candle for the most recent opposite-colour candle within N bars (default 5). Bullish OB = last bearish candle; bearish OB = last bullish candle. Store origin time, creation time, distal/proximal prices and source break ID.

### Hybrid boundaries (approved)
- Bullish OB source candle is bearish: zone bottom = source low; zone top = source open. This includes lower wick + body and excludes upper wick.
- Bearish OB source candle is bullish: zone bottom = source open; zone top = source high. This includes body + upper wick and excludes lower wick.
- Midpoint = (zone top + zone bottom) / 2.
- If source candle is doji/open equals close, skip by default and log reason; optional doji handling must be an input.
- Invalidation: candle close beyond distal boundary plus configured buffer.

## FVG (three-candle)
At close of current candle:
- Bullish if current low > high[2]; zone = [high[2], current low].
- Bearish if current high < low[2]; zone = [current high, low[2]].
- Gap size must meet configurable ATR/tick minimum.
- Optional displacement filter on middle candle.
Do not confirm using unfinished bar.

## Zone lifecycle per later candle
1. Check overlap; set first-touch once.
2. Track fill percentage consistently from proximal edge.
3. Mark mitigation at selected threshold: touch / 50% / full fill.
4. Invalidate on close beyond distal boundary + buffer.
5. Expire after max age.
6. Keep invalidated historical zones optionally with changed style.
7. Process zones existing before current candle before adding new zones, preventing instant false retest on creation candle.

OB/FVG presence is not an entry. Entry trigger must be separately selected (e.g. retest + rejection candle, internal MSS, or SFP — all on chart TF).
