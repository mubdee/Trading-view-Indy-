# Module 02 — Liquidity, Sweeps, IDM and MSS

Timeframe: chart TF only.

## Liquidity pools
Sources (independent toggles): major swings (on), internal swings (off), EQH/EQL merge (on), PDH/PDL (on), PWH/PWL (on).
- A confirmed swing high creates a **BSL** pool, swing low an **SSL** pool, stored from the confirmation candle.
- A new swing within tolerance of an active swing pool merges into it → EQH / EQL with touch count.
- PDH/PDL/PWH/PWL: running high/low of the current day/week from chart bars; published only when the day/week has **closed** (first partial period in history is skipped). Older PD/PW pools are replaced. Disabled when chart TF ≥ D / ≥ W.
- Max 80 pools; pool expires after 500 bars.

## Sweep vs breakout (closed candle, pools that existed before this candle)
- BSL: high > level + buffer. Close back **below** level = **sweep** (bearish context). Close above = breakout / taken.
- SSL mirrors.
- Either way the pool is consumed. The penetrating candle is also handed to the SFP module (`04`).

## IDM — approved definition
IDM = a confirmed internal swing liquidity point swept (with reclaim) before MSS.

Bullish sequence (bearish mirrors):
1. **Context**: a protected low exists (and, in Strict, major bias is bullish).
2. **Candidate**: on confirmation of an internal swing low that is **above** the protected low. Newest candidate replaces an older unswept one.
3. **Sweep**: low < IDM − sweep buffer **and** the same candle closes back above IDM (default). Optional: reclaim within N later bars (input, off by default). A close below IDM with N = 0 = acceptance → reset `IDM_ACCEPTED_BEYOND`.
4. On the sweep, snapshot the **MSS level** = latest confirmed internal swing high (input: Internal / Major).
5. **MSS**: a later candle breaks the MSS level (break mode) **and** is a displacement candle (window N = 0; with N > 0 the displacement may be up to N bars earlier but not before the sweep).
   - Break without displacement → reset `BREAK_NO_DISPLACEMENT`.
   - Sweep and break on the same candle → reset `SAME_BAR_AMBIGUOUS` (unless the same-bar option is enabled).
6. After MSS the setup moves to zone wait (`03`, `06`).

Invalidation / expiry (before MSS):
- Close below protected low → `PROTECTED_BROKEN`
- Candidate older than 100 bars → `EXPIRED_MAX_BARS`
- Protected low changed → `PROTECTED_CHANGED`
- No MSS level available → `NO_MSS_LEVEL`

MSS occurring before the IDM sweep never qualifies. IDM candidate / sweep is context, never an entry.

## IDM disabled
If IDM is off and "liquidity sweep → MSS path" is on, a reclaimed SSL/BSL sweep takes the place of the IDM sweep; the rest of the sequence is identical. It is never bypassed silently while IDM is on.

## Drawings
`IDM?` dotted (candidate) → `IDM ✓` solid (swept) → `IDM → MSS` → `IDM ✕` grey dashed (invalidated / expired, reason in tooltip).
