# TradingView SMC Indicator — Master Specification
Version: 1.2 — Open issues (file 10) resolved | Target: TradingView Pine Script v6

## 1. Project objective
Build one configurable TradingView indicator for XAUUSD and Forex that displays:
- Full market structure and liquidity context
- OB + FVG zones
- SFP + Rejection Blocks
- IDM / Inducement
- MTF context and SMT divergence
- Confirmed setup / entry markers

The indicator is an analytical indicator, not an auto-trading strategy. It must explain which conditions passed and why a setup was or was not confirmed.

## 2. User-approved decisions
Full record with dates: `09_DECISIONS_AND_CHANGELOG.md`.
- IDM: Inducement = a confirmed internal swing liquidity point that is swept before MSS.
- Timeframes: all timeframes configurable by the user; no fixed timeframe stack (§4).
- OB boundaries: Hybrid (§6).
- Display: show all structures AND entry signals.
- SFP: multi-candle definition with body-break confirmation or wick-break + high-volume follow-through (`04_SFP_REJECTION_BLOCK.md`).
- First break from Neutral: wick-only = Fake BOS, body close = real BOS (`01_MARKET_STRUCTURE.md`).
- Rejection Block default boundary = rejection-side wick only.
- Default entry trigger = Rejection Candle; one entry per setup.
- MSS, protected swing, lifecycle stages and zone linkage are defined in §7–§8.
- SMT comparison pairs: auto-mapped per chart symbol with manual override and inverse-correlation support (`05_MTF_SMT.md`).

Spec source of truth: `00`–`05`, `06_ENTRY_CONFLUENCE_ALERTS.md`, `07`, `08`, `09_DECISIONS_AND_CHANGELOG.md`, `11_PRECODING_DESIGN.md`.
(`06_CONFLUENCE_ALERTS_UI.md` and `09_DECISIONS_TO_CONFIRM.md` were retired in v1.2.)

## 3. Mandatory engineering rules
- Pine Script v6.
- Confirmed events use closed candles. Optional intrabar preview, if implemented, must be off by default and visually distinct.
- No future leak, no repainting confirmed events, no misleading negative offsets.
- A pivot requiring right-side bars becomes known only after those bars close. Distinguish pivot's historical candle from its confirmation candle.
- Use safe `request.security()` patterns. Never present incomplete HTF values as confirmed.
- Every event has a unique ID/state and triggers once.
- Handle `na`, insufficient history, tick size, price precision, missing comparison data, symbol suffixes and object limits.
- All thresholds are inputs. No hidden assumptions.
- Do not claim win rate or compile success without reproducible tests / actual TradingView compilation.

## 4. Timeframe configuration
All timeframe inputs must be user-adjustable:
- Structure timeframe: chart timeframe by default, or selectable timeframe.
- Context/HTF timeframe: selectable, with option Off.
- Setup/MTF timeframe: selectable, with option Off.
- Entry/LTF timeframe: chart timeframe by default.
- Each module must state which timeframe it reads.
- Provide a mode `Chart TF` so a module can run directly on the active chart timeframe.
- Do not hard-code H4/H1/M15/M5/M1.

### 4.1 Phase 1 support rule (approved)
- Every module supports `Chart TF` and any timeframe **higher than or equal to** the chart timeframe.
- If a module's selected timeframe is lower than the chart timeframe, that module is disabled and the dashboard shows a warning. Ordinary `request.security()` is never used to fake intrabar data.
- Lower-timeframe support through `request.security_lower_tf()` is deferred to Phase 2.
- PDH/PDL are auto-disabled when chart TF >= 1D; PWH/PWL are auto-disabled when chart TF >= 1W. The dashboard states this.

### 4.2 Default timeframe set (approved)
All module timeframes default to `Chart TF`, except HTF Context = `Off` and Setup TF = `Off`.

## 5. Starting inputs (configurable defaults)
| Input | Default |
|---|---|
| Major pivot L/R | 3/3 |
| Internal pivot L/R | 1/1 |
| Break source | CLOSE; optional WICK |
| Equal-level tolerance | ATR(14) × 0.05 |
| FVG minimum size | ATR(14) × 0.05 |
| OB search | last opposite candle within 5 bars |
| Displacement body | >= ATR(14) × 1.0 |
| Displacement close location | outer 25% of candle range |
| Displacement window (bars before MSS break) | 0 = break candle must itself be displacement |
| MSS break level | Internal (option: Major) |
| Signal confirmation | bar close |
| Break buffer | 0 |
| Sweep buffer | 0 |
| OB/FVG invalidation buffer | 0 |
| Mitigation threshold | Touch |
| Zone max age | 300 bars |
| IDM candidate max bars | 100 bars |
| IDM reclaim within N bars | Off (sweep candle must reclaim) |
| Liquidity pool max age | 500 bars |
| SMT timestamp tolerance | 2 bars |
| Max boxes / lines / labels | 500 / 500 / 500 |
| Entry trigger | Rejection Candle |
| Entries per setup | 1 |
These are initial test defaults, not universal trading truths. SFP-specific defaults are in `04_SFP_REJECTION_BLOCK.md`.

