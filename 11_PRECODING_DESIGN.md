# 11 — Pre-coding Design (Master §11)
Version: 1.0 | Date: 2026-11-12 | Spec: v1.2
Status: **waiting for approval (file 10 §F6)**. No Pine code is written until then.

This file covers the steps Master §11 requires before coding: exact candle-by-candle rules, the remaining ambiguities, architecture, pseudocode, and the test map.

---

## 1. Remaining ambiguities
| ID | Item | Spec currently uses |
|---|---|---|
| F1 | Must the SFP sweep candle close back across R? | Yes |
| F2 | SFP trigger level T | Sweep candle low/high |
| F3 | SFP confirmation window | 5 bars |
| F4 | Strong-volume rule | vol >= SMA20 × 1.5 and candle closes in SFP direction |
| F5 | Fake BOS scope | Neutral bias only |
| §6.3 | Internal MSS / SFP trigger definitions (below) | Proposed here, needs approval |

Everything else in the spec is fixed.

---

## 2. Architecture

### 2.1 One script, one source of truth per module
`indicator("SMC Suite", overlay = true, max_boxes_count = 500, max_lines_count = 500, max_labels_count = 500)`

Each module is a group of functions plus `var` state. Modules talk to each other only through the shared types below and event IDs. There are no hidden globals.

### 2.2 User-defined types
```
type Pivot      // confirmed swing
    float price
    int   pivotBar      // historical candle of the swing
    int   pivotTime
    int   confirmBar    // bar on which it became known (pivotBar + R)
    bool  isHigh
    bool  isMajor
    string label        // HH/HL/LH/LL/EQH/EQL

type Level      // structure level or liquidity pool
    int   id
    float price
    bool  isHigh
    string source       // MAJOR / INTERNAL / EQ / PDH / PDL / PWH / PWL
    int   createdBar
    int   originPivotBar
    float originCandleLow   // for SFP F2 option (b)
    float originCandleHigh
    int   touches
    int   state         // ACTIVE / FAKE_BROKEN / CONSUMED / EXPIRED

type Zone       // OB / FVG / RB
    int   id
    string kind         // OB / FVG / RB
    bool  bullish
    float top
    float bottom
    int   createdBar
    int   sourceBreakId // MSS/BOS id for OB/FVG, sweep id for RB
    int   firstTouchBar
    int   state         // ACTIVE / MITIGATED / INVALIDATED / EXPIRED
    box   bx

type SfpPending
    int   sweepId
    bool  bearish
    float trigger       // T
    float sweepExtreme  // high[s] (bearish) / low[s] (bullish)
    int   sweepBar
    int   wickBreakBar  // na until a wick break happens

type Setup      // one per side (bull, bear)
    int   stage         // NONE / CONTEXT_FOUND / IDM_CANDIDATE / IDM_SWEPT / MSS_CONFIRMED / ZONE_WAIT / ENTRY_TRIGGERED
    float protectedPrice
    int   protectedId
    int   idmLevelId
    float idmPrice
    int   idmCreatedBar
    int   sweepBar
    float mssLevel      // internal (or major) swing snapshotted at sweep time
    int   mssId
    int   mssBar
    array<int> linkedZoneIds
    string lastReason
```

### 2.3 Event IDs
A single `var int nextId` increments for every event. An event stores its ID and type. An alert or label is emitted only when an ID is created, so it fires exactly once. Realtime rollback is handled by Pine itself, because `var` state and objects created on unconfirmed ticks are rolled back. All state transitions are also gated by `barstate.isconfirmed`, so confirmed history equals realtime-closed results.

### 2.4 Timeframe execution (Phase 1)
- `tfOf(input)` resolves `Chart TF` → `timeframe.period`.
- `tfAllowed(tf) = timeframe.in_seconds(tf) >= timeframe.in_seconds()`. If false, the module is off and the dashboard shows `"<module>: TF lower than chart — disabled (Phase 2)"`.
- **Module TF = chart:** runs natively with full drawing.
- **Module TF > chart:** module function runs inside `request.security(syminfo.tickerid, tf, f()[1], lookahead = barmerge.lookahead_on)` and returns a tuple of scalars (bias, active levels, last event id/price/time). The chart side detects a change in the returned event id and draws it at `xloc.bar_time` using the returned HTF timestamp. Only completed HTF bars are used.
- All `request.*` calls sit at global scope and are unconditional (a Pine requirement). Disabled modules ignore the result.
- Request budget: Structure, Liquidity, OB/FVG, SFP/RB, IDM, HTF, Setup, and SMT = at most 8 tuple calls, plus PDH/PDL and PWH/PWL = 2. This is well under Pine's limit.

