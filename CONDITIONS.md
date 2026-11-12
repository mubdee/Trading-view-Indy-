# SMC Indicator — โครงสร้างเงื่อนไขทั้งหมด (Single Source of Truth)
Version: 3.0 DRAFT ให้ผู้ใช้ตรวจแก้ | Date: 2026-11-12 | Target: TradingView Pine Script v6 (indicator)

> **วิธีใช้ไฟล์นี้**
> - ทุกเงื่อนไขมีรหัส เช่น `ST-07` แก้ได้ทีละบรรทัด และอ้างรหัสเวลาสั่งแก้โค้ดได้
> - `⚠ Qxx` = จุดที่ยังต้องให้คุณยืนยัน (ดูรายการที่ §16) ค่าที่เขียนไว้คือค่าที่จะใช้ไปก่อน
> - ถ้าอยากเปลี่ยนเงื่อนไขข้อไหน ให้เขียนต่อท้ายข้อนั้นว่า `แก้ไข: ...`
> - ไฟล์นี้แทนที่ไฟล์สเปกเดิมทั้งหมด (00–13) ไฟล์เก่ายังดูได้ใน git history และ branch เดิม

---

## 0. ที่มาของไฟล์นี้ และสิ่งที่ผิดในชุดเดิม

ไฟล์นี้รวมงานที่แยกกันอยู่ 3 สาย:

| สาย | สิ่งที่มี | ปัญหา |
|---|---|---|
| `claude/open-issues-review` (สเปก v1.1 + คำตอบใน 10) | สเปกเดิม + คำตอบ A–E ของผู้ใช้ | ยังเป็น MTF ลำดับการคำนวณผิด (MSS ถูกวางก่อน IDM และ displacement) |
| `claude/apply-open-issues-v1-2` (โค้ด `SMC_Suite.pine` v1.5) | โครงสร้างแบบ LuxAlgo Inducement, OB A+, Demand/Supply, จุดเข้าเป็นกล่อง, เส้น SMT ตามคำสั่ง "แก้ไข:" ของผู้ใช้ | ยังมี HTF context |
| `single-timeframe` / `main` (โค้ด `SMC_SingleTF.pine` v2.0) | เหลือ TF เดียวตามคำขอผู้ใช้ | **สร้างจากสเปกเก่า จึงทำให้การแก้ v1.3–v1.5 หายไปหมด** ทั้งโครงสร้าง IDM ใน CHoCH/BOS, OB A+, D/S และกล่องจุดเข้า (เป็นเหตุให้โครงสร้างผิด) |

**การรวมในไฟล์นี้:** ใช้ TF เดียว (คำขอล่าสุด) + โครงสร้างและโซนแบบ v1.5 (คำสั่ง "แก้ไข:" ของผู้ใช้) + การตัดสินใจทั้งหมดจากไฟล์ 10

ปัญหาเชิงตรรกะที่แก้ไว้ในไฟล์นี้:
1. **MSS มีสองนิยามที่ขัดกัน** v2.0 ใช้ "IDM sweep แล้วทะลุ internal swing ด้วย displacement" ส่วน v1.5 ใช้ "BOS (หลังกิน IDM) หรือ CHoCH ที่เป็นแท่ง displacement" → ไฟล์นี้ใช้แบบ v1.5 เพราะผู้ใช้สั่งให้ IDM อยู่ในโครงสร้าง CHoCH/BOS (⚠ Q01)
2. **IDM มีสองที่** (โมดูลโครงสร้างกับโมดูล setup) → ให้มีที่เดียวคือใน Structure Engine (§6)
3. **FVG กับ Demand/Supply** ผู้ใช้สั่งให้เอา FVG ออกและใช้ D/S แทน → FVG เหลือแค่ "imbalance" ที่ใช้คัด OB A+ (§8)
4. **SFP กับ Liquidity Sweep ซ้ำกัน** → Sweep = เหตุการณ์แรก ส่วน SFP = การยืนยันหลัง sweep (§9) แก้ไข: SFP=การยืนยันหลังsweepแล้วมีแท่งที่เป็นเนื้อเทียนปิดต่ำกว่าโลล่าสุดของsweepถ้าปิดด้วยไส้เทียนจะไม่นับเป็นSFP
5. **ไม่มีเงื่อนไขยกเลิกระหว่างรอโซน** → เพิ่มไว้ที่ §11
6. **HTF / MTF** → ตัดออกทั้งหมด ใช้ bias ของโครงสร้างบนกราฟแทน

---

## 1. สัญลักษณ์

| สัญลักษณ์ | ความหมาย |
|---|---|
| `t` | แท่งปัจจุบันที่ **ปิดแล้ว** |
| `x[k]` | ค่าของ k แท่งก่อนหน้า |
| `ATR` | `ta.atr(14)` |
| `tol` | `ATR × 0.05` (ใช้กับ equal level) |
| `body` / `range` | `abs(close-open)` / `high-low` |
| `bodyTop` / `bodyBot` | `max(open,close)` / `min(open,close)` |
| `bull` / `bear` / `doji` | `close>open` / `close<open` / `close==open` |
| `buf_brk`, `buf_swp`, `buf_inv`, `buf_rb` | buffer ทุกตัวค่าเริ่มต้น = `0 × ATR` |
| `volSpike` | `volume >= ta.sma(volume, 20) × 1.5` |

