# Module 04 — SFP + Rejection Block

## Timeframe
SFP reference level timeframe and confirmation timeframe are configurable independently (Chart TF or >= chart TF in Phase 1).

## SFP — user definition (approved 2026-11-12)
> SFP = a candle sweeps a high, then price breaks back through the low of that high **with a candle body (close)**. A wick-only break is not an SFP — **unless** the candle right after the wick break has strong volume; then it counts as an SFP.

SFP is therefore a multi-candle event that follows a sweep. It is not the same event as the sweep: the sweep is the parent, SFP is its confirmation, and SFP stores the parent sweep ID. Sweep and SFP keep separate alerts.

### Bearish SFP (bullish mirrors: sweep below a low, break up through the trigger high)
1. **Reference level R**: an active BSL (confirmed swing high, EQH, PDH/PWH if enabled) that existed before candle `s`.
2. **Sweep candle s**: `high[s] > R + sweep buffer` and `close[s] < R` (closes back below; same rule as Module 02 sweep). *[F1]*
3. **Trigger level T** — input `SFP trigger level`: *[F2]*
   - `Sweep candle low` (default): `T = low[s]`
   - `Swept swing candle low`: `T = low` of the pivot candle that formed R
4. **Confirmation window**: candles after `s` up to `SFP max bars` (default 5). *[F3]* For each closed candle `c` in the window:
   - **Body break**: `close[c] < T` → SFP confirmed at close of `c`.
   - **Wick break**: `low[c] < T` and `close[c] >= T` → record wick break, no SFP yet. If the next candle `c+1` is a **strong-volume candle**, SFP is confirmed at close of `c+1`. If not, keep watching the window; a later body break still confirms.
   - Strong-volume candle: `volume[c+1] >= SMA(volume, 20)[c] × 1.5` (both inputs) and, by default, `c+1` closes in the SFP direction (bearish candle `close < open`). *[F4]*
5. **Cancel before confirmation**: a close above `high[s]` (sweep extreme) → `SFP_INVALIDATED`; window ends with no confirmation → `SFP_EXPIRED`.
6. One SFP per sweep. Event ID = parent sweep ID + `SFP` tag.

### Volume note
On XAUUSD and Forex, TradingView `volume` is broker tick volume, not exchange volume. If `volume` is `na` or zero on the symbol, the volume path is unavailable: only body-break SFPs are confirmed, and the dashboard shows "SFP volume: N/A".

## Rejection Block
Created at the close of a qualified sweep candle (liquidity raid that closes back across the level). Source candle = sweep candle `s`. Boundary mode input:
- `Wick` (default, approved): rejection-side wick only.
  - Bearish RB = `[max(open, close), high]`
  - Bullish RB = `[low, min(open, close)]`
- `Body`: `[min(open, close), max(open, close)]`
- `Full`: `[low, high]`
Invalidation: close beyond the wick extreme + buffer (bearish: close > high[s] + buffer; bullish: close < low[s] − buffer). If the rejection-side wick has zero length, skip RB and log the reason.

## Each candle
Snapshot pre-existing levels -> detect penetration/reclaim (sweep) -> create RB after close -> advance pending SFP windows (body break / wick break / volume follow-through / cancel / expire) -> on later candles track RB first touch/mitigation/invalidation/expiry. RB alone is not an order.

Edge cases: both sides swept = ambiguous; no active reference = no SFP; multiple nearby levels resolved deterministically (nearest level beyond which the wick closed back) and exposed in debug; a new sweep of the same side while an SFP window is open replaces the pending one (older one → `SFP_EXPIRED`).
