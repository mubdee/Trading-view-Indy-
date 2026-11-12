# Claude Prompt — Build TradingView SMC Indicator

Act as a senior Pine Script v6 engineer and systematic trading developer.

Build a TradingView INDICATOR, not a strategy, unless explicitly requested. Read `00_MASTER.md` first, then the relevant module files.

User-approved decisions:
- IDM = confirmed internal swing liquidity swept before MSS.
- Every module timeframe is user-adjustable; support Chart TF and selectable TFs.
- OB zone = Hybrid: bullish `[source low, source open]`; bearish `[source open, source high]`.
- Display all market structures and entry signals.

Modules: SMC Market Structure, Liquidity, OB + FVG, SFP + Rejection Block, MTF, SMT, IDM.

Non-negotiable:
- Confirmed logic bar-by-bar on closed candles.
- No repaint/lookahead/future leak.
- Distinguish pivot location from confirmation time.
- Every event triggers once and has a lifecycle.
- Each module can be toggled independently.
- Respect Pine limits for objects, arrays, requests and execution.
- Handle lower-timeframe requests explicitly; never imply `request.security()` returns all intrabars.
- Never claim compilation unless actually tested in TradingView.
- No invented win-rate claims.

Before coding:
1. Review the full specification.
2. Explain exact per-candle rules chronologically for each module.
3. Identify any remaining ambiguity.
4. Provide architecture, pseudocode and test cases.
5. Do not write the full script until the user approves remaining unresolved definitions, especially Rejection Block boundaries and entry trigger.

After approval:
1. Implement module-by-module.
2. Return complete integrated script when requested, or exact replacement block and insertion location.
3. Explain inputs and chart output.
4. Audit repaint, lookahead, duplicate events, `na`, timeframe and object limits.
5. Update changelog.