---

## 2. กฎกลาง (บังคับทุกโมดูล)

- `G-01` Pine Script v6 เป็น **indicator** ไม่ใช่ strategy
- `G-02` **ใช้ Timeframe เดียว** คือ TF ของกราฟ ไม่มี HTF/LTF/MTF และไม่มี input เลือก TF ถ้าจะดู TF อื่นให้เปลี่ยน TF ของกราฟเอง
- `G-03` ทุก logic ทำงานตอนแท่งปิด (`barstate.isconfirmed`) และไม่มี intrabar preview
- `G-04` ห้าม future leak ห้าม repaint ห้ามใช้ offset ติดลบ และห้ามใช้ `lookahead_on`
- `G-05` Pivot ที่มี R แท่งด้านขวาจะ **รู้ผลที่แท่ง pivot + R** โดยวาดที่แท่ง pivot ได้ แต่ห้าม logic ใดใช้ก่อนแท่งยืนยัน
- `G-06` `request.security()` ใช้ได้เฉพาะดึงสัญลักษณ์ SMT บน `timeframe.period` (lookahead off, `ignore_invalid_symbol = true`)
- `G-07` ทุกเหตุการณ์มี ID เกิดครั้งเดียว และทุกการ reset ต้องมี reason code
- `G-08` ข้อมูล `na` หรือ history ไม่พอ ให้ถือเป็น `N/A` ไม่ใช่ fail และไม่ออกสัญญาณ
- `G-09` ตั้ง object cap เป็น boxes/lines/labels = 500/500/500 ลบตัวเก่าสุดก่อน และติดตาม pool ได้สูงสุด 80 ตัว zone สูงสุด 150 ตัว
- `G-10` ทุก threshold ต้องเป็น input ห้ามอ้าง win rate และห้ามอ้างว่า compile ผ่านถ้ายังไม่ได้ทดสอบใน TradingView
- `G-11` ห้ามคัดลอกโค้ดของอินดิเคเตอร์อ้างอิง (LuxAlgo, Milana Trades, JP7FX) ให้เขียนเองจากหลักการที่เผยแพร่

---

## 3. Inputs และค่าเริ่มต้น

| กลุ่ม | Input | ค่าเริ่มต้น |
|---|---|---|
| 0 General | Profile | `Balanced` (Strict / Balanced / Context-only) ⚠ Q10 |
| | Entry trigger | `Rejection Candle` (Rejection Candle / Internal MSS / SFP / Any) |
| | Entry zone | `OB A+ or D/S` (OB A+ or D/S / OB A+ only / D/S only / OB A+ overlapping D/S) |
| | Internal MSS trigger: max bars after first touch | 10 |
| | Entry zone length | 20 แท่ง |
| | Dashboard / Debug / Checklist tooltip | on / off / off |
| 1 Structure | CHoCH swing length (major) | 5 ⚠ Q02 |
| | IDM swing length (internal) | 2 ⚠ Q02 |
| | IDM ต้องปิดกลับ (reclaim) | off ⚠ Q03 |
| | Break buffer / Equal tolerance | 0 × ATR / 0.05 × ATR |
| 2 Liquidity | Pools from CHoCH swings / PDH-PDL / PWH-PWL | on / on / on |
| | Sweep buffer / Pool max age | 0 × ATR / 500 แท่ง |
| 3 MSS | CHoCH can be MSS | on |
| | Displacement body ≥ / close in outer | 1.0 × ATR / 25 % |
| | Displacement window | 0 แท่ง |
| 4 Zones | OB lookback / OB filter | 5 แท่ง / `A+ only` |
| | A+: ต้อง sweep liquidity / lookback | on / 10 แท่ง |
| | A+: ต้องมี imbalance / ขนาดขั้นต่ำ | on / 0.05 × ATR |
| | D/S: max base / base body ≤ | 3 แท่ง / 50 % ของ range |
| | Mitigated at / Remove zone when | Touch / Close beyond |
| | Zone display / max age | Fresh only / 300 แท่ง |
| 5 SFP / RJB | SFP trigger level | `Sweep candle` ⚠ Q05 |
| | SFP window / volume SMA / mult / ต้องปิดตามทิศ | 5 / 20 / 1.5 / on |
| | RJB boundary / create when / box length | Wick / SFP confirmed / 10 แท่ง |
| 6 SMT | Comparison / manual / inverse / tolerance | Auto map / `OANDA:XAGUSD` / off / 2 แท่ง |
| 7 Style | Theme | เทาดำ เส้นหนา 1 ⚠ Q12 |

---

## 4. ลำดับการคำนวณต่อแท่ง

