# Module 04 — SFP + Rejection Block

Timeframe: chart TF only.

## SFP — user definition (replaces the earlier "wick + close back" proposal)
> SFP = a candle sweeps a high and then price breaks back through the **low** of that candle **with a candle body**. A wick-only break is not an SFP — unless the candle right after it has strongly higher volume.

Bearish SFP:
1. **Sweep candle**: high > an active BSL / swing high / EQH / PDH / PWH level + sweep buffer (the level must have existed before this candle). Its close may be anywhere.
2. On the following candles (window: 5 bars):
   - Close **above** the sweep candle high → cancelled (price accepted above).
   - Close **below** the sweep candle low → **SFP confirmed (body)**.
   - Low below the sweep candle low but close not below → wick break; then if the **next** candle has volume ≥ SMA(volume, 20) × 1.5 → **SFP confirmed (volume)**.
   - Nothing within the window → expired.
3. A newer sweep candle replaces the pending one.

Bullish SFP mirrors: sweep below SSL, then a close above the sweep candle high (or wick break + high-volume next candle); cancelled by a close below the sweep candle low.

Notes:
- Symbols without volume data cannot use the volume path (body path still works).
- The label is placed on the **confirmation** candle (tooltip names the sweep candle) so nothing appears earlier than it was known.
- Reference point is the sweep candle's opposite extreme — see `10_OPEN_QUESTIONS.md` Q1.

## Rejection Block
Created only when an SFP is confirmed, from the **sweep candle**:
| Mode | Bearish | Bullish |
|---|---|---|
| Wick (default) | `[max(open, close), high]` | `[low, min(open, close)]` |
| Body | `[min(o, c), max(o, c)]` | same |
| Full | `[low, high]` | same |
Invalidation: close beyond the wick extreme (high for bearish, low for bullish) + buffer. A wick zone of zero height is skipped. Same lifecycle as other zones. An RB alone is not an order.
