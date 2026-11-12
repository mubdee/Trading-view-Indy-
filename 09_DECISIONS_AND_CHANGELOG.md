# Decisions and Change Log

## User-approved decisions — 2026-11-12
- IDM: Inducement = confirmed internal swing liquidity swept before MSS.
- Timeframes: all module timeframes user-adjustable; support Chart TF and selectable timeframe.
- OB: Hybrid boundary.
  - Bullish: `[source low, source open]`.
  - Bearish: `[source open, source high]`.
- Indicator display: all structures + entry signals.

## Still needs explicit definition before final code
1. Rejection Block boundary: wick zone / body zone / full candle. Proposed default is rejection-side wick-to-body area.
2. Entry trigger: rejection candle / internal MSS / SFP / any enabled trigger.
3. Whether IDM reclaim must occur on the sweep candle (default yes) or within N later bars.
4. Exact HTF/context/setup/entry timeframe selections. Inputs will be configurable; no fixed stack is assumed.
5. SMT comparison symbol(s).

## Change log
### v1.1
- Added user-approved IDM sequence and state machine.
- Added configurable timeframe architecture for every module.
- Locked OB Hybrid boundary formula.
- Specified all-structure plus entry-signal display.
- Added bar-by-bar tests for IDM sequencing and Hybrid OB boundaries.