```
 1. Series: ATR, displacement, volume SMA, range       (ทุกแท่ง)
 2. ยืนยัน pivot major (len 5) / internal (len 2) → ป้าย HH/HL/LH/LL/EQH/EQL, สร้าง pool
 3. PDH/PDL/PWH/PWL (เมื่อวัน/สัปดาห์ก่อนปิดครบแล้ว)
 4. Structure Engine: Fake BOS / CHoCH / BOS / IDM / sweep ของโครงสร้าง   (§6)
 5. MSS = BOS/CHoCH + displacement → เปิด setup                           (§7)
 6. Internal break (ใช้เป็น trigger)
 7. Liquidity sweep / breakout ของ pool ที่มีอยู่ก่อนแท่ง t                (§5)
 8. Lifecycle ของ zone เดิม (touch / mitigate / invalidate / expire)      (§8)
 9. สร้าง OB (ให้เกรด A+) และ D/S ใหม่                                    (§8)
10. SFP: เดิน pending ก่อน → ยืนยัน → RJB แล้วค่อยลงทะเบียน sweep ใหม่      (§9)
11. ผูก zone เข้ากับ MSS (แท่ง MSS และ MSS+1)                            (§11)
12. SMT                                                                   (§10)
13. Entry (เช็ค conflict, 1 entry ต่อ setup)                               (§11)
14. Alerts → Dashboard → Zone statistics
```

---

## 5. Liquidity

- `LQ-01` Pool จาก CHoCH swing (major): swing high → **BSL** และ swing low → **SSL** โดยเริ่มใช้งานได้ตั้งแต่แท่งยืนยัน
- `LQ-02` Swing ใหม่ที่อยู่ห่างจาก pool ฝั่งเดียวกันไม่เกิน `tol` → รวมกันเป็น EQH / EQL และนับ touch
- `LQ-03` PDH/PDL/PWH/PWL สร้างจากแท่งของกราฟเองหลังวัน/สัปดาห์ก่อนปิดครบแล้ว (ช่วงแรกที่ไม่ครบให้ข้าม) ตัวใหม่แทนตัวเก่า ปิดอัตโนมัติเมื่อ TF ≥ 1D (วัน) และ ≥ 1W (สัปดาห์)
- `LQ-04` **Sweep BSL**: `high > pool + buf_swp` AND `close < pool` → bearish sweep
- `LQ-05` **Breakout BSL**: `close > pool` → taken (ไม่ใช่ sweep)
- `LQ-06` SSL ใช้กฎเดียวกันกลับด้าน
- `LQ-07` ถ้าแท่งเดียวโดนหลาย pool ฝั่งเดียวกัน → ใช้ระดับที่ไกลสุด (BSL สูงสุด / SSL ต่ำสุด)
- `LQ-08` ถ้าแท่งเดียว sweep ทั้ง BSL และ SSL → `AMBIGUOUS` ไม่นับทั้งคู่
- `LQ-09` Pool ที่ถูก sweep หรือ breakout ให้ลบออก และ pool อายุเกิน 500 แท่ง → หมดอายุ
- `LQ-10` ภาพ: เส้นทึบที่ระดับที่ถูกกวาด และไฮไลต์ไส้ของแท่งที่กวาด

---

## 6. Market Structure Engine (อิงหลักการ LuxAlgo Market Structure with Inducements)

### 6.1 Swing
- `SW-01` Major (CHoCH) swing = `ta.pivothigh/low(len=5, len=5)` และ Internal (IDM) swing = `ta.pivothigh/low(2, 2)`
- `SW-02` จัดประเภทเทียบกับ swing ก่อนหน้าในชั้นเดียวกัน: ห่างไม่เกิน `tol` → EQH/EQL, ไม่อย่างนั้น HH/LH หรือ HL/LL (ป้ายใช้ major swing)

### 6.2 ตัวแปรสถานะ
| ตัวแปร | ความหมาย |
|---|---|
| `trend` | 0 = Neutral, 1 = ขาขึ้น, −1 = ขาลง |
| `pLo` / `pHi` | Protected low (ขาขึ้น) / Protected high (ขาลง) |
| `tMax` / `tMin` | Trailing high / low ของช่วงโครงสร้างปัจจุบัน (เป้าของ BOS) |
| `idm`, `idmTaken` | ระดับ IDM ปัจจุบัน และกินแล้วหรือยัง |
| `pull` | จุดสุดของการย่อตัวหลังจากกิน IDM (เป็น protected ตัวใหม่เมื่อเกิด BOS) |
| `phaseBar`, `legStart` | แท่งที่เริ่มช่วงโครงสร้าง และแท่งเริ่มขา (ใช้ผูก zone) |

### 6.3 Trend = Neutral (ช่วงเริ่มต้น, คำตอบ B6)
- `ST-01` แท่ง **ปิด** เลย major swing high ล่าสุดที่ยังไม่ถูกใช้ + `buf_brk` → **BOS ↑ (จริง)** → `startPhase(+1)`
- `ST-02` ไส้เลยแต่ไม่ได้ปิดเลย → **Fake BOS** (เส้นจุด) trend ยังเป็น 0 และระดับยังไม่ถูกใช้
- `ST-03` ฝั่งลงใช้กฎเดียวกันกลับด้าน

