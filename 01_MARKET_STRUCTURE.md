# Module 01 — Market Structure

Timeframe: chart TF only.

## Inputs
Major pivot L/R (3/3) · internal pivot L/R (1/1) · break source CLOSE / WICK · break buffer (ticks + ATR×) · equal-level tolerance (ATR × 0.05) · show major / internal / swing labels / fake BOS.

## Pivots
- `ta.pivothigh/low(L, R)`. The pivot candle is R bars back; the pivot becomes **known on the confirmation candle**. Labels are drawn on the pivot candle, but no logic uses the pivot before confirmation.
- Classification against the previous confirmed pivot of the same layer: within tolerance = EQH / EQL; higher high = HH, lower high = LH; higher low = HL, lower low = LL.
- Major and internal layers are fully independent (own levels, own bias).

## Break test (each closed candle, per layer)
1. Levels = last confirmed swing high / low that is not yet consumed.
2. Break condition:
   - If bias is **neutral**: real break needs a **body close** beyond level + buffer.
   - If bias exists: selected mode — CLOSE (close beyond) or WICK (high/low beyond).
3. Classification: break with prior bias = **BOS**; against prior bias = **CHoCH**; from neutral = BOS. Bias is set to the break direction and the level is consumed (one event per level).
4. **Fake BOS** (user decision): bias neutral, wick beyond the level but close not beyond → marked `fake BOS` once per level; bias stays neutral; level stays active.
5. Both levels broken on one candle → `AMB` (ambiguous), both consumed, no directional event.

## Protected swings
- Protected low = major swing low that was the latest confirmed major low when the latest bullish major BOS/CHoCH happened. If no bullish break since the last bearish break, protected low = latest confirmed major low.
- Protected high mirrors.
- A bearish major break clears the bullish "seen" flag (and vice versa), so protection falls back to the latest major swing.

## MSS
MSS is **not** a label for every BOS. It exists only inside the setup state machine (`02`): IDM sweep → break of the snapshotted level with displacement.

## Edge cases
Gap beyond level (close test still applies) · wick-only break (mode / fake BOS) · pivot confirmation and break on the same bar (impossible by pivot definition) · equal levels · insufficient history (logic waits for ATR).
