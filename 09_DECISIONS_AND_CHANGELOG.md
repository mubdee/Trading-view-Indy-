# Decisions and Change Log

All user decisions in one place. Source of each answer in brackets (`archive/10_OPEN_ISSUES_FOR_USER.md` = user's filled-in questionnaire).

## Locked decisions
| # | Topic | Decision | Source |
|---|---|---|---|
| 1 | IDM | Confirmed internal swing liquidity swept before MSS | v1.1 |
| 2 | Timeframe | **Single timeframe** — chart TF only, no MTF | user request (v1.2) |
| 3 | OB | Hybrid: bull `[low, open]`, bear `[open, high]` | v1.1 |
| 4 | Display | All structures + entry signals | v1.1 |
| 5 | Duplicate files | `06_ENTRY_CONFLUENCE_ALERTS.md` and `09_DECISIONS_AND_CHANGELOG.md` are authoritative; old ones archived | A1, A2 |
| 6 | SFP | Sweep candle, then **body** close through that candle's opposite extreme; wick-only counts only if the next candle has strong volume | A3 (user's own definition) |
| 7 | MSS level | Latest confirmed internal swing before IDM sweep; input Internal / Major | B1 |
| 8 | Displacement window | N = 0 (MSS candle is the displacement candle) | B2 |
| 9 | Protected swing | Origin swing of the latest major break; else latest major swing; "dealing range" model dropped | B3 |
| 10 | CONTEXT_FOUND | Protected swing exists (+ major bias in Strict) | B4 (HTF part removed in v1.2) |
| 11 | Zone to retest | Only OB/FVG from that MSS's displacement leg | B5 |
| 12 | First break from neutral | Wick = fake BOS, body = real BOS | B6 (user's own answer) |
| 13 | Rejection Block | Rejection-side wick (option a); inputs Wick / Body / Full; invalidation beyond wick + buffer | B7 |
| 14 | Entry trigger | Default Rejection Candle (touch zone, close beyond proximal, close in favourable half) | B8 |
| 15 | Entries per setup | 1, then reset `ENTRY_DONE` | B9 |
| 16 | Strict profile | Direction must match (neutral fails); Balanced allows neutral — now based on major structure bias | B10 (HTF → major bias in v1.2) |
| 17 | IDM reclaim | Sweep candle itself; "within N bars" input off by default | B11 |
| 18 | PD/PW on high TF | Auto-off when chart TF ≥ D / ≥ W | C2 |
| 19 | Default values | All recommended values in `00_MASTER.md` §5 | D |
| 20 | SMT symbol | Claude's choice: `OANDA:XAGUSD` for gold, selectable for Forex, inverse option | D / E1 (confirmed in chat) |
| — | C1, E2 (per-module timeframes) | Cancelled — single timeframe | v1.2 |

## Change log
### v2.0 — 2026-11-12
- Full rewrite of the spec as one consistent set; superseded files moved to `archive/`.
- First implementation: `src/SMC_SingleTF.pine` (Pine v6, **not yet compiled in TradingView**).
- Added user SFP definition (A3) and fake-BOS rule (B6) to the spec and code.
- SMT default symbol `OANDA:XAGUSD` + inverse-correlation input.
- New `10_OPEN_QUESTIONS.md` for interpretations made while coding.

### v1.2
- Removed multi-timeframe analysis; all modules on chart TF; Strict uses major bias instead of HTF.

### v1.1
- IDM sequence and state machine; Hybrid OB; configurable timeframes (later removed in v1.2); all-structure display.