### 6.4 Trend = ขาขึ้น (ขาลงใช้กฎเดียวกันกลับด้าน)
| รหัส | เงื่อนไข | ผล |
|---|---|---|
| `ST-04` | **IDM candidate**: internal swing low ยืนยันแล้ว AND `pivotBar > phaseBar` AND `> pLo` AND `< tMax` AND ยังไม่กิน IDM | `idm` = ระดับนั้น (ตัวใหม่แทนตัวเก่า) |
| `ST-05` | **กิน IDM**: `low < idm − buf_swp` (ถ้าเปิด reclaim ต้อง `close > idm` ด้วย) | `idmTaken = true`, `pull = low` |
| `ST-06` | หลังจากกิน IDM: ถ้า `low < pull` | `pull = low` (ตามจุดต่ำสุดของการย่อ) |
| `ST-07` | **CHoCH ↓**: `close < pLo − buf_brk` AND (`idmTaken` OR กิน IDM ในแท่งนี้ OR ยังไม่มี IDM เลย) | trend → −1, `startPhase(−1)` |
| `ST-08` | **BOS ↑**: ไม่ใช่ CHoCH AND `idmTaken` AND `close > tMax + buf_brk` | `contPhase()` |
| `ST-09` | ทำไฮใหม่แต่ยังไม่กิน IDM | ไม่นับเป็น BOS แค่ `tMax = high` |
| `ST-10` | ไส้ทะลุ `pLo` แต่ไม่ได้ปิด หรือไส้ทะลุ `tMax` หลังกิน IDM แต่ไม่ได้ปิด | Structure sweep "x" (ครั้งเดียวต่อระดับ) |
| `ST-11` | `high > tMax` | `tMax = high` |

### 6.5 การเปลี่ยนช่วงโครงสร้าง
- `ST-12` `startPhase(+1)` (หลัง CHoCH↑ หรือ BOS แรก): `pLo` = low ต่ำสุดตั้งแต่แท่งของระดับที่ถูกทะลุจนถึงแท่ง t (ย้อนได้ไม่เกิน 300 แท่ง), `tMax = high`, ล้าง IDM, `legStart = แท่งของ pLo` ⚠ Q04
- `ST-13` `contPhase()` (หลัง BOS ต่อเนื่อง): `pLo = pull`, `legStart = แท่งของ pull`, `tMax = high` และล้าง IDM
- `ST-14` CHoCH ฝั่งตรงข้ามจะ reset setup ของฝั่งเดิมด้วย reason `PROTECTED_BROKEN`

### 6.6 ภาพ
BOS = เส้นทึบ, CHoCH = เส้นประ, Fake BOS / sweep = เส้นจุด + "x", IDM = เส้นจุดสีเทาจาก swing ไปถึงแท่งที่กิน พร้อมป้าย "IDM" (IDM ที่ยังไม่ถูกกินซ่อนไว้เป็นค่าเริ่มต้น), ถ้าเป็น MSS ให้ติดป้าย "MSS" บนเส้น BOS/CHoCH

---

## 7. Displacement + MSS

- `DP-01` Bull displacement: `range > 0` AND `bull` AND `body ≥ ATR × 1.0` AND `close ≥ low + range × 0.75`
- `DP-02` Bear displacement: กฎเดียวกันกลับด้าน (`close ≤ low + range × 0.25`)
- `MS-01` **MSS ↑** = แท่งที่เกิด **BOS ↑** (ต้องกิน IDM แล้วตาม ST-08) หรือ **CHoCH ↑** (ถ้าเปิด "CHoCH can be MSS") AND แท่งนั้นเป็น Bull displacement (window = 0)
- `MS-02` ถ้า window = N > 0: displacement เกิดได้ภายใน N แท่งก่อน break แต่ต้องอยู่หลัง `legStart`
- `MS-03` BOS/CHoCH ที่ไม่มี displacement → ยังวาดเป็นโครงสร้างตามปกติ แต่ไม่เปิด setup
- `MS-04` เมื่อเกิด MSS ให้เปิด setup ฝั่งนั้น (`stage = MSS`) และบันทึก `mssId`, `mssBar`, `legStart`, `mssType` (BOS/CHoCH)
- `MS-05` IDM sweep เพียงอย่างเดียวไม่ใช่ MSS และไม่ใช่ entry

---

## 8. Zones: OB A+ และ Demand / Supply

