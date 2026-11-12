# TradingView SMC Indicator — Master Specification
Version: 2.0 — Consolidated rewrite, single timeframe | Target: Pine Script v6 | Source: `src/SMC_SingleTF.pine`

## 1. Objective
One analytical TradingView **indicator** (not a strategy) for XAUUSD and Forex that shows, on the **chart timeframe only**:
- Market structure: HH / HL / LH / LL / EQH / EQL, major + internal BOS / CHoCH, fake BOS, MSS
- Liquidity: BSL / SSL / EQH / EQL / PDH / PDL / PWH / PWL and sweeps
- IDM (inducement): candidate → swept → MSS → invalidated / expired
- OB (Hybrid) + FVG zones with lifecycle
- SFP (user definition) + Rejection Block
- SMT divergence against a comparison symbol on the same timeframe
- Confirmed entry markers with a passed / failed checklist, dashboard and alerts

It explains which conditions passed and why a setup was or was not confirmed. No win-rate claims.

## 2. Locked decisions (summary — full log in `09_DECISIONS_AND_CHANGELOG.md`)
| Topic | Decision |
|---|---|
| Timeframe | Single timeframe. Every module reads the chart TF. No HTF/LTF/MTF logic, no timeframe inputs. |
| IDM | Confirmed internal swing liquidity swept (with reclaim) before MSS. |
| MSS level | Latest confirmed **internal** swing before the IDM sweep (input: Internal / Major). |
| Displacement vs MSS | Window N = 0: the MSS-breaking candle must itself be the displacement candle. |
| Protected swing | Major swing that started the leg of the latest major break; if no break yet, latest major swing. |
| First break from neutral | Body close = real BOS (sets bias). Wick only = **fake BOS** (bias stays neutral). |
| OB boundary | Hybrid. Bull `[source low, source open]`; bear `[source open, source high]`. Doji source → skip. |
| SFP | Sweep candle, then a later candle **closes (body)** beyond the sweep candle's opposite extreme. A wick-only break counts only if the **next** candle has high volume. |
| Rejection Block | Rejection-side wick of the SFP sweep candle (Bull `[low, min(o,c)]`, Bear `[max(o,c), high]`); input Wick / Body / Full. |
| Zone to retest | Only the OB / FVG created by that MSS's displacement leg (linked by ID). |
| Entry trigger | Default `Rejection Candle`; options Internal MSS / SFP / Any. |
| Entries per setup | One, then reset with reason `ENTRY_DONE`. |
| Strict profile | Major bias must match direction (neutral fails). Balanced allows neutral / countertrend (labelled CT). |
| SMT | Default `OANDA:XAGUSD` (direct correlation) for gold; user-selectable symbol + Inverse option for Forex/DXY. |

## 3. Engineering rules (non-negotiable)
- Pine Script v6. Indicator, not strategy.
- All logic runs on **closed candles** (`barstate.isconfirmed`). No intrabar preview.
- No future leak, no repainting of confirmed events, no negative offsets, no `lookahead_on`.
- A pivot with R right-side bars is **known only R bars later**. It is drawn on the pivot candle, but nothing may use it before its confirmation candle.
- `request.security()` is used **only** for the SMT symbol on `timeframe.period` (lookahead off).
- Every event has an ID/state and fires once. Every setup reset carries a reason code.
- Handle `na`, insufficient history, tick size, missing comparison data, symbol suffixes and object limits.
- Every threshold is an input. Never claim compile success or results without real tests in TradingView.

## 4. Timeframe
- All modules use the chart timeframe. To analyse another timeframe the user changes the chart.
- PDH/PDL/PWH/PWL are built from chart bars of the **completed** previous day/week (no other-TF request). Disabled automatically when chart TF ≥ D (day levels) or ≥ W (week levels).

## 5. Default inputs
| Input | Default |
|---|---|
| Major / internal pivot L/R | 3/3 · 1/1 |
| Break source | CLOSE (WICK optional, applies only once a bias exists) |
| Break / sweep / invalidation buffer | 0 ticks + 0 × ATR |
| Equal-level tolerance | ATR(14) × 0.05 |
| Displacement | body ≥ ATR(14) × 1.0, close in outer 25 % of range |
| OB search | last opposite candle within 5 bars, requires structure break on the displacement candle |
| FVG minimum | ATR(14) × 0.05 |
| Mitigation | Touch |
| Zone max age · IDM candidate max · pool max age | 300 · 100 · 500 bars |
| IDM reclaim | sweep candle itself (N = 0) |
| SFP window · volume rule | 5 bars · volume ≥ SMA(20) × 1.5 |
| SMT tolerance | 2 bars |
| Profile · trigger | Balanced · Rejection Candle |
| Max boxes / lines / labels | 500 / 500 / 500 |
These are test defaults, not trading truths.

## 6. Per-candle processing order (what the script does on every close)
1. ATR, displacement, volume average (series, every bar)
2. Confirm major / internal pivots → swing labels, liquidity pools, SMT arm
3. Previous-day / week levels (on day/week change)
4. Major + internal BOS / CHoCH / fake BOS / ambiguous → protected swings
5. Liquidity sweeps / breakouts of pools that existed before this candle
6. SFP pending check → SFP confirm → Rejection Block; register new sweep candle
7. Lifecycle of zones that existed before this candle (touch, mitigation, invalidation, expiry)
8. SMT evaluation
9. Setup state machine per direction (IDM → MSS → zone wait → trigger)
10. Entry resolution (conflict check, one entry per setup)
11. New zones: MSS-linked OB, general OB, FVG (added last → no instant false retest)
12. Dashboard (last bar) and alerts

## 7. Signal lifecycle
`NONE → CONTEXT_FOUND → IDM_CANDIDATE → IDM_SWEPT → MSS_CONFIRMED → ZONE_WAIT → ENTRY`
Any stage may reset to `NONE` with a reason code (see `06`). A zone alone is never an entry.

## 8. Module files
| File | Content |
|---|---|
| `01_MARKET_STRUCTURE.md` | Pivots, BOS / CHoCH, fake BOS, protected swings |
| `02_LIQUIDITY_IDM.md` | Pools, sweeps, PD/PW levels, IDM state machine, MSS |
| `03_OB_FVG.md` | Displacement, Hybrid OB, FVG, zone lifecycle |
| `04_SFP_REJECTION_BLOCK.md` | SFP (user definition), Rejection Block |
| `05_SMT.md` | SMT divergence |
| `06_ENTRY_CONFLUENCE_ALERTS.md` | Profiles, triggers, reason codes, alerts, dashboard |
| `07_TEST_REPAINT_AUDIT.md` | Test list, test status, repaint audit |
| `08_CLAUDE_BUILD_PROMPT.md` | Prompt for future changes |
| `09_DECISIONS_AND_CHANGELOG.md` | All approved decisions + change log |
| `10_OPEN_QUESTIONS.md` | Interpretations that still need user confirmation |
| `archive/` | Superseded files (kept for history) |