### 2.5 Object management
Each kind (box/line/label) has a FIFO array. Before creating an object, if the array is at the cap, delete the oldest object. Invalidated zones can be kept (restyled) or deleted (input).

---

## 3. Per-candle order (chart-TF run)
Steps run top to bottom on every confirmed bar:

```
0  guard     : if bar_index < warmup (max(majorL+majorR, 20, ATR len)) → state only, no signals
1  utils     : atr, rng, body, upWick, dnWick, isBull, isBear, mintick
2  pivots    : ph/pl = ta.pivothigh/low(L, R) for major and internal
               on non-na → Pivot{pivotBar = bar_index - R, confirmBar = bar_index}
               → classify HH/LH/HL/LL/EQ → create Level(s) / liquidity pool
               → IDM candidate check (§4.4) → protected-swing fallback update
3  snapshot  : copy active structure levels + pools (anything created before this candle's break test)
4  structure : break test per major / internal (§4.1) → BOS / CHoCH / FAKE_BOS → bias → protected update
5  liquidity : sweep test on snapshotted pools (§4.3) → sweep events
6  idm       : sweep/reclaim/invalidate/expire per side (§4.4)
7  disp      : displacement flags for this candle (§4.5)
8  mss       : for setups in IDM_SWEPT (§4.6)
9  zones     : (a) lifecycle of EXISTING zones first (touch/mitigate/invalidate/expire)
               (b) then create new OB (on displacement) / FVG / RB
10 sfp       : create SfpPending from step-5 sweeps; advance all pending (§4.7)
11 mtf       : read HTF bias from security result
12 smt       : compare pivots with comparison-symbol pivots (§4.8)
13 setup     : state machine + entry trigger (§5, §6)
14 output    : draw, dashboard (last bar only), alerts
```
Why this order: pivots are confirmed before the snapshot, so a pivot confirmed this bar can be broken from the next bar on. (It cannot be broken on its confirmation bar anyway, because the right-side bars are by definition below or above it.) Existing zones are processed before new ones, so a zone cannot be "retested" on the candle that created it.

---

## 4. Module rules, candle by candle

### 4.1 Structure break (Module 01)
```
for each structure s in [major, internal]:
  lvlH = s.lastActiveHigh (snapshot), lvlL = s.lastActiveLow
  wickUp  = high  > lvlH + buf ;  bodyUp  = close > lvlH + buf
  wickDn  = low   < lvlL - buf ;  bodyDn  = close < lvlL - buf
  if s.bias == NEUTRAL:
      up = bodyUp ; dn = bodyDn
      if up and dn            → AMBIGUOUS (no event)
      else if up              → BOS bull, bias = BULL, consume lvlH
      else if dn              → BOS bear, bias = BEAR, consume lvlL
      else:
         if wickUp and lvlH.state != FAKE_BROKEN → FAKE_BOS bull, lvlH.state = FAKE_BROKEN (still breakable)
         if wickDn and lvlL.state != FAKE_BROKEN → FAKE_BOS bear, ...
  else:
      up = breakMode == CLOSE ? bodyUp : wickUp   (same for dn)
      if up and dn            → AMBIGUOUS
      else if up              → bias == BULL ? BOS : CHoCH ; bias = BULL ; consume
      else if dn              → mirrored
  if s is major and a bullish break happened:
      protectedLow = latest confirmed major low with pivotBar < bar_index  (origin of the breaking leg)
      if protectedLow changed → setups[bull] reset PROTECTED_CHANGED (unless stage >= MSS_CONFIRMED)
```
Fake BOS is emitted once per level (`FAKE_BROKEN` stops repeats). If a later major low with `pivotBar` before the break bar is confirmed (after R bars), it replaces the protected low. This is the case where the leg origin had not been confirmed yet when the break happened.