### 8.1 Order Block (Hybrid)
- `OB-01` เมื่อเกิดแท่ง displacement ให้ค้นย้อนหลัง 1..5 แท่งหาแท่งสีตรงข้ามล่าสุด (Bull OB ใช้แท่ง bear)
- `OB-02` Bull OB = `[low, open]` ของแท่งต้นทาง (ไส้ล่าง + ตัวแท่ง ไม่รวมไส้บน) และ Bear OB = `[open, high]`
- `OB-03` แท่ง doji ไม่ถือเป็นสีตรงข้าม ⚠ Q07 และแท่งต้นทางเดียวกันห้ามสร้าง OB ซ้ำ
- `OB-04` **เกรด A+ ต้องผ่านครบ:**
  1. เกิดจากแท่ง displacement
  2. ขานั้น sweep liquidity: low ต่ำสุดของขา (ตั้งแต่แท่งต้นทางถึงแท่ง displacement) ต่ำกว่า low ต่ำสุดของ 10 แท่งก่อนแท่งต้นทาง (ปิดเงื่อนไขนี้ได้)
  3. ขานั้นทิ้ง imbalance (3-candle gap ≥ 0.05 × ATR) ใช้แค่คัด OB ไม่วาด FVG โดย gap ที่จบที่แท่ง D+1 ก็นับ (ปิดเงื่อนไขนี้ได้)
  4. ขานั้นทำ MSS (§7)
  5. ยังไม่ถูกแตะ (fresh)
- `OB-05` OB filter = `All` → ใช้ OB ทุกตัวโดยไม่บังคับ A+

### 8.2 Demand / Supply
- `DS-01` Base = แท่งติดกัน 1–3 แท่งก่อนแท่ง displacement ที่ `body ≤ 50% ของ range`
- `DS-02` Demand = `[low ต่ำสุดของ base, bodyTop สูงสุดของ base]`
- `DS-03` Supply = `[bodyBot ต่ำสุดของ base, high สูงสุดของ base]`
- `DS-04` ไม่มี base → ไม่สร้าง D/S

### 8.3 Zone lifecycle (ตรวจ zone เดิมก่อนสร้าง zone ใหม่)
- `ZN-01` แตะ (Bull): `low ≤ top` → first touch (ครั้งเดียว)
- `ZN-02` ความลึก = `(top − low) / (top − bottom)` วัดจากขอบ proximal
- `ZN-03` Mitigated ตามเกณฑ์ (Touch / 50% / Full) ถ้าแสดงแบบ `Fresh only` ให้ลบกล่อง แต่ zone ยังใช้ entry ได้จนกว่าจะ invalidated ⚠ Q08
- `ZN-04` Invalidated (Bull): `close < bottom − buf_inv` (หรือ `low` ถ้าเลือก Wick through)
- `ZN-05` Expired: อายุเกิน 300 แท่ง
- `ZN-06` แบบ `All active`: กล่องหดตามราคาที่เข้าไปในโซน
- `ZN-07` ภาพ: กล่องยืดไปทางขวา ป้ายในกล่อง เช่น "Demand OB A+" มีเส้นประ 50% และแผงสถิติ Fresh / Mitigated / Active
- `ZN-08` วาดเฉพาะ zone ที่ผูกกับ MSS ส่วน zone อื่นซ่อนไว้ (เปิดได้ด้วย "Also draw zones not linked to an MSS")

---

## 9. SFP + Rejection Block (คำตอบ A3)

> นิยามของผู้ใช้: แท่งขึ้นไปกวาดไฮ แล้วกลับมาทะลุโลของไฮนั้น **ด้วยเนื้อเทียน** ถ้าทะลุแค่ไส้จะไม่นับ ยกเว้นแท่งถัดไปมีวอลุ่มรุนแรง

Bearish SFP (Bullish ใช้กฎเดียวกันกลับด้าน):
| รหัส | เงื่อนไข |
|---|---|
| `SF-01` | แท่ง sweep `S` = แท่งที่เกิด BSL sweep (LQ-04) ซึ่ง pool ต้องมีอยู่ก่อนแท่ง S |
| `SF-02` | ระดับยืนยัน `L_ref` = low ของแท่ง S (`Sweep candle`) หรือ low ของแท่ง swing ที่ถูกกวาด (`Swept swing candle`) ⚠ Q05 |
| `SF-03` | **ยืนยันด้วยเนื้อเทียน**: ภายใน 5 แท่งหลัง S มีแท่งที่ `close < L_ref` → SFP ↓ |
| `SF-04` | **ยืนยันด้วยไส้ + volume**: มีแท่ง C ที่ `low < L_ref` แต่ `close ≥ L_ref` แล้วแท่ง C+1 มี `volSpike` และปิดตามทิศ (`bear`) → SFP ↓ ยืนยันที่ C+1 |
| `SF-05` | ถ้าก่อนยืนยันมีแท่งปิดเหนือ `high[S]` → ยกเลิก |
| `SF-06` | หมดหน้าต่าง 5 แท่ง → หมดอายุ และถ้ามี sweep ใหม่เกิดขึ้น ให้แทน pending เดิม |
| `SF-07` | ถ้าใช้โหมด `Swept swing candle` และแท่ง S ปิดใต้ `L_ref` อยู่แล้ว → ยืนยันที่แท่ง S เลย |
| `SF-08` | สัญลักษณ์ที่ไม่มีข้อมูล volume ใช้ได้แค่ SF-03 (Forex/XAU ใช้ tick volume ของโบรก) |
| `SF-09` | ภาพ (v1.5): **ไม่มีป้าย SFP** มีแค่เส้นจุดที่ระดับยืนยันและกล่อง RJB |

