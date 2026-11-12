# Decisions and Change Log

## User-approved decisions — 2026-11-12
- IDM: Inducement = confirmed internal swing liquidity swept before MSS.
- Timeframes: ~~all module timeframes user-adjustable~~ → superseded in v1.2: single timeframe (chart TF only).
- OB: Hybrid boundary.
  - Bullish: `[source low, source open]`.
  - Bearish: `[source open, source high]`.
- Indicator display: all structures + entry signals.

## Still needs explicit definition before final code
1. Rejection Block boundary: wick zone / body zone / full candle. Proposed default is rejection-side wick-to-body area.
2. Entry trigger: rejection candle / internal MSS / SFP / any enabled trigger.
3. Whether IDM reclaim must occur on the sweep candle (default yes) or within N later bars.
4. ~~Exact HTF/context/setup/entry timeframe selections~~ — resolved in v1.2: no MTF, chart TF only.
5. SMT comparison symbol(s).

## Change log
### v1.2
- Removed multi-timeframe analysis: no HTF context, Setup TF, Entry/LTF TF or per-module timeframe inputs. All modules run on chart TF.
- Strict profile: HTF alignment replaced with major structure bias (chart TF) alignment; countertrend = setup against major bias.
- PDH/PDL/PWH/PWL built from chart bars of completed previous day/week (no `request.security()`).
- SMT kept; comparison symbol requested on chart TF only.
- Tests 18 and 23 replaced accordingly.

### v1.1
- Added user-approved IDM sequence and state machine.
- Added configurable timeframe architecture for every module.
- Locked OB Hybrid boundary formula.
- Specified all-structure plus entry-signal display.
- Added bar-by-bar tests for IDM sequencing and Hybrid OB boundaries.