### 4.2 Protected swing
- `protectedLow` = as in §4.1. If no bullish major break exists yet, it is the latest confirmed major low.
- Broken when `close < protectedLow`. Then the bull setup resets with `PROTECTED_BROKEN`.

### 4.3 Liquidity sweep (Module 02)
```
for each ACTIVE BSL pool p (snapshot):
  if high > p.price + sweepBuf and close < p.price → SWEEP bear (id), p.state = CONSUMED
  else if close > p.price                           → BREAKOUT (acceptance), p.state = CONSUMED
SSL mirrored. If one candle sweeps both a BSL and an SSL → AMBIGUOUS, no SFP from it.
Multiple pools swept by one candle on one side → keep the farthest pool as the parent, list others in debug.
Expire pools older than poolMaxAge.
```

### 4.4 IDM (bull; bear mirrored)
```
on internal low pivot confirmed:
   if protectedLow exists and pivot.price > protectedLow and setup.stage <= CONTEXT_FOUND:
       setup.idm = pivot ; stage = IDM_CANDIDATE ; idmCreatedBar = bar_index
       (if a newer qualifying internal low appears while still IDM_CANDIDATE → replace; old one listed in debug)
each bar while IDM_CANDIDATE:
   if close < protectedLow                       → reset PROTECTED_BROKEN
   else if bar_index - idmCreatedBar > idmMaxBars → reset IDM_EXPIRED
   else if low < idmPrice - sweepBuf:
        if close > idmPrice                        → stage = IDM_SWEPT, sweepBar = bar_index,
                                                   mssLevel = latest confirmed internal high (or major, per input) with confirmBar <= bar_index
        else if reclaimWithinN enabled            → wait up to N bars for close > idmPrice
        else                                      → candidate consumed without reclaim → reset IDM_EXPIRED
```

### 4.5 Displacement
`dispBull = rng > 0 and body >= atr*dispMult and close >= high - rng*0.25 and isBull` (bear mirrored).
`lastDispBullBar` is stored for the displacement window.

### 4.6 MSS (bull)
```
while stage == IDM_SWEPT:
   if close < protectedLow → reset PROTECTED_BROKEN
   brk = breakMode == CLOSE ? close > mssLevel + buf : high > mssLevel + buf
   if brk:
       if bar_index == sweepBar → reset SAME_BAR_AMBIGUOUS
       else if dispWindow == 0 ? dispBull : (lastDispBullBar > sweepBar and bar_index - lastDispBullBar <= dispWindow):
            MSS bull (id) ; stage = MSS_CONFIRMED ; mssBar = bar_index
       else: no MSS (debug "break without displacement"); remain IDM_SWEPT
```

### 4.7 SFP (bearish; bullish mirrored) — defaults pending F1–F4
```
on SWEEP bear event at candle s:            (F1: sweep already requires close < R)
   T = sfpTriggerMode == SWEEP_LOW ? low[s] : pool.originCandleLow   (F2)
   push SfpPending{T, sweepExtreme = high[s], sweepBar = s}
   if another pending bear SFP exists → older one SFP_EXPIRED
   also create bearish RB from candle s (wick zone [max(o,c), high]) if upper wick > 0
each later bar for each pending p:
   if close > p.sweepExtreme                     → SFP_INVALIDATED, remove
   else if bar_index - p.sweepBar > sfpMaxBars   → SFP_EXPIRED, remove     (F3)
   else if not na(p.wickBreakBar) and bar_index == p.wickBreakBar + 1:
        if volOk and close < open → SFP bear CONFIRMED (via volume), remove   (F4)
        else p.wickBreakBar = na and fall through to the body test below on this same bar
   if still pending:
     if close < p.trigger                        → SFP bear CONFIRMED (body), remove
     else if low < p.trigger                     → p.wickBreakBar = bar_index
volOk = not na(volume) and volume > 0 and volume >= ta.sma(volume, volLen)[1] * volMult
```
The SMA uses `[1]` so the average does not include the candle being tested.

### 4.8 SMT
- Compare symbol resolved once at bar 0 from the auto map (keeps prefix/suffix) or the manual input.
- `[cmpPH, cmpPL, cmpTime] = request.security(cmp, timeframe.period, [ta.pivothigh(L,R), ta.pivotlow(L,R), time[R]])`. This uses the same pivot L/R on the comparison symbol and is confirmed the same way.
- Keep the last two compare pivots of each side. When a chart pivot low is confirmed:
  - Direct: chart LL and compare pivot low within ±tolerance bars by time is not LL → bullish SMT.
  - Inverse: chart LL and compare pivot **high** within tolerance is not HH → bullish SMT.
