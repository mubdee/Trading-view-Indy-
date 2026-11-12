# Module 03 — Displacement, Order Block, Fair Value Gap

Timeframe: chart TF only (detection and retest).

## Displacement (closed candle)
Bullish: range > 0, close > open, body ≥ ATR(14) × 1.0, close in the upper 25 % of the range. Bearish mirrors.

## Order Block — Hybrid (approved)
- Search back from the displacement candle, up to 5 bars, for the most recent opposite-colour candle.
- Bullish OB (source bearish): zone = `[source low, source open]` — lower wick + body, no upper wick.
- Bearish OB (source bullish): zone = `[source open, source high]` — body + upper wick, no lower wick.
- If the first non-same-colour candle found is a **doji** (open = close): skip the OB (default, reason `DOJI_SOURCE`) or use it (input).
- Creation:
  - **Setup OB**: on MSS, from the MSS displacement candle; linked to the setup by MSS ID (thicker border).
  - **General OB**: on any displacement candle that also breaks structure (major or internal) — the break requirement is an input.
  - The same source candle never creates two OBs.
- Optional 50 % line.

## FVG (three-candle, confirmed on the close of candle 3)
- Bullish: `low > high[2]`, zone `[high[2], low]`. Bearish: `high < low[2]`, zone `[high, low[2]]`.
- Size ≥ ATR × 0.05 (+ ticks). Optional: middle candle must be displacement.
- An FVG is linked to a setup when its middle candle lies inside that setup's displacement leg (`dispBar … mssBar`).

## Zone lifecycle (zones that existed before this candle are processed first)
1. Overlap → first touch (once).
2. Fill measured from the proximal edge (bull: top, bear: bottom).
3. Mitigation threshold: Touch (default) / 50 % / Full → dashed border. A mitigated zone can still serve the entry until invalidated.
4. Invalidation: close beyond the distal edge + buffer.
5. Expiry: 300 bars.
6. Invalidated / expired zones are kept faded (input) or deleted. Max 150 tracked zones.
7. New zones are added after the lifecycle step, so a zone cannot be "retested" by its own creation candle.

OB / FVG presence is never an entry by itself.