Rejection Block (RJB, คำตอบ B7 = a):
- `RB-01` สร้างเมื่อ SFP ยืนยันแล้ว (option: สร้างตั้งแต่ตอน sweep) จากแท่ง S ⚠ Q06
- `RB-02` Wick (ค่าเริ่มต้น): Bear = `[bodyTop, high]` / Bull = `[low, bodyBot]` ส่วน Body = `[bodyBot, bodyTop]` และ Full = `[low, high]`
- `RB-03` ไส้ที่สูงเป็น 0 → ไม่สร้าง
- `RB-04` Invalidated: Bear `close > high[S] + buf_rb` / Bull `close < low[S] − buf_rb`
- `RB-05` กล่องยาวคงที่ 10 แท่ง ป้าย "RJB" มุมขวาบน และไม่ใช่ entry zone
- `RB-06` เงื่อนไข trapped wick / signal wick ของต้นแบบยังไม่ได้ใส่ ⚠ Q11

---

## 10. SMT Divergence (TF เดียวกับกราฟ)

- `SM-01` คู่เทียบอัตโนมัติ (`Auto map`):

| กราฟ | คู่เทียบ | Inverse |
|---|---|---|
| XAUUSD | XAGUSD | ไม่ |
| EURUSD ↔ GBPUSD | กันและกัน | ไม่ |
| AUDUSD ↔ NZDUSD | กันและกัน | ไม่ |
| USDJPY | USDCHF | ไม่ |
| USDCHF | EURUSD | ใช่ |
| อื่น ๆ | `Manual` (ค่าเริ่มต้น `OANDA:XAGUSD`) + checkbox inverse | – |

  ใช้ prefix ของโบรกเกอร์เดียวกับกราฟ ⚠ Q09
- `SM-02` คำนวณ major pivot ของคู่เทียบด้วย L/R เดียวกัน ผ่าน `request.security(comp, timeframe.period, …)`
- `SM-03` **Bull SMT**: กราฟยืนยัน major LL แต่คู่เทียบมี pivot low ที่เวลาต่างกันไม่เกิน ±2 แท่ง และ **ไม่** ทำ LL (กรณี inverse: pivot high ของคู่เทียบไม่ทำ HH) โดยรอได้ไม่เกิน 2 แท่งหลังยืนยัน
- `SM-04` Bear SMT ใช้กฎเดียวกันกลับด้านที่ HH
- `SM-05` ไม่มีข้อมูล / ไม่พบ pivot ที่จับคู่ได้ / สัญลักษณ์เดียวกับกราฟ → `N/A` (ไม่นับว่า fail)
- `SM-06` ภาพ: เส้นเชื่อม swing สองจุดของกราฟที่เกิด SMT พร้อมป้าย "SMT"
- `SM-07` SMT เป็นข้อมูลประกอบเท่านั้น ไม่ใช่เงื่อนไขบังคับของโปรไฟล์ใด ⚠ Q13

---

## 11. Setup → Entry

### 11.1 State ต่อฝั่ง
`NONE → MSS → ZONE (ผูกโซนแล้ว รอราคากลับมา) → ENTRY → NONE`

| รหัส | เงื่อนไข |
|---|---|
| `SE-01` | NONE → MSS: MS-01 และผ่าน bias ตามโปรไฟล์ (SE-09) |
| `SE-02` | ผูก zone ที่แท่ง MSS และ MSS+1: zone ทิศเดียวกันที่แท่งต้นทางอยู่ในช่วง `[legStart, mssBar]` ตามโหมด Entry zone |
| `SE-03` | MSS → ZONE: ผูกได้ ≥ 1 zone ถ้าถึงแท่ง MSS+1 แล้วยังไม่มี → reset `NO_LINKED_ZONE` |
| `SE-04` | ZONE → ENTRY: zone ที่ผูกไว้ยังไม่ invalidated AND ถูกแตะ AND trigger ผ่าน (11.2) |
| `SE-05` | ENTRY → NONE: reset `ENTRY_DONE` (1 entry ต่อ setup) |
| `SE-06` | Reset `PROTECTED_BROKEN` เมื่อเกิด CHoCH ฝั่งตรงข้าม |
| `SE-07` | Reset `ZONE_INVALIDATED` / `ZONE_EXPIRED` เมื่อ zone ที่ผูกไว้หมดทุกตัว |
| `SE-08` | ถ้าเกิด MSS ใหม่ฝั่งเดิมระหว่างรอ ให้ใช้ setup ใหม่แทน ⚠ Q14 |
| `SE-09` | Bias = `trend` ของ Structure Engine: Strict ต้องตรงทิศ ส่วน Balanced ยอมให้ neutral หรือสวนได้แต่ติดป้าย `CT` |

