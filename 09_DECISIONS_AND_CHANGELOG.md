# Decisions and Change Log
(Single decisions file. `09_DECISIONS_TO_CONFIRM.md` was retired in v1.2; its items 2, 3, 4, 6, 8 use the Master §5 defaults, items 1, 5, 12 were already approved, item 9's fixed TF stack is rejected per Master §4.)

## User-approved decisions — 2026-11-12 (round 1)
- IDM: Inducement = confirmed internal swing liquidity swept before MSS.
- Timeframes: all module timeframes user-adjustable; support Chart TF and selectable timeframe.
- OB: Hybrid boundary.
  - Bullish: `[source low, source open]`.
  - Bearish: `[source open, source high]`.
- Indicator display: all structures + entry signals.

## User-approved decisions — 2026-11-12 (round 2, file 10)
| ID | Decision |
|---|---|
| A1 | `06_ENTRY_CONFLUENCE_ALERTS.md` is the Module 06 source; `06_CONFLUENCE_ALERTS_UI.md` deleted. |
| A2 | `09_DECISIONS_AND_CHANGELOG.md` is the decisions source; `09_DECISIONS_TO_CONFIRM.md` deleted. |
| A3 | **User's own definition.** SFP = sweep of a high/low, then a candle **body close** back through the trigger level. Wick-only break ≠ SFP, unless the next candle has strong volume. SFP is a follow-up event of the sweep (stores parent sweep ID); alerts stay separate. Details + follow-ups F1–F4: Module 04. |
| B1 | MSS breaks the most recent confirmed internal swing that existed before the IDM sweep; input Internal/Major, default Internal. |
| B2 | Displacement window N bars, default 0 (break candle must be displacement). |
| B3 | Protected major swing = major swing that started the leg of the latest BOS/CHoCH; fallback latest confirmed major swing. Dealing-range model removed. |
| B4 | CONTEXT_FOUND = valid protected swing on that side + HTF filter passes (skipped if HTF Off). |
| B5 | ZONE_WAIT only for OB/FVG created by that MSS displacement leg (`sourceBreakId`). |
| B6 | **User's own rule.** From Neutral: wick-only break = Fake BOS (bias unchanged, level stays active); body close = real BOS and sets bias. |
| B7 | RB default = rejection-side wick only; Wick/Body/Full selectable; invalidation = close beyond wick extreme + buffer. |
| B8 | Default entry trigger = Rejection Candle (formula in Module 06). |
| B9 | One entry per setup, reset with `ENTRY_DONE`. |
| B10 | Strict requires HTF alignment (Neutral fails); Balanced allows Neutral. |
| B11 | IDM reclaim on sweep candle by default; within-N-bars input off. |
| C1 | Phase 1: Chart TF or TF >= chart; lower TF disables module with warning; LTF via `request.security_lower_tf()` in Phase 2. |
| C2 | PDH/PDL off on chart TF >= 1D; PWH/PWL off on chart TF >= 1W. |
| D | All recommended defaults accepted (Master §5). |
| E1 | User delegated SMT pairing to Claude → auto map table in Module 05 (XAUUSD↔XAGUSD, EURUSD↔GBPUSD, AUDUSD↔NZDUSD, USDCHF↔EURUSD inverse, USDJPY↔USDCHF) + Manual mode + Inverse correlation input added. |
| E2 | Default TFs: all Chart TF, HTF Context Off, Setup TF Off. |

## Still open before final code
- SFP follow-ups F1–F4 (`10_OPEN_ISSUES_FOR_USER.md` §F). Module 04 currently uses the proposed defaults.
- Approval of `11_PRECODING_DESIGN.md`.

## Change log
### v1.2 — 2026-11-12
- Applied all file-10 answers (A–E) across 00–08.
- Rewrote SFP as multi-candle body-break / wick+volume event linked to parent sweep.
- Added Fake BOS for first break from Neutral.
- Defined MSS level, displacement window, protected swing, lifecycle stages and reset codes, zone linkage, Rejection Candle, one entry per setup.
- Phase 1 timeframe rule; PDH/PWH auto-disable.
- Default inputs table; SMT auto map with inverse correlation.
- Deleted `06_CONFLUENCE_ALERTS_UI.md` and `09_DECISIONS_TO_CONFIRM.md`.
- Added tests 25–40 (Module 07) and `11_PRECODING_DESIGN.md`.

### v1.1
- Added user-approved IDM sequence and state machine.
- Added configurable timeframe architecture for every module.
- Locked OB Hybrid boundary formula.
- Specified all-structure plus entry-signal display.
- Added bar-by-bar tests for IDM sequencing and Hybrid OB boundaries.

### Code v1.2.0 — 2026-11-12
- Added `SMC_Suite_v1.2.pine` (Pine Script v6), written to spec v1.2 with the recommended answers to F1–F6.
- Phase 1 limitation: detection modules run on Chart TF only; HTF context and SMT use `request.security()`. Per-module TF inputs and the IDM-off alternate path are not implemented yet.
- Not compiled yet: compile status unknown until tested in TradingView.

### Code v1.2.1 — 2026-11-12
- Grey/black theme, all lines width 1, lighter zone fills.
- Zones removed from chart once mitigated (input), and always on invalidation/expiry unless "Keep invalidated" is on. Swept/broken liquidity lines and reset IDM lines are deleted.
