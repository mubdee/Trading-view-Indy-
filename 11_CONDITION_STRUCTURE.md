# 11 — Condition Structure (โครงสร้างเงื่อนไขรวม)
Version: 0.1 DRAFT สำหรับให้ผู้ใช้ตรวจแก้ | Date: 2026-11-12 | Target: Pine Script v6
แหล่งที่มา: 00–10 รวมคำตอบของผู้ใช้ในไฟล์ 10

> วิธีแก้ไฟล์นี้
> - ทุกเงื่อนไขมีรหัส เช่น `ST-03` แก้ได้ทีละบรรทัด และอ้างรหัสเวลาคุยกันได้
> - `⚠ Qxx` หมายถึงจุดที่ผมต้องเลือกค่าเอง เพราะสเปกหรือคำตอบยังไม่ครอบคลุม (รายการคำถามอยู่ที่ §6)
> - ไฟล์นี้ตั้งใจให้เป็น **Single Source of Truth** ถ้าขัดกับไฟล์ 00–09 ให้ยึดไฟล์นี้

---

## 0. สัญลักษณ์ที่ใช้

| สัญลักษณ์ | ความหมาย |
|---|---|
| `t` | แท่งปัจจุบันที่ **ปิดแล้ว** (ทุกเงื่อนไขประเมินตอนปิดแท่ง) |
| `x[k]` | ค่าของ k แท่งก่อนหน้า |
| `ATR` | `ta.atr(14)` |
| `tol` | `ATR * 0.05` (ใช้กับ equal level) |
| `body` | `abs(close - open)` |
| `range` | `high - low` |
| `bodyTop / bodyBot` | `max(open,close)` / `min(open,close)` |
| `bull candle` | `close > open` |
| `bear candle` | `close < open` |
| `doji` | `close == open` (ค่าเริ่มต้น, ⚠ Q01) |
| `buf_*` | buffer ต่าง ๆ ทุกตัวค่าเริ่มต้น = 0 (ตาม 10-D) |
| `volSpike` | `volume >= ta.sma(volume, 20) * 1.5` (⚠ Q09) |

---

## 1. ปัญหาโครงสร้างของสเปกเดิมที่พบ