### 11.2 Trigger (Bull ส่วน Bear ใช้กฎเดียวกันกลับด้าน)
- `TR-01` **Rejection Candle** (ค่าเริ่มต้น): `low ≤ zoneTop` AND `close > zoneTop` AND `close ≥ low + range × 0.5`
- `TR-02` **Internal MSS**: internal BOS/CHoCH ↑ (swing len 2) ภายใน 10 แท่งหลังแตะ zone ครั้งแรก
- `TR-03` **SFP**: SFP ↑ ยืนยันแล้ว และไส้ของแท่ง sweep อยู่ในโซน
- `TR-04` **Any**: trigger ใดก็ได้

### 11.3 Entry zone mode
`OB A+ or D/S` (ค่าเริ่มต้น) / `OB A+ only` / `D/S only` / `OB A+ overlapping D/S` (ต้องมีทั้งสองแบบและซ้อนกัน)

### 11.4 ผลลัพธ์
- `EN-01` วาด **กล่องจุดเข้า** BUY/SELL + ชนิดโซน ยาว 20 แท่ง ไม่มีป้ายใหญ่
- `EN-02` ถ้า BUY และ SELL เกิดในแท่งเดียวกัน → `CONFLICT` ไม่ให้สัญญาณ และทั้งสอง setup รอต่อ
- `EN-03` Context-only → ไม่วาดจุดเข้า (`CONTEXT_ONLY`)
- `EN-04` Checklist tooltip (ปิดไว้เป็นค่าเริ่มต้น): MSS type + displacement, zone, trigger, bias, SMT (info) และไม่ใช่ความน่าจะเป็น

---

## 12. Alerts / Dashboard

- `AL-01` `alertcondition()` ที่แท่งปิด เกิดครั้งเดียวต่อเหตุการณ์: BOS↑↓, CHoCH↑↓, Fake BOS, MSS↑↓, Liquidity sweep, IDM candidate, IDM taken, SFP↑↓, New OB, New D/S, Zone mitigated, SMT↑↓, BUY, SELL (แนะนำให้ตั้ง alert แบบ "Once per bar close")
- `UI-01` Dashboard: profile + trigger, trend + protected L/H, สถานะ IDM, setup ทั้งสองฝั่ง + reason ล่าสุด, SMT (คู่ที่ใช้ / N/A), PD/PW พร้อมใช้หรือไม่
- `UI-02` Debug mode: แสดง reason ของการ reset บนกราฟ

---

## 13. Reason codes
`ENTRY_DONE` · `PROTECTED_BROKEN` · `NO_LINKED_ZONE` · `ZONE_INVALIDATED` · `ZONE_EXPIRED` · `CONFLICT` · `CONTEXT_ONLY` · `BIAS_NOT_ALIGNED` · `AMBIGUOUS` · `DATA_NA`

---

## 14. Tests ที่ต้องผ่าน (ทดสอบแบบทีละแท่งใน Bar Replay)
1. Pivot ไม่ถูกใช้ก่อนครบ R แท่ง
2. Neutral: ไส้ทะลุ = Fake BOS (trend ไม่เปลี่ยน) ส่วนปิดทะลุ = BOS
3. ทำไฮใหม่ก่อนกิน IDM → ไม่มี BOS
4. IDM นอกกรอบ (ใต้ pLo หรือเหนือ tMax) → ไม่ถูกนับ
5. CHoCH ก่อนกิน IDM ไม่เกิด (ยกเว้นกินในแท่งเดียวกัน หรือยังไม่มี IDM)
6. BOS → pLo ย้ายไปที่ `pull`
7. BOS/CHoCH ที่ไม่มี displacement → ไม่มี MSS
8. OB Hybrid ใช้ขอบตามสูตร และไม่ใช้แท่ง doji
9. OB A+ ตกเมื่อปิดเงื่อนไขใดเงื่อนไขหนึ่งที่บังคับอยู่ไม่ผ่าน
10. D/S ใช้ขอบตาม DS-02/03
11. Zone ไม่ถูก retest ในแท่งที่สร้าง
12. Sweep กับ breakout แยกกันถูก
13. SFP ไม่เกิดเมื่อทะลุแค่ไส้และไม่มี volume, เกิดเมื่อแท่งถัดไปมี volSpike
14. RJB สร้างหลัง SFP ยืนยัน
15. SMT = N/A เมื่อไม่มีข้อมูลคู่เทียบ
16. 1 entry ต่อ setup, BUY/SELL แท่งเดียวกัน = ไม่มีสัญญาณ
17. Alert ไม่ซ้ำตอน realtime
18. Reload และ Bar Replay ให้เหตุการณ์ย้อนหลังเหมือนเดิม
19. ใช้ได้กับ tick size และ suffix ของสัญลักษณ์ที่ต่างกัน
20. ไม่เกิน object limit และไม่มีสัญญาณเมื่อ history ไม่พอ

Repaint audit: ไม่มี `lookahead_on`, ไม่มี offset ติดลบ, แยกตำแหน่ง pivot กับเวลายืนยัน, ไม่มี intrabar preview
ส่งมอบ: source Pine v6 + ผล compile (เฉพาะที่ทดสอบจริง) + ตารางผล test + ข้อจำกัด

