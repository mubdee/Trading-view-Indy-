# SMC Single-TF — TradingView Indicator

Pine Script v6 indicator for XAUUSD / Forex: market structure, liquidity, IDM, Hybrid OB + FVG, SFP + Rejection Block, SMT and confirmed entry markers — **on the chart timeframe only**.

> ⚠ The script has **not been compiled in TradingView yet**. Please compile and report any errors.

## Install
1. Open TradingView → Pine Editor → New → Indicator.
2. Replace everything with the content of [`src/SMC_SingleTF.pine`](src/SMC_SingleTF.pine).
3. Save → Add to chart. If the editor shows an error, copy the message and line number.

## Use
- Change the chart timeframe to analyse another timeframe (the script never mixes timeframes).
- Settings groups: 1 Structure · 2 Liquidity · 3 IDM/MSS · 4 OB/FVG · 5 SFP/RB · 6 SMT · 7 Entry · 8 Display.
- For gold the SMT symbol defaults to `OANDA:XAGUSD`. For Forex choose a correlated pair; tick *Inverse* for DXY-type pairs.
- Alerts: create an alert on the indicator and pick a condition; use "Once per bar close".

## Spec
Start with [`00_MASTER.md`](00_MASTER.md). Open questions for the author: [`10_OPEN_QUESTIONS.md`](10_OPEN_QUESTIONS.md).