| # | ปัญหา | ที่อยู่ | วิธีแก้ในไฟล์นี้ |
|---|---|---|---|
| P1 | **ลำดับ dependency ผิด** Master §9 ให้ BOS/CHoCH/**MSS** (ข้อ 3) มาก่อน Liquidity (4), IDM (5) และ Displacement (6) แต่ MSS ต้องใช้ IDM sweep และ displacement ก่อน จึงคำนวณ MSS ในข้อ 3 ไม่ได้ | 00 §9 | แยก MSS ออกจาก BOS/CHoCH และย้ายไปหลัง IDM (§3) |
| P2 | **State machine ซ้อนกันสองชุด** มีทั้ง Signal lifecycle (00 §8) และ IDM state (02) โดยไม่ระบุว่าเชื่อมกันอย่างไร | 00 §8, 02 | รวมเป็น Setup FSM ชุดเดียว แยกฝั่ง Bull/Bear (§4 M11) |
| P3 | **SFP กับ Sweep เป็นนิยามเดียวกัน** แต่ผู้ใช้ให้นิยาม SFP ใหม่ไว้ใน 10-A3 | 04, 10-A3 | SFP = ขั้นต่อจาก sweep และต้องทะลุ low ของแท่ง high ด้วยเนื้อเทียน (§4 M5) |
| P4 | **Break mode CLOSE/WICK ขัดกับคำตอบ B6** ผู้ใช้ตอบว่าทะลุด้วยไส้ = BOS หลอก, ทะลุด้วยเนื้อ = BOS จริง | 01, 10-B6 | จัดเป็นสองประเภท `BOS` / `FAKE_BOS` (§4 M3) |
| P5 | **Zone ที่ผูกกับ MSS ยังไม่มีเวลาเกิด** FVG ที่มี displacement เป็นแท่งกลางจะรู้ผลที่แท่ง D+1 ไม่ใช่ที่แท่ง D | 03, 10-B5 | กำหนดหน้าต่างเวลาของ zone (§4 M8) |
| P6 | **ไม่มีเงื่อนไขยกเลิกในขั้น ZONE_WAIT** สเปกมีแต่ invalidation ก่อนเกิด MSS | 00 §8 | เพิ่ม reset condition (§4 M11) |
| P7 | **Entry TF ขัดกับ C1** C1 ปิด LTF ไว้ในเฟส 1 แต่ trigger เขียนว่า "LTF trigger" | 06, 10-C1 | เฟส 1 ใช้ trigger บน Chart TF เท่านั้น |
| P8 | **Protected swing อัปเดตเมื่อไร** B3 ให้นิยามไว้แล้ว แต่ยังไม่ระบุกรณีที่ swing low ของขานั้นยังไม่ได้รับการยืนยันตอนเกิด break | 10-B3 | ⚠ Q04 |
| P9 | ไฟล์ซ้ำ `06_CONFLUENCE_ALERTS_UI.md` และ `09_DECISIONS_TO_CONFIRM.md` ผู้ใช้อนุมัติให้ยกเลิกแล้ว (A1/A2) แต่ไฟล์ยังอยู่ | repo | ผู้ใช้ลบเอง หรือสั่งให้ผมลบ |
| P10 | 01 §9 เขียนนิยาม MSS ไว้ในโมดูล Market Structure แต่ไม่มี input ของ IDM | 01 | MSS อยู่ใน M7 ที่เดียว |
| P11 | SMT ไม่มี input Inverse correlation (E1) | 05 | เพิ่มไว้ใน §4 M10 |

---

## 2. กฎกลาง (Global)

- `G-01` ใช้ Pine v6 เป็น indicator ไม่ใช่ strategy
- `G-02` เหตุการณ์ที่ยืนยันแล้วทุกตัวประเมินที่ `barstate.isconfirmed` (หรือที่ bar close ในประวัติ)
- `G-03` ทุกเหตุการณ์มี `id` ที่ไม่ซ้ำ (`bar_index` ของแท่งที่ยืนยัน + ชนิด + ฝั่ง) และ trigger เพียงครั้งเดียว
- `G-04` Pivot เก็บสองเวลา คือ `locBar` (แท่งที่เป็นยอดหรือก้น) และ `confBar = locBar + R` (แท่งที่เริ่มรู้ผล) ทุก logic ใช้ได้ตั้งแต่ `confBar` เป็นต้นไป
- `G-05` ห้ามใช้ `lookahead_on` ที่ไม่มี `[1]` และห้ามใช้ offset ติดลบเพื่อทำให้สัญญาณดูเหมือนมาก่อน
- `G-06` TF ที่ต่ำกว่า chart ให้ปิดโมดูลนั้นและแสดงคำเตือนบน dashboard (10-C1)
- `G-07` ข้อมูลเป็น `na` หรือ history ไม่พอ ให้ถือเป็น `N/A` ไม่ใช่ fail และไม่ออกสัญญาณ
- `G-08` ตั้ง object cap เป็น boxes/lines/labels = 500/500/500 และลบตัวเก่าสุดก่อน (FIFO)
- `G-09` Setup ฝั่ง Bull และ Bear ทำงานแยกกันและอาจ active พร้อมกันได้ ถ้าทั้งสองฝั่ง entry ในแท่งเดียวกัน = `CONFLICT` และไม่ออก entry

---

## 3. ลำดับการคำนวณต่อแท่ง (แก้ P1)

```
 1. M1  Candle utils + ATR + displacement flags + volSpike
 2. M2  ยืนยัน pivot ที่ครบ R แท่งแล้ว (major / internal) → จัดประเภท HH/HL/LH/LL/EQH/EQL
 3.     SNAPSHOT: คัดลอกระดับทุกตัวที่มีอยู่ "ก่อน" แท่ง t (pivot ใหม่ของแท่งนี้ยังไม่เข้า)
 4. M3  BOS / CHoCH / FAKE_BOS (major + internal แยกกัน) ใช้ snapshot
 5.     อัปเดต protected swing
 6. M4  Liquidity pool: ตรวจ sweep / breakout ด้วย snapshot แล้วเพิ่ม pool ใหม่จาก pivot ของแท่งนี้
 7. M5  SFP: ตรวจ pending SFP ที่ค้างอยู่ และสร้าง pending ใหม่จาก sweep
 8. M6  Rejection Block จาก SFP ที่ยืนยันแล้ว
 9. M7  IDM: candidate → sweep → MSS (ใช้ displacement จาก M1)
10. M8  Zone lifecycle ของ zone เดิม (ตรวจก่อน) → สร้าง OB/FVG ใหม่ (ทีหลัง)
11. M9  MTF bias (HTF ที่ปิดแล้ว)
12. M10 SMT
13. M11 Setup FSM → Entry trigger → profile check
14. M12 Alerts → Render → Dashboard / Debug
```

---

## 4. เงื่อนไขรายโมดูล

### M1 — Candle utilities
- `CU-01` Bull displacement: `range > 0` AND `bull candle` AND `body >= ATR * 1.0` AND `close >= low + range * 0.75`
- `CU-02` Bear displacement: `range > 0` AND `bear candle` AND `body >= ATR * 1.0` AND `close <= low + range * 0.25`
- `CU-03` `volSpike` ตามตาราง §0 ใช้กับ SFP (M5) ⚠ Q09: Forex และ XAU บน TradingView มีแค่ tick volume

### M2 — Pivots
| รหัส | เงื่อนไข |
|---|---|
| `PV-01` | Major pivot high: `ta.pivothigh(high, 3, 3)` ไม่เป็น na ที่แท่ง t → `price = high[3]`, `locBar = t-3`, `confBar = t` |
| `PV-02` | Major pivot low: `ta.pivotlow(low, 3, 3)` ใช้กฎเดียวกัน |
| `PV-03` | Internal pivot: L/R = 1/1 กฎเดียวกัน และเก็บแยก array |
| `PV-04` | จัดประเภท high เทียบกับ pivot high ที่ยืนยันก่อนหน้า (ชนิดเดียวกัน): `abs(diff) <= tol` → `EQH`, `>` → `HH`, `<` → `LH` (เช็ค EQ ก่อน) |
| `PV-05` | Low ใช้กฎเดียวกัน: `EQL` / `HL` / `LL` |
| `PV-06` | กรณีมี high เท่ากันหลายแท่ง (plateau) ให้ใช้พฤติกรรมของ `ta.pivothigh` ใน Pine ⚠ Q02 |
| `PV-07` | เก็บ `low` ของแท่ง pivot high ไว้ด้วย (`pivLow`) และเก็บ `high` ของแท่ง pivot low (`pivHigh`) เพราะ SFP ต้องใช้ (M5) |

### M3 — Market Structure (BOS / CHoCH / FAKE_BOS)
ระดับเป้าหมาย (ใช้ค่าจาก snapshot):
- `ST-01` `tgtHigh` = pivot high ล่าสุดที่ยืนยันแล้วและยังไม่ถูกใช้ (unconsumed)
- `ST-02` `tgtLow` = pivot low ล่าสุดที่ยืนยันแล้วและยังไม่ถูกใช้

นิยามการทะลุ:
- `ST-03` Body break ขึ้น: `close > tgtHigh + buf_break`
- `ST-04` Wick-only break ขึ้น: `high > tgtHigh + buf_break` AND `close <= tgtHigh + buf_break`
- `ST-05` ขาลงใช้กฎเดียวกันกลับด้าน

การจัดประเภท (แก้ P4 ตามคำตอบ B6):

| Bias ก่อนแท่ง t | เหตุการณ์ | ผลลัพธ์ | Bias ใหม่ | ระดับถูกใช้? |
|---|---|---|---|---|
| NEUTRAL | Body break ขึ้น | `BOS ↑` (จริง) | BULL | ใช่ |
| NEUTRAL | Wick-only break ขึ้น | `FAKE_BOS ↑` | NEUTRAL | ไม่ (⚠ Q03) |
| BULL | Body break ขึ้น | `BOS ↑` | BULL | ใช่ |
| BULL | Body break ลง | `CHoCH ↓` | BEAR | ใช่ |
| BEAR | Body break ลง | `BOS ↓` | BEAR | ใช่ |
| BEAR | Body break ขึ้น | `CHoCH ↑` | BULL | ใช่ |
| ใด ๆ | Wick-only (bias ≠ NEUTRAL) | ไม่ใช่ structure event (ส่งต่อให้ M4 เป็น sweep) | เดิม | ไม่ (⚠ Q03) |
| ใด ๆ | ทะลุทั้งสองฝั่งในแท่งเดียว | `AMBIGUOUS` | เดิม | ไม่ |

- `ST-06` Input `Break mode = WICK` จะนับ wick-only break เป็น BOS/CHoCH ด้วย (ค่าเริ่มต้น = CLOSE)
- `ST-07` Pivot ที่ยืนยันที่แท่ง t ยังไม่อยู่ใน snapshot จึงทะลุได้ตั้งแต่แท่ง t+1
- `ST-08` Major กับ Internal คำนวณอิสระกัน มี `biasMajor` และ `biasInt` แยกกัน
- `ST-09` Gap ข้ามระดับ (open อยู่เลยระดับไปแล้ว) ยังนับเป็น body break ถ้า close เลยระดับ

Protected swing (B3):
- `ST-10` เมื่อเกิด `BOS↑/CHoCH↑` ของ major: `protLow` = major pivot low ล่าสุดที่ยืนยันแล้ว และ `locBar < breakBar` ⚠ Q04
- `ST-11` ถ้ายังไม่เคยมี break ขึ้น: `protLow` = major pivot low ล่าสุดที่ยืนยันแล้ว
- `ST-12` `protHigh` ใช้กฎเดียวกันกลับด้าน
- `ST-13` เมื่อ `protLow` เปลี่ยน ให้ส่ง event `PROT_CHANGED_BULL` ไปที่ M7 และ M11

### M4 — Liquidity pools & sweeps
- `LQ-01` แหล่งของ pool (เปิดปิดแยกกัน): major pivot, internal pivot, EQH/EQL, PDH/PDL, PWH/PWL, session (ค่าเริ่มต้น = ปิด)
- `LQ-02` PDH/PDL: `request.security(syminfo.tickerid, "D", high[1], lookahead = barmerge.lookahead_on)` ปิดอัตโนมัติเมื่อ chart TF ≥ D (C2) ส่วน PWH/PWL ใช้ "W" และปิดอัตโนมัติเมื่อ chart TF ≥ W
- `LQ-03` Pool = `{id, price, side(BSL/SSL), source, createdBar, touches, state}` โดย state มี `ACTIVE / SWEPT / BROKEN / EXPIRED`
- `LQ-04` Merge: ถ้าระดับใหม่อยู่ห่างจาก pool ACTIVE ฝั่งเดียวกันไม่เกิน `tol` ให้ `touches += 1`, ติดป้าย EQ และให้ราคา = ค่าที่สุดโต่งกว่า (BSL ใช้ max, SSL ใช้ min)
- `LQ-05` **Sweep SSL (bullish context)**: `low < pool - buf_sweep` AND `close > pool` → pool = SWEPT และบันทึก `{poolId, penetration = pool - low, reclaim = close - pool, t}`
- `LQ-06` **Breakout SSL**: `close < pool - buf_sweep` → pool = BROKEN (ไม่ใช่ sweep)
- `LQ-07` BSL ใช้กฎเดียวกันกลับด้าน
- `LQ-08` ถ้าแท่งเดียว sweep หลาย pool: pool หลักคือตัวที่ราคาไปถึงไกลสุด (SSL ที่ต่ำสุด) ส่วนตัวอื่นแสดงใน debug
- `LQ-09` ถ้าแท่งเดียว sweep ทั้ง BSL และ SSL = `AMBIGUOUS` และไม่มี sweep ฝั่งใด
- `LQ-10` Pool อายุเกิน 500 แท่ง → EXPIRED

### M5 — SFP (นิยามใหม่ตาม 10-A3)
> คำตอบผู้ใช้: "แท่งขึ้นไปสวีปไฮ แล้วกลับมาทะลุ **โลของไฮนั้น** ด้วยเนื้อเทียน ถ้าทะลุด้วยไส้ไม่นับ ยกเว้นทะลุด้วยไส้แล้วแท่งถัดไปมีวอลุ่มรุนแรง"

Bearish SFP:
| รหัส | เงื่อนไข |
|---|---|
| `SF-01` | อ้างอิง swing high `H` ที่ active และยืนยันก่อนแท่ง t โดย `L_ref` = low ของแท่ง pivot high นั้น (`pivLow`, PV-07) ⚠ Q05 |
| `SF-02` | แท่ง sweep `S`: `high[S] > H + buf_sweep` → สร้าง `SFP_PENDING` (ไม่บังคับว่าแท่ง S ต้องปิดกลับใต้ H) ⚠ Q06 |
| `SF-03` | **ยืนยันแบบเนื้อเทียน**: ภายใน `N_sfp` แท่งนับจาก S (รวม S) มีแท่ง C ที่ `close < L_ref - buf_sweep` → `SFP ↓ CONFIRMED` ที่แท่ง C ⚠ Q07 (ค่าเริ่มต้น `N_sfp = 5`) |
| `SF-04` | **ยืนยันแบบไส้ + volume**: แท่ง C มี `low < L_ref` AND `close >= L_ref` (ทะลุแค่ไส้) แล้วแท่ง C+1 มี `volSpike` → `SFP ↓ CONFIRMED` ที่แท่ง C+1 ⚠ Q08 |
| `SF-05` | ทะลุด้วยไส้แต่แท่งถัดไปไม่มี volSpike → ไม่นับ และยังรอ SF-03 ต่อได้จนหมดหน้าต่าง |
| `SF-06` | Pending ถูกยกเลิกเมื่อ (a) หมดหน้าต่าง `N_sfp` หรือ (b) มีแท่งปิดเหนือ `high[S]` ⚠ Q06 |
| `SF-07` | หนึ่ง swing high สร้าง SFP ได้ครั้งเดียว และ SFP อ้าง `sweepId` ของ M4 ถ้ามี (แก้ A3) |

- Bullish SFP ใช้กฎเดียวกันกลับด้าน: sweep ใต้ swing low แล้วปิดทะลุ **high ของแท่ง pivot low** ขึ้นไปด้วยเนื้อเทียน

### M6 — Rejection Block (B7 = แบบ a)
- `RB-01` สร้างเมื่อเกิด `SFP CONFIRMED` เท่านั้น ⚠ Q10 แท่งต้นทางคือแท่ง sweep `S`
- `RB-02` Bearish RB (Wick mode, ค่าเริ่มต้น): `[bodyTop[S], high[S]]`
- `RB-03` Bullish RB (Wick mode): `[low[S], bodyBot[S]]`
- `RB-04` โหมดอื่นใน input: `Body` = `[bodyBot, bodyTop]` และ `Full` = `[low, high]`
- `RB-05` เริ่มใช้งานได้ที่แท่งยืนยัน SFP แต่วาดย้อนไปที่ตำแหน่งแท่ง S ได้ (แค่ภาพ ไม่ใช่เวลาที่รู้ผล)
- `RB-06` Invalidation: Bearish `close > high[S] + buf_zone` / Bullish `close < low[S] - buf_zone`
- `RB-07` Lifecycle เหมือน zone (ZN-01..06)

### M7 — IDM + MSS (Bullish ส่วน Bearish ใช้กฎเดียวกันกลับด้าน)
| รหัส | เงื่อนไข |
|---|---|
| `ID-01` | ต้องมี `protLow` อยู่ก่อน ถ้าไม่มีจะไม่สร้าง candidate |
| `ID-02` | Internal pivot low ยืนยันที่แท่ง t AND `price > protLow` → `IDM_CANDIDATE` (เริ่มใช้ได้ตั้งแต่แท่ง t) |
| `ID-03` | ถ้ามีหลาย candidate ให้ใช้ตัวล่าสุดที่ยังไม่ถูกใช้ ตัวอื่นแสดงใน debug |
| `ID-04` | **Sweep**: `low < idm - buf_sweep` AND `close > idm` บนแท่งเดียวกัน → `IDM_SWEPT` (บันทึก `sweepBar`) |
| `ID-05` | (Option ปิดไว้) Reclaim ภายใน N แท่ง: low ทะลุแล้วมีแท่งปิดเหนือ idm ภายใน N แท่ง |
| `ID-06` | ทะลุแล้วปิดใต้ idm และไม่ reclaim → candidate = `INVALIDATED (IDM_ACCEPTED)` ⚠ Q11 |
| `ID-07` | ตอน sweep ให้ล็อก `mssLevel` = internal pivot high ล่าสุดที่ยืนยันแล้ว และ `locBar < sweepBar` (B1, input เลือก Internal/Major ได้) ถ้าไม่มี → `INVALIDATED (NO_MSS_LEVEL)` |
| `ID-08` | **MSS**: แท่ง `t > sweepBar` AND `close > mssLevel + buf_break` AND แท่ง t เป็น Bull displacement (CU-01, window N=0 ตาม B2) → `MSS_CONFIRMED` (บันทึก `mssBar = D = t`) |
| `ID-09` | ทะลุ mssLevel แต่ไม่ใช่ displacement → ยังไม่ใช่ MSS ให้รอต่อ และ log `BREAK_NO_DISP` ⚠ Q12 |
| `ID-10` | Sweep กับ break เกิดในแท่งเดียวกัน → ไม่นับ และ `INVALIDATED (SAME_BAR)` |
| `ID-11` | ก่อนเกิด MSS ถ้า `close < protLow - buf_break` → `INVALIDATED (PROT_BROKEN)` |
| `ID-12` | `PROT_CHANGED_BULL` → `EXPIRED (PROT_CHANGED)` |
| `ID-13` | อายุ candidate เกิน 100 แท่ง (นับจาก ID-02) → `EXPIRED (MAX_BARS)` |
| `ID-14` | Internal break ขึ้นที่เกิดขณะยังเป็น CANDIDATE (ยังไม่ sweep) ไม่นับเป็น MSS |

### M8 — OB / FVG
OB ที่ผูกกับ MSS (B5):
- `OB-01` ค้นย้อนจากแท่ง D: `k = 1..5` หาแท่ง **bear candle** ล่าสุด (สำหรับ Bull OB) ข้ามแท่ง doji ⚠ Q01
- `OB-02` Bull OB = `[low[src], open[src]]` และ Bear OB = `[open[src], high[src]]` (Hybrid ที่ล็อกไว้แล้ว)
- `OB-03` `mid = (top + bottom) / 2` และมีเส้น 50% เป็น option
- `OB-04` สร้างที่แท่ง D แต่ retest ได้ตั้งแต่ D+1
- `OB-05` ไม่จำกัดว่า src ต้องอยู่หลัง sweepBar ⚠ Q13
- `OB-06` ไม่พบแท่งสีตรงข้ามภายใน 5 แท่ง → log `NO_OB_SOURCE` (setup ยังใช้ FVG ได้)

FVG:
- `FV-01` Bull FVG ที่แท่ง t: `low > high[2]` → zone `[high[2], low]` | Bear FVG: `high < low[2]` → `[high, low[2]]`
- `FV-02` `size >= ATR * 0.05` (และอย่างน้อย 1 tick)
- `FV-03` FVG ที่ผูกกับ MSS = FVG ทิศเดียวกันที่แท่งกลาง (t-1) อยู่ในช่วง `[sweepBar, D]` จึงรู้ผลได้ช้าสุดที่แท่ง D+1 (แก้ P5)
- `FV-04` FVG อื่นที่ไม่ผูกกับ MSS ยังวาดเป็น context ได้ แต่ใช้ entry ไม่ได้

Zone lifecycle (OB / FVG / RB) ตรวจทุกแท่ง **ก่อน** สร้าง zone ใหม่:
- `ZN-01` First touch (Bull): `low <= top` (ตั้งครั้งเดียว)
- `ZN-02` Fill % (Bull) = `clamp((top - low) / (top - bottom), 0, 1)` วัดจากขอบ proximal
- `ZN-03` Mitigated เมื่อถึงเกณฑ์ (ค่าเริ่มต้น = Touch, ตัวเลือก: 50% / Full)
- `ZN-04` Invalidated (Bull): `close < bottom - buf_zone`
- `ZN-05` Expired: อายุเกิน 300 แท่ง
- `ZN-06` Zone ที่ invalidated/expired แสดงต่อได้ด้วยสีจาง (option) แต่ใช้ entry ไม่ได้

### M9 — MTF Context
- `MT-01` `htfBias = request.security(sym, htfTF, biasMajor[1], lookahead = barmerge.lookahead_on)` ใช้แค่ค่าของแท่ง HTF ที่ปิดแล้ว
- `MT-02` `htfTF = Off` → bias = N/A (ข้ามเงื่อนไข HTF)
- `MT-03` `htfTF < chart TF` → ปิดและเตือน (G-06)
- `MT-04` ข้อมูล na หรือไม่พอ → NEUTRAL

### M10 — SMT
- `SM-01` คู่เทียบอัตโนมัติ (input `SMT symbol = Auto` หรือพิมพ์เอง):

| Chart | Comparison (ค่าเริ่มต้น) | Inverse |
|---|---|---|
| XAUUSD | `<prefix>:XAGUSD` | ไม่ |
| XAUUSD (ทางเลือก) | `TVC:DXY` | ใช่ |
| EURUSD | `<prefix>:GBPUSD` | ไม่ |
| GBPUSD | `<prefix>:EURUSD` | ไม่ |
| AUDUSD | `<prefix>:NZDUSD` | ไม่ |
| NZDUSD | `<prefix>:AUDUSD` | ไม่ |
| USDCHF | `<prefix>:EURUSD` | ใช่ |
| USDJPY / USDCAD | `TVC:DXY` | ไม่ |
| คู่อื่น | ต้องพิมพ์เอง ถ้าไม่พิมพ์ = N/A | – |

  `<prefix>` = `syminfo.prefix` ของกราฟ ส่วนการจับคู่ใช้ `syminfo.basecurrency + syminfo.currency` เพื่อรองรับ suffix ของโบรกเกอร์ ⚠ Q14
- `SM-02` เพิ่ม input `Inverse correlation` (ค่าเริ่มต้นตามตาราง) ถ้าเป็นคู่ inverse ให้เทียบ low ของกราฟกับ high ของคู่เทียบ
- `SM-03` ดึง pivot ของคู่เทียบด้วย `request.security(comp, timeframe.period, [pivotPrice, pivotLocTime, prevPivotPrice])` ไม่ใช้ lookahead และใช้ L/R เดียวกับ major
- `SM-04` **Bull SMT**: กราฟยืนยัน major `LL` และคู่เทียบมี pivot low ที่ `|locTime ต่างกัน| <= 2 แท่ง` และไม่ได้ทำ LL (`>= prev`) → `SMT ↑` ที่แท่งที่ทั้งสองฝั่งยืนยันแล้ว
- `SM-05` Bear SMT ใช้กฎเดียวกันกลับด้านที่ HH
- `SM-06` คู่เทียบไม่มีข้อมูลหรือเป็น na → `N/A` (ไม่นับว่า fail)

### M11 — Setup FSM + Entry
State (ต่อฝั่ง): `NONE → CONTEXT_FOUND → IDM_CANDIDATE → IDM_SWEPT → MSS_CONFIRMED → ZONE_WAIT → ENTRY_TRIGGERED → NONE`

| รหัส | จาก → ไป | เงื่อนไข |
|---|---|---|
| `FS-01` | NONE → CONTEXT_FOUND | มี `protLow` AND HTF ผ่านตามโปรไฟล์ (B4) |
| `FS-02` | CONTEXT → IDM_CANDIDATE | ID-02 |
| `FS-03` | IDM_CANDIDATE → IDM_SWEPT | ID-04 |
| `FS-04` | IDM_SWEPT → MSS_CONFIRMED | ID-08 |
| `FS-05` | MSS_CONFIRMED → ZONE_WAIT | แท่งถัดไป (D+1) และมี zone ที่ผูกกับ MSS ≥ 1 (OB หรือ FVG) ถ้าไม่มีเลย → reset `NO_ZONE` |
| `FS-06` | ZONE_WAIT → ENTRY_TRIGGERED | zone ที่ผูกกับ MSS ถูกแตะ AND trigger ผ่าน AND profile ผ่าน |
| `FS-07` | ENTRY → NONE | ทันที, reason `ENTRY_DONE` (1 entry ต่อ setup, B9) |

Reset (ทุก state → NONE พร้อม reason):
- `RS-01` ID-06, ID-07, ID-10, ID-11, ID-12, ID-13
- `RS-02` ใน ZONE_WAIT: zone ที่ผูกกับ MSS ทุกตัว invalidated/expired → `ZONE_INVALID`
- `RS-03` ใน ZONE_WAIT: `close < protLow - buf_break` → `PROT_BROKEN` ⚠ Q15 (protLow ใช้ตัวไหนหลัง MSS)
- `RS-04` ใน ZONE_WAIT: เกิด MSS ฝั่งตรงข้าม → `OPPOSITE_MSS`
- `RS-05` ZONE_WAIT นานเกิน `maxWait` แท่ง (input ใหม่ ค่าเริ่มต้น 50) → `WAIT_EXPIRED` ⚠ Q16

Entry trigger (B8 ทำงานบน Chart TF ในเฟส 1 ตาม P7):
- `TR-01` **Rejection Candle (ค่าเริ่มต้น)** Bull: `low <= zoneTop` AND `close > zoneTop` AND `close >= low + range * 0.5`
- `TR-02` **Internal MSS**: เกิด internal `BOS↑/CHoCH↑` (body break) ภายใน `N` แท่งหลังจากแตะ zone ⚠ Q17
- `TR-03` **SFP**: เกิด `SFP ↑ CONFIRMED` โดยแท่ง sweep `S` แตะ zone
- `TR-04` **Any enabled**: TR ใดก็ได้ที่เปิดไว้

Profile:
| โปรไฟล์ | HTF | IDM sweep | MSS+Disp | Zone | Trigger |
|---|---|---|---|---|---|
| Strict | ต้องตรงทิศ (neutral = fail) | ต้องมี | ต้องมี | OB **และ/หรือ** FVG ⚠ Q18 | ต้องมี |
| Balanced | ตรงทิศหรือ neutral ⚠ Q19 | ต้องมี | ต้องมี | OB หรือ FVG | ต้องมี 1 |
| Context-only | – | – | – | – | ไม่มี entry marker |

- `PF-01` ถ้าปิด IDM module ให้เลือกเส้นทางสำรองได้: liquidity sweep (LQ-05) → MSS (ใช้ ID-07..10 โดยแทน IDM ด้วย pool ที่ถูก sweep) และห้าม bypass เองขณะ IDM ยังเปิดอยู่
- `PF-02` ถ้า HTF ตรงข้ามแต่ profile ยอมให้ผ่าน → ติดป้าย `COUNTERTREND`
- `PF-03` Score = จำนวนเงื่อนไขที่ผ่าน/ทั้งหมด แสดงเป็น checklist ไม่ใช่ความน่าจะเป็น

### M12 — Alerts / Render / Dashboard
- `AL-01` `alertcondition()`: BOS, CHoCH, FAKE_BOS, MSS, Sweep, IDM candidate, IDM swept, SFP, New OB, New FVG, Mitigation, RB, SMT, Entry Bull, Entry Bear
- `AL-02` ทุก alert ใช้ flag ที่เกิดที่ bar close และเกิดครั้งเดียวต่อ event id
- `UI-01` Dashboard แสดง TF ของแต่ละโมดูล, HTF bias, biasMajor/Int, protLow/High, IDM state (ทั้งสองฝั่ง), zone state, SMT (OK/N/A), checklist และคำเตือน TF
- `UI-02` Debug mode แสดง reason code ล่าสุดของแต่ละฝั่ง

---

## 5. Reason codes
`ENTRY_DONE` `IDM_ACCEPTED` `NO_MSS_LEVEL` `BREAK_NO_DISP` `SAME_BAR` `PROT_BROKEN` `PROT_CHANGED` `MAX_BARS` `NO_ZONE` `NO_OB_SOURCE` `ZONE_INVALID` `OPPOSITE_MSS` `WAIT_EXPIRED` `CONFLICT` `AMBIGUOUS` `HTF_FAIL` `TF_LOWER_THAN_CHART` `DATA_NA`

---

## 6. คำถามที่ยังเปิด (ค่าในวงเล็บคือค่าที่ผมใช้ไปก่อน)

ตอบได้ในรูปแบบเดิม: `[x] ตามที่แนะนำ` หรือเขียนใน `คำตอบ:`

| Q | คำถาม | ค่าที่ใช้ไปก่อน | ตอบ |
|---|---|---|---|
| Q01 | Doji คืออะไร: `open == close` เท่านั้น หรือ `body <= range * X%` และใน OB search ให้ข้าม doji แล้วหาต่อ หรือยกเลิก OB เลย | `open==close` และข้ามแล้วหาต่อ | [ ] |
| Q02 | ถ้า high เท่ากันสองแท่ง (plateau) นับเป็น pivot หรือไม่ | ตามพฤติกรรม `ta.pivothigh` | [ ] |
| Q03 | FAKE_BOS ใช้เฉพาะตอน bias = NEUTRAL หรือทุก bias และหลังจาก FAKE_BOS แล้วระดับนั้นยังทะลุซ้ำเป็น BOS จริงได้หรือไม่ | เฉพาะ NEUTRAL และยังทะลุซ้ำได้ | [ ] |
| Q04 | Protected low ตอนเกิด break ถ้า swing low ของขานั้นยังไม่ได้รับการยืนยัน (ต้องรอ R แท่ง) ให้ใช้ low ล่าสุดที่ยืนยันแล้วไปก่อน แล้วอัปเดตเมื่อยืนยัน หรือใช้ `lowest low` ระหว่าง swing high ที่ถูกทะลุกับแท่ง break | ใช้ตัวยืนยันล่าสุดไปก่อน | [ ] |
| Q05 | "โลของไฮนั้น" ใน SFP หมายถึง (a) low ของ **แท่ง pivot high** หรือ (b) low ของ **แท่งที่ขึ้นไป sweep** | (a) | [ ] |
| Q06 | SFP: แท่ง sweep ต้องปิดกลับใต้ high เดิมด้วยหรือไม่ และถ้ามีแท่งปิดเหนือ high ของแท่ง sweep ให้ยกเลิกหรือไม่ | ไม่บังคับปิดกลับ / ยกเลิก | [ ] |
| Q07 | SFP: การทะลุ low ด้วยเนื้อเทียนต้องเกิดภายในกี่แท่งหลัง sweep และนับเฉพาะ `close` ทะลุ หรือต้อง `open` อยู่เหนือและ `close` อยู่ใต้ (ตัวแท่งตัดผ่าน) | 5 แท่ง และนับแค่ close | [ ] |
| Q08 | SFP แบบไส้ + volume: แท่งถัดไปที่มี volume แรงต้องเป็นแท่งสีตามทิศด้วยหรือไม่ (เช่น bearish SFP ต้องเป็นแท่งแดง) | ไม่บังคับ | [ ] |
| Q09 | "วอลุ่มรุนแรง" กำหนดอย่างไร (Forex/XAU มีแค่ tick volume) | `vol >= SMA(vol,20) * 1.5` | [ ] |
| Q10 | Rejection Block สร้างจาก SFP เท่านั้น หรือจาก sweep ธรรมดา (ไส้ทะลุแล้วปิดกลับ) ด้วย | SFP เท่านั้น | [ ] |
| Q11 | IDM ถูกทะลุแล้วปิดใต้ (ไม่ reclaim) ให้ยกเลิก candidate หรือไม่ | ยกเลิก | [ ] |
| Q12 | ปิดทะลุ mssLevel แต่แท่งนั้นไม่ใช่ displacement: ให้รอแท่ง displacement ถัดไปที่ปิดเหนือระดับ หรือยกเลิก setup | รอต่อ | [ ] |
| Q13 | OB source ต้องอยู่หลังแท่ง IDM sweep หรือไม่ | ไม่บังคับ | [ ] |
| Q14 | คู่ SMT ในตาราง SM-01 ถูกต้องหรือไม่ (โบรกเกอร์ของคุณใช้ prefix อะไร) | ตามตาราง | [ ] |
| Q15 | หลัง MSS แล้ว protected low คือตัวเดิม หรือเลื่อนไปเป็น low ของขา MSS (ซึ่งมักเป็น low ของ IDM sweep) | low ของ IDM sweep | [ ] |
| Q16 | ZONE_WAIT รอได้นานสุดกี่แท่ง | 50 | [ ] |
| Q17 | Trigger Internal MSS ต้องเกิดภายในกี่แท่งหลังแตะ zone | 5 | [ ] |
| Q18 | Strict ต้องมีทั้ง OB และ FVG หรือมีอย่างใดอย่างหนึ่งก็พอ | อย่างใดอย่างหนึ่ง | [ ] |
| Q19 | Balanced: ถ้า HTF ตรงข้ามให้ block หรือผ่านแต่ติดป้าย COUNTERTREND | ผ่าน + COUNTERTREND | [ ] |

---

## 7. ไฟล์เดิมที่ไฟล์นี้แทนที่
- `06_CONFLUENCE_ALERTS_UI.md` และ `09_DECISIONS_TO_CONFIRM.md` → ยกเลิกแล้ว (A1/A2) รอลบ
- `00_MASTER.md` §8 และ §9 → ใช้ §3 และ M11 ของไฟล์นี้แทน
- `04_SFP_REJECTION_BLOCK.md` นิยาม SFP → ใช้ M5 แทน
- `01_MARKET_STRUCTURE.md` ข้อ 5 และข้อ 9 → ใช้ M3 และ M7 แทน

## 8. Test เพิ่มเติม (ต่อจาก 07)
25. FAKE_BOS เกิดเฉพาะกรณี wick-only และ bias ไม่เปลี่ยน
26. SFP ไม่ยืนยันเมื่อทะลุ L_ref แค่ไส้และแท่งถัดไปไม่มี volSpike
27. SFP ยืนยันที่แท่ง C+1 เมื่อ C ทะลุแค่ไส้และ C+1 มี volSpike
28. FVG ที่ผูกกับ MSS ยืนยันที่ D+1 ไม่ใช่ D
29. MSS ที่ไม่ใช่ displacement ไม่เลื่อน state
30. Setup reset เมื่อ zone ที่ผูกกับ MSS ถูก invalidate ทั้งหมด
31. SMT = N/A เมื่อคู่เทียบไม่มีข้อมูล และไม่ทำให้ Strict fail
