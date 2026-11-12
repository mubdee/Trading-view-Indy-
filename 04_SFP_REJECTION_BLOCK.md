# Module 04 — SFP + Rejection Block

## Timeframe
SFP reference level timeframe and confirmation timeframe are configurable independently.

## SFP
Bearish SFP: active reference BSL/swing high existed before current candle; current high exceeds it by buffer; candle closes back below it. Bullish is mirrored below SSL and closes back above. Optional filters: close in outer half of range, wick/body ratio, volume (off by default). Confirm at close. Store level ID, penetration, reclaim and timestamp. If close remains beyond level, classify breakout, not SFP.

## Rejection Block
Create only after a qualified liquidity raid/SFP candle rejects the level. Boundary mode is an input:
- Wick zone
- Body zone
- Full candle range
Default proposal: rejection-side wick-to-body area; invalidation beyond wick extreme + buffer. Do not silently change this definition. User can later select another mode.

## Each candle
Snapshot pre-existing levels -> detect penetration/reclaim -> measure rejection candle -> create RB after close -> on later candles track first touch/mitigation/invalidation/expiry. RB alone is not an order.

Edge cases: both sides swept = ambiguous; no active reference = no SFP; multiple nearby levels resolved deterministically and exposed in debug.