- If there is no compare pivot within tolerance, or the data is na, the result is N/A.

---

## 5. Setup state machine (bull; bear mirrored)
```
NONE:            if protectedLow exists and htfOk(profile) → CONTEXT_FOUND
CONTEXT_FOUND:   if !htfOk → NONE (CONTEXT_LOST) ; IDM candidate creation → IDM_CANDIDATE (§4.4)
IDM_CANDIDATE:   §4.4
IDM_SWEPT:       §4.6
MSS_CONFIRMED:   collect linked zones: OB with sourceBreakId == mssId; FVG whose middle bar ∈ [sweepBar+1, mssBar]
                 (check again on mssBar+1 for a FVG completing there) → ZONE_WAIT
                 if none found by mssBar+1 → NONE (NO_LINKED_ZONE)
ZONE_WAIT:       if close < protectedLow → NONE (PROTECTED_BROKEN)
                 if all linked zones INVALIDATED/EXPIRED → NONE (ZONE_INVALIDATED / ZONE_EXPIRED)
                 if trigger(§6) on a linked ACTIVE/MITIGATED zone and no bear trigger same bar and profile filters pass
                      → ENTRY_TRIGGERED: draw marker + checklist, alert, then NONE (ENTRY_DONE)
htfOk(Strict)   = HTF off or htfBias == BULL
htfOk(Balanced) = HTF off or htfBias != BEAR
Context-only: the machine runs (for the dashboard) but never draws an entry marker.
```
Both sides run independently. If both reach ENTRY_TRIGGERED on the same candle, neither fires (`CONFLICT`).
Countertrend label: an opposing HTF bias already blocks entries in both Strict and Balanced. So "countertrend" is shown on the dashboard only, when the HTF bias and the chart-TF structure bias disagree.

## 6. Entry triggers
### 6.1 Rejection Candle (default, approved)
Bull: `low <= z.top and close > z.top and close >= low + 0.5*rng and rng > 0`. Bear mirrored with `z.bottom`.

### 6.2 Any enabled trigger
OR of all triggers enabled in inputs.

### 6.3 Proposed — needs approval (F6)
- **Internal MSS trigger (bull):** an internal-structure bullish BOS/CHoCH (any bias) occurs on a candle after the linked zone's first touch, while the zone is still ACTIVE/MITIGATED, and within `triggerMaxBars` (default 10) of that first touch.
- **SFP trigger (bull):** a bullish SFP is confirmed, and its sweep candle's low traded inside a linked zone.

---

## 7. Alerts
- `alertcondition(cond, title, message)` per event type (static text, bull/bear separate). `cond` = "an event of that type was created on this confirmed bar".
- Also one `alert(msg, alert.freq_once_per_bar_close)` with dynamic text (symbol, TF, price, event ID) for users who choose "Any alert() function call".
- Each event ID creates at most one alert.

## 8. Dashboard (last bar only, `table`)
Rows: active profile and its rules · each module TF and status (OK / disabled + reason) · structure bias major/internal · protected high/low · liquidity nearest BSL/SSL · IDM state per side · SMT (symbol, Direct/Inverse, last result or N/A) · SFP volume availability · setup stage per side + last reset reason · entry checklist (✔ / ✘ / N/A).

## 9. Test map
Every test in `07_TEST_REPAINT_AUDIT.md` (1–40) maps to a rule above. For bar-by-bar verification, a `debug` input prints event IDs, stage changes and rejection reasons as labels. The test plan is a manual Bar Replay checklist per test, plus a reload comparison (test 21). Results will be reported as pass / fail / not tested. Compilation is only claimed after it has actually been compiled in TradingView.

## 10. Known Phase 1 limitations
- No lower-TF modules (LTF entry needs Phase 2).
- An HTF module only draws events from the time the chart's history covers the HTF bars. Objects drawn for HTF events use `xloc.bar_time`.
- Tick volume quality differs by broker, so the SFP volume path can be N/A.
- The SMT auto map covers only the listed pairs. Other symbols need Manual mode.