## 6. OB Hybrid boundary — approved definition
Hybrid means include the distal wick plus candle body, but exclude the wick on the proximal side:
- Bullish OB (source candle is bearish): zone = `[source low, source open]`.
- Bearish OB (source candle is bullish): zone = `[source open, source high]`.
This definition is fixed unless the user later changes it. Show the zone midpoint and allow a separate optional 50% line.
If source candle is doji / open equals close, apply a configurable doji rule or skip the OB; do not guess.

## 7. IDM — approved definition
IDM is a confirmed internal swing liquidity point that is swept before MSS.

### 7.1 Protected major swing (approved)
- Protected low (bullish side) = the confirmed major swing low that started the leg which produced the most recent bullish major BOS/CHoCH. If no bullish break exists yet, use the most recent confirmed major swing low.
- Protected high (bearish side) mirrors this.
- The "dealing range model" option is removed from Phase 1.

### 7.2 Bullish sequence
1. A confirmed internal swing low forms above the current protected low.
2. Mark it as a bullish IDM candidate only after its pivot is confirmed.
3. Price trades below the IDM level by the sweep buffer.
4. The sweep candle itself closes back above the IDM level (default). Optional input: reclaim within N bars (off by default).
5. Mark IDM as swept once and preserve the event ID.
6. Wait for bullish MSS (§7.3).
7. Only then may the setup advance to OB/FVG retest and entry-trigger stages.
8. If price closes below the protected low before bullish MSS, invalidate the bullish IDM setup.
Bearish sequence mirrors this using an internal swing high below the protected high, sweep above it, reclaim close below, then bearish MSS.

### 7.3 MSS (approved)
- Bullish MSS = a candle that breaks the **most recent confirmed internal swing high that existed before the IDM sweep**, using the selected break mode and break buffer.
- Input `MSS break level`: `Internal` (default) / `Major` (most recent confirmed major swing high before the sweep).
- Displacement: input `Displacement window N bars`, default 0 = the MSS break candle must itself qualify as displacement. With N > 0, a displacement candle `d` must satisfy `sweepBar < d <= breakBar` and `breakBar - d <= N`.
- MSS on the same candle as the IDM sweep is rejected by default (sequence ambiguous).
- Bearish mirrors.

Important:
- IDM sweep alone is not MSS and is not an entry.
- MSS occurring before the IDM sweep does not satisfy this sequence.
- A candidate expires when its protected major swing changes, after `IDM candidate max bars`, or after invalidation.
- Candidate / swept / confirmed / invalidated states must be visually distinct.

## 8. Signal lifecycle
`NONE -> CONTEXT_FOUND -> IDM_CANDIDATE -> IDM_SWEPT -> MSS_CONFIRMED -> ZONE_WAIT -> ENTRY_TRIGGERED -> NONE`

| Stage | Entered when |
|---|---|
| CONTEXT_FOUND | A valid protected major swing exists for that side AND the HTF filter passes for the active profile (skipped if HTF = Off). |
| IDM_CANDIDATE | A qualified IDM candidate exists (§7.2 steps 1–2). |
| IDM_SWEPT | Candidate swept and reclaimed (§7.2 steps 3–5). |
| MSS_CONFIRMED | §7.3 satisfied. |
| ZONE_WAIT | Waiting for retest of an OB or FVG **created by that MSS displacement leg**, linked by `sourceBreakId`. Other active zones do not qualify. |
| ENTRY_TRIGGERED | Selected entry trigger fires inside the linked zone. Exactly one entry per setup, then reset to NONE with reason `ENTRY_DONE`. |

Reset reason codes: `PROTECTED_BROKEN`, `PROTECTED_CHANGED`, `IDM_EXPIRED`, `SAME_BAR_AMBIGUOUS`, `CONTEXT_LOST`, `NO_LINKED_ZONE`, `ZONE_INVALIDATED`, `ZONE_EXPIRED`, `ENTRY_DONE`.
A zone alone is not an entry signal.

## 9. Dependency order
1. Candle / ATR utilities
2. Confirmed major and internal pivots
3. BOS / CHoCH / Fake BOS / MSS state
4. Liquidity pools and sweeps
5. IDM candidate and sweep state
6. Displacement
7. OB / FVG zones
8. SFP / Rejection Block
9. MTF context
10. SMT divergence
11. Confluence / entry triggers / alerts
12. Rendering / dashboard / debug / performance

## 10. Chart output
Show all structures plus entry signals:
- HH / HL / LH / LL / EQH / EQL
- Major and internal BOS / CHoCH / Fake BOS / MSS
- BSL / SSL / PDH / PDL / PWH / PWL and sweeps
- IDM candidate, swept, confirmed, invalidated
- Bullish / bearish OB and FVG zones with lifecycle
- SFP and Rejection Block markers/zones
- MTF bias and timeframe status
- SMT divergence markers
- Entry signal markers with passed/failed condition checklist
- Dashboard and confirmed-event alerts

## 11. Required Claude workflow
Before coding: explain exact candle-by-candle rules, ambiguities, pseudocode and tests. Do not write full script until user approves the remaining decisions.
After approval: implement module-by-module, integrate, audit repaint/lookahead, provide source and honest test status.

Status (v1.2): pre-coding design is in `11_PRECODING_DESIGN.md`. Waiting on SFP follow-up questions F1–F4 in `10_OPEN_ISSUES_FOR_USER.md` and approval of file 11 before coding.