---

## 15. Workflow สำหรับ Claude ตอนเขียนโค้ดต่อ
1. อ่านไฟล์นี้ทั้งหมดก่อน และยึดรหัสเงื่อนไขในไฟล์นี้
2. ทุกครั้งที่แก้โค้ด ให้แปะโค้ดเต็มในแชทและ push ขึ้น GitHub
3. บอกว่าแก้รหัสไหนบ้าง และอัปเดต §17 changelog
4. ตอบเป็นภาษาไทย

---

## 16. คำถามที่ยังเปิด
ตอบในรูปแบบ `[x] ตามที่ใช้อยู่` หรือเขียนใน `คำตอบ:`

| Q | คำถาม | ค่าที่ใช้อยู่ | ตอบ |
|---|---|---|---|
| Q01 | MSS ใช้แบบ v1.5 (BOS หลังกิน IDM หรือ CHoCH + displacement) หรือแบบ v2.0 (IDM sweep → ทะลุ internal swing high + displacement) | v1.5 | [ ] |
| Q02 | Swing length ใช้ 5/2 (แบบ LuxAlgo, v1.5) หรือ pivot 3/3 และ 1/1 (สเปกเดิม) | 5 / 2 | [ ] |
| Q03 | กิน IDM ต้องปิดกลับเหนือ IDM ด้วยหรือไม่ (สเปกเดิมบอกว่าต้องปิดกลับ แต่ v1.5 ใช้แค่ไส้ทะลุ) | ไม่ต้องปิดกลับ | [ ] |
| Q04 | Protected low หลัง CHoCH = low ต่ำสุดระหว่างระดับที่ถูกทะลุกับแท่ง break ใช่หรือไม่ | ใช่ | [ ] |
| Q05 | SFP "โลของไฮนั้น" = low ของแท่งที่กวาด หรือ low ของแท่ง swing ที่ถูกกวาด | แท่งที่กวาด | [ ] |
| Q06 | RJB สร้างตอน SFP ยืนยัน หรือตั้งแต่ตอน sweep | SFP ยืนยัน | [ ] |
| Q07 | Doji ตอนค้นหา OB: ข้ามแล้วค้นต่อ หรือยกเลิก OB | ข้ามแล้วค้นต่อ | [ ] |
| Q08 | Zone ที่ mitigated แล้ว (ลบกล่องไปแล้ว) ยังใช้เป็น entry ได้หรือไม่ | ได้ จนกว่าจะ invalidated | [ ] |
| Q09 | โบรกเกอร์ / prefix ที่ใช้ (เช่น OANDA, FXCM, ของโบรกเอง) | prefix เดียวกับกราฟ | [ ] |
| Q10 | โปรไฟล์เริ่มต้น Balanced หรือ Strict | Balanced | [ ] |
| Q11 | RJB ต้องมีเงื่อนไข trapped wick / signal wick หรือไม่ (ถ้าต้องการให้บอกนิยาม) | ยังไม่มี | [ ] |
| Q12 | สีเทาดำ หรือเขียว/แดงแบบต้นแบบ | เทาดำ | [ ] |
| Q13 | SMT ควรเป็นเงื่อนไขบังคับใน Strict หรือไม่ | ไม่บังคับ | [ ] |
| Q14 | เกิด MSS ใหม่ฝั่งเดิมขณะรอโซน: แทน setup เดิม หรือเก็บไว้ทั้งคู่ | แทน | [ ] |
| Q15 | อยากเพิ่ม FVG กลับมาเป็นโซนแสดงผล (ไม่ใช่ entry) หรือไม่ | ไม่ | [ ] |
| Q16 | ต้องมีเฟส 2 (โซนจาก TF ใหญ่แบบ JP7FX) หรือยึด TF เดียวตลอด | TF เดียว | [ ] |

---

## 17. Changelog
### v3.0 — 2026-11-12
- รวมไฟล์สเปกทั้งหมดเหลือไฟล์เดียว และลบไฟล์เก่าออกจาก `main`
- รวม: TF เดียว (v2.0) + โครงสร้าง LuxAlgo-IDM, OB A+, D/S, กล่องจุดเข้า, เส้น SMT, ไม่มีป้าย SFP (v1.3–v1.5) + คำตอบ A–E (ไฟล์ 10)
- ตัด HTF / MTF และ FVG ในฐานะโซน entry
- เพิ่มรายการคำถามที่ยังเปิด Q01–Q16
- ยังไม่มีโค้ดที่ตรงกับ v3.0 รอผู้ใช้ตรวจไฟล์นี้ก่อน

### ประวัติก่อนหน้า
- v2.0: สเปกและโค้ด TF เดียว (`SMC_SingleTF.pine`) อยู่ที่ branch `single-timeframe` และ `main` เดิม
- v1.3–v1.5: `SMC_Suite.pine` อยู่ที่ branch `claude/apply-open-issues-v1-2`
- v1.1–v1.2: สเปกเดิม + คำตอบไฟล์ 10 อยู่ที่ branch `claude/open-issues-review`
