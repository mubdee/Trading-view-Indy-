# TradingView SMC Indicator — Master Specification
Version: 1.2 — Single-timeframe (chart TF only) | Target: TradingView Pine Script v6

## 1. Project objective
Build one configurable TradingView indicator for XAUUSD and Forex that displays:
- Full market structure and liquidity context
- OB + FVG zones
- SFP + Rejection Blocks
- IDM / Inducement
- SMT divergence (comparison symbol on the same chart timeframe)
- Confirmed setup / entry markers

The indicator is an analytical indicator, not an auto-trading strategy. It must explain which conditions passed and why a setup was or was not confirmed.

## 2. User-approved decisions
- IDM: Inducement = a confirmed internal swing liquidity point that is swept before MSS.
- Timeframe: single timeframe only. Every module runs on the active chart timeframe; no multi-timeframe (HTF/LTF) analysis.
- OB boundaries: Hybrid.
- Display: show all structures AND entry signals.

## 3. Mandatory engineering rules
- Pine Script v6.
- Confirmed events use closed candles. Optional intrabar preview, if implemented, must be off by default and visually distinct.
- No future leak, no repainting confirmed events, no misleading negative offsets.
- A pivot requiring right-side bars becomes known only after those bars close. Distinguish pivot's historical candle from its confirmation candle.
- `request.security()` is used only to fetch the SMT comparison symbol on the chart timeframe (`timeframe.period`, lookahead off). No other-timeframe requests.
- Every event has a unique ID/state and triggers once.
- Handle `na`, insufficient history, tick size, price precision, missing comparison data, symbol suffixes and object limits.
- All thresholds are inputs. No hidden assumptions.
- Do not claim win rate or compile success without reproducible tests / actual TradingView compilation.

## 4. Timeframe
Single-timeframe design:
- All modules (structure, liquidity, IDM, OB/FVG, SFP/RB, SMT, entry trigger) read the active chart timeframe only.
- No timeframe inputs, no HTF context, no Setup/MTF timeframe, no LTF entry timeframe.
- No `request.security()` for other timeframes and no `request.security_lower_tf()`.
- To analyse another timeframe, the user switches the chart timeframe.

## 5. Starting inputs (configurable proposals)
- Major pivot L/R: 3/3
- Internal pivot L/R: 1/1
- Break source: CLOSE; optional WICK
- Equal-level tolerance: ATR(14) * 0.05
- FVG minimum size: ATR(14) * 0.05
- OB search: last opposite candle within 5 bars
- Displacement body: >= ATR(14) * 1.0
- Displacement close location: outer 25% of candle range
- Signal confirmation: bar close
These are initial test defaults, not universal trading truths.

## 6. OB Hybrid boundary — approved definition
Hybrid means include the distal wick plus candle body, but exclude the wick on the proximal side:
- Bullish OB (source candle is bearish): zone = `[source low, source open]`.
- Bearish OB (source candle is bullish): zone = `[source open, source high]`.
This definition is fixed unless the user later changes it. Show the zone midpoint and allow a separate optional 50% line.
If source candle is doji / open equals close, apply a configurable doji rule or skip the OB; do not guess.

## 7. IDM — approved working definition
IDM is a confirmed internal swing liquidity point that is swept before MSS.
Bullish sequence:
1. A confirmed internal swing low forms above the currently protected major swing low (or within the active bullish dealing range, according to the selected structure model).
2. Mark it as a bullish IDM candidate only after its pivot is confirmed.
3. Price trades below the IDM level by the configured sweep buffer.
4. The same candle or a later candle closes back above the IDM level. The reclaim rule is configurable; default requires the sweep candle itself to close back above.
5. Mark IDM as swept once and preserve the event ID.
6. After the IDM sweep, wait for bullish MSS with qualifying displacement.
7. Only then may the setup advance to OB/FVG retest and entry-trigger stages.
8. If price closes below the protected major low before bullish MSS, invalidate the bullish IDM setup.
Bearish sequence mirrors this using an internal swing high below the protected major swing high, sweep above it, reclaim close below, then bearish MSS.

Important:
- IDM sweep alone is not MSS and is not an entry.
- MSS occurring before the IDM sweep does not satisfy this sequence by default.
- A candidate expires when its protected major swing changes, after configurable max bars, or after invalidation.
- Candidate / swept / confirmed / invalidated states must be visually distinct.

## 8. Signal lifecycle
`NONE -> CONTEXT_FOUND -> IDM_CANDIDATE -> IDM_SWEPT -> MSS_CONFIRMED -> ZONE_WAIT -> ENTRY_TRIGGERED`
At any stage, invalidation or expiry resets the setup with a reason code.
A zone alone is not an entry signal.

## 9. Dependency order
1. Candle / ATR utilities
2. Confirmed major and internal pivots
3. BOS / CHoCH / MSS state
4. Liquidity pools and sweeps
5. IDM candidate and sweep state
6. Displacement
7. OB / FVG zones
8. SFP / Rejection Block
9. SMT divergence
10. Confluence / entry triggers / alerts
11. Rendering / dashboard / debug / performance

## 10. Chart output
Show all structures plus entry signals:
- HH / HL / LH / LL / EQH / EQL
- Major and internal BOS / CHoCH / MSS
- BSL / SSL / PDH / PDL / PWH / PWL and sweeps
- IDM candidate, swept, confirmed, invalidated
- Bullish / bearish OB and FVG zones with lifecycle
- SFP and Rejection Block markers/zones
- Structure bias (chart timeframe)
- SMT divergence markers
- Entry signal markers with passed/failed condition checklist
- Dashboard and confirmed-event alerts

## 11. Required Claude workflow
Before coding: explain exact candle-by-candle rules, ambiguities, pseudocode and tests. Do not write full script until user approves the remaining decisions.
After approval: implement module-by-module, integrate, audit repaint/lookahead, provide source and honest test status.
