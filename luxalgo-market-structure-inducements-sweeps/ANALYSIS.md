# วิเคราะห์เงื่อนไข: Market Structure with Inducements & Sweeps [LuxAlgo]

| ไฟล์ | ใช้ทำอะไร |
|---|---|
| [`original.pine`](original.pine) | **ไฟล์หลัก** (Pine v5) ห้ามแก้ ใช้เทียบ |
| [`editable.pine`](editable.pine) | เวอร์ชันสำหรับแก้ไข: แยกเงื่อนไขเป็นตัวแปร `cond_*` มีคอมเมนต์ภาษาไทย และเพิ่ม Alert |
| `ANALYSIS.md` | เอกสารนี้ |
| [`MAIN-ANALYSIS.md`](MAIN-ANALYSIS.md) | วิเคราะห์ไฟล์หลักพร้อมเลขบรรทัด และสิ่งที่เปลี่ยนจากเวอร์ชัน v6 ของ LuxAlgo Library |

**ที่มา:** [LuxAlgo Library](https://www.luxalgo.com/library/indicator/market-structure-with-inducements-sweeps/) และ [TradingView](https://www.tradingview.com/script/kMDlPZuz-Market-Structure-with-Inducements-Sweeps-LuxAlgo/), เผยแพร่ 26 ก.พ. 2024

**เวอร์ชัน:** ไฟล์หลัก `original.pine` ใช้ **Pine Script v5** ส่วน `editable.pine` ใช้ **Pine Script v6** ทั้งสองไฟล์ใช้ syntax `input(...)` แบบเดียวกัน และ logic เหมือนกัน (ดูรายละเอียดใน `MAIN-ANALYSIS.md`)

**License:** [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/) © LuxAlgo
ใช้/แก้/แจกต่อได้ แต่ **ต้องให้เครดิต LuxAlgo**, **ห้ามใช้เชิงพาณิชย์**, และงานที่ดัดแปลงต้องใช้ license เดียวกัน (ไฟล์ทั้งสองจึงเก็บ header license ไว้)

---

## 1. ภาพรวม

Indicator นี้วาด Market Structure แบบ SMC ลงบนกราฟ 4 อย่าง:

| ป้าย | ความหมาย | ใช้ราคาอะไรตัดสิน |
|---|---|---|
| **CHoCH** (เส้นประ) | Change of Character: โครงสร้างกลับทิศ | **close** ทะลุ swing ใหญ่ |
| **IDM** (เส้นจุด) | Inducement: pullback แทงสภาพคล่องของ swing เล็ก | **ไส้เทียน** (low/high) |
| **BOS** (เส้นทึบ) | Break of Structure: เทรนด์ไปต่อ | **close** ทะลุ trailing extreme และต้องมี IDM ก่อน |
| **x** (เส้นจุด) | Sweep: ไส้กวาด extreme แต่ปิดกลับเข้ามา | **ไส้** ทะลุ + **close** กลับ |

ลำดับที่ indicator คาดหวังในขาขึ้น:

```
CHoCH ↑  →  IDM (ย่อลงแทง swing low เล็ก)  →  BOS ↑  →  IDM  →  BOS ↑  → ...
                                     (ระหว่างนี้อาจมี x = sweep high)
```

ขาลงเป็นกลับด้านทั้งหมด

---

## 2. Settings (Inputs)

| Input | ค่าเริ่มต้น | ตัวแปร | ผล |
|---|---|---|---|
| CHoCH Detection Period | 50 | `len` | ความยาว swing ใหญ่ ยิ่งมาก CHoCH ยิ่งน้อยและเป็นระยะยาว |
| IDM Detection Period | 3 | `shortLen` | ความยาว swing เล็ก ยิ่งมาก IDM ยิ่งต้องเป็น pullback ที่ใหญ่ขึ้น → BOS เกิดยากขึ้น |
| Bullish / Bearish Elements | #089981 / #ff5252 | `bullCss` / `bearCss` | สี |
| Show CHoCH / BOS / Inducements / Sweeps / Swings | true | `show*` | เปิด/ปิดการแสดงผล (**ไม่กระทบ logic** ปิดไปแล้ว state ยังคำนวณเหมือนเดิม) |
| สี IDM / Sweeps | gray | `idmCss` / `sweepsCss` | สี |

---

## 3. ตัวแปร State หลัก

| ตัวแปร | ความหมาย |
|---|---|
| `os` | ทิศโครงสร้าง: `1` = Bullish, `0` = Bearish (เริ่มต้น 0) |
| `topy`, `topx` / `btmy`, `btmx` | ราคา/แท่งของ swing high / low **ใหญ่** ล่าสุด (ระดับ CHoCH) |
| `top_crossed` / `btm_crossed` | swing ใหญ่นั้นถูก close ทะลุแล้ว (กันนับซ้ำ) |
| `stopy`, `stopx` / `sbtmy`, `sbtmx` | swing high / low **เล็ก** ล่าสุด (ระดับ IDM) |
| `stop_crossed` / `sbtm_crossed` | IDM ฝั่งขาลง / ขาขึ้น เกิดแล้วใน leg นี้ ("ใบอนุญาต" ให้ BOS) |
| `max`, `max_x1` / `min`, `min_x1` | trailing high สูงสุด / low ต่ำสุด นับจาก CHoCH ล่าสุด (ระดับ BOS และ Sweep) |

---

## 4. เงื่อนไขทีละขั้น

ชื่อในวงเล็บคือชื่อตัวแปรใน `editable.pine`

### 4.1 การหา Swing: ฟังก์ชัน `swings(len)`

```pine
os := high[len] > ta.highest(len) ? 0 : low[len] < ta.lowest(len) ? 1 : os[1]
top = os == 0 and os[1] != 0 ? high[len] : na
btm = os == 1 and os[1] != 1 ? low[len]  : na
```

- ดูแท่งย้อนหลัง `len` แท่ง ถ้า high ของมันสูงกว่าทุกแท่งใน `len` แท่งหลังจากนั้น แปลว่าเป็นยอด (`os = 0`)
- swing high จะถูกบันทึกเฉพาะ **ตอนสถานะเปลี่ยน** จากก้นมาเป็นยอด (สลับกันเสมอ ได้ high-low-high-low)
- ⚠️ **Lag:** swing ยืนยันช้า `len` แท่ง (ค่าเริ่มต้น 50 แท่งสำหรับ CHoCH และ 3 แท่งสำหรับ IDM) วงกลม Swing ที่ plot ใช้ `offset = -len` จึงดูเหมือนอยู่ที่จุดจริง แต่ตอน real-time มันปรากฏทีหลัง
- ฟังก์ชันถูกเรียก 2 ครั้ง: `swings(len)` ได้ swing ใหญ่ และ `swings(shortLen)` ได้ swing เล็ก

### 4.2 CHoCH

| เงื่อนไข | โค้ด |
|---|---|
| close ทะลุ swing high ใหญ่ (`cond_chochBullBreak`) | `close > topy and not top_crossed` → `os := 1` |
| close ทะลุ swing low ใหญ่ (`cond_chochBearBreak`) | `close < btmy and not btm_crossed` → `os := 0` |
| **แสดง CHoCH** (`cond_choch`) | `os != os[1]` |

- CHoCH จะแสดงเฉพาะเมื่อ `os` **เปลี่ยนทิศจริง** ถ้าทิศเดิมเป็นขาขึ้นแล้ว close ทะลุ swing high ใหญ่อีก จะไม่มีป้ายอะไร แต่ `top_crossed` ถูกตั้งเป็น true
- เมื่อเกิด CHoCH ระบบจะ **รีเซ็ต leg ใหม่**: `max = high`, `min = low` ของแท่งนั้น และล้าง `stop_crossed` / `sbtm_crossed`
- CHoCH **ไม่ต้องมี IDM ก่อน**

### 4.3 IDM (Inducement)

| ฝั่ง | โค้ด |
|---|---|
| Bullish (`cond_idmBull`) | `low < sbtmy and not sbtm_crossed and os == 1 and sbtmy != btmy` |
| Bearish (`cond_idmBear`) | `high > stopy and not stop_crossed and os == 0 and stopy != topy` |

- ในขาขึ้น: ราคา **ย่อลงจนไส้แทง** swing low เล็กล่าสุด ถือว่าสภาพคล่องฝั่ง pullback ถูกกินแล้ว
- `sbtmy != btmy`: ถ้า swing เล็กกับ swing ใหญ่เป็นจุดเดียวกัน จะไม่นับเป็น IDM (กันซ้อนกับระดับ CHoCH)
- นับได้ **ครั้งเดียวต่อรอบ BOS** (`sbtm_crossed` ถูกตั้งเป็น true จนกว่าจะเกิด BOS หรือ CHoCH)

### 4.4 BOS

| ฝั่ง | โค้ด |
|---|---|
| Bullish (`cond_bosBull`) | `close > max and sbtm_crossed and os == 1` |
| Bearish (`cond_bosBear`) | `close < min and stop_crossed and os == 0` |

- `max` คือ high สูงสุดของ leg ณ **แท่งก่อนหน้า** (เพราะ `max` ถูกอัปเดตด้วย high ของแท่งปัจจุบันหลังเช็กเงื่อนไข)
- **ต้องมี IDM ก่อน** (`sbtm_crossed`) นี่คือจุดต่างจาก BOS แบบทั่วไป
- เกิด BOS แล้ว `sbtm_crossed := false` → BOS ครั้งถัดไปต้องมี IDM ใหม่ก่อน
- `max` **ไม่ถูกรีเซ็ต** หลัง BOS แต่ไล่ตามต่อ ดังนั้น BOS ถัดไปต้องปิดเหนือ high สูงสุดใหม่

### 4.5 Sweep (x)

| ฝั่ง | โค้ด |
|---|---|
| ขาขึ้น กวาด high (`cond_sweepBull`) | `high > max and close < max and os == 1 and n - max_x1 > 1` |
| ขาลง กวาด low (`cond_sweepBear`) | `low < min and close > min and os == 0 and n - min_x1 > 1` |

- ไส้ทะลุ trailing extreme แต่ **ปิดกลับเข้ามา**
- `n - max_x1 > 1`: extreme ต้องเกิดก่อนแท่งปัจจุบันอย่างน้อย 2 แท่ง
- **ไม่ต้องมี IDM** และไม่เปลี่ยน state ใด ๆ (แสดงผลอย่างเดียว)
- หลัง sweep `max` จะเลื่อนขึ้นไปเป็น high ของไส้นั้น (ข้อ 4.6) ทำให้ BOS ถัดไปต้องปิดเหนือไส้ที่กวาด

### 4.6 อัปเดต Trailing max/min (ท้ายทุกแท่ง)

```pine
max := math.max(high, max)
min := math.min(low, min)
if max > max[1] → max_x1 := n
if min < min[1] → min_x1 := n
```

⚠️ ต้องอยู่ **หลัง** การเช็ก BOS/Sweep เสมอ ถ้าย้ายขึ้นไปก่อน `close > max` จะเป็นจริงไม่ได้

### 4.7 Extensions (แท่งสุดท้าย)

ที่ `barstate.islast` จะวาดเส้น "ระดับรอทะลุ":

| ถ้า os = 1 (ขาขึ้น) | ถ้า os = 0 (ขาลง) |
|---|---|
| CHoCH ↓ ที่ `btmy` (จุดที่จะกลับเป็นขาลง) | CHoCH ↑ ที่ `topy` |
| BOS ↑ ที่ `max` | BOS ↓ ที่ `min` |
| IDM ที่ `sbtmy` (ถ้ายังไม่ถูกแทง) ยื่นไปข้างหน้า 15 แท่ง | IDM ที่ `stopy` (ถ้ายังไม่ถูกแทง) |

---

## 5. ลำดับการทำงานในแต่ละแท่ง (ห้ามสลับ)

1. คำนวณ swing ใหญ่ + เล็ก
2. บันทึก swing ใหญ่ใหม่ → `topy`/`btmy`
3. เช็ก close ทะลุ swing ใหญ่ → เปลี่ยน `os`
4. ถ้า `os` เปลี่ยน → วาด CHoCH + รีเซ็ต leg
5. IDM ขาขึ้น → BOS ขาขึ้น
6. IDM ขาลง → BOS ขาลง
7. Sweep
8. อัปเดต trailing `max`/`min`
9. Extensions (แท่งสุดท้าย) + plot วงกลม swing

ข้อสังเกต: IDM กับ BOS **เกิดในแท่งเดียวกันได้** (ข้อ 5 เช็ก IDM ก่อน BOS) และ CHoCH กับ Sweep ก็เกิดแท่งเดียวกันได้

---

## 6. ข้อสังเกต / จุดที่ควรรู้

1. **สี IDM ฝั่ง Bearish ในต้นฉบับใช้ `color.gray` ตายตัว** ไม่ได้ใช้ `idmCss` ทำให้เปลี่ยนสี IDM ใน Settings แล้วฝั่งขาลงไม่เปลี่ยนตาม → แก้แล้วใน `editable.pine`
2. **Swing lag:** CHoCH อิง swing ที่ยืนยันช้า 50 แท่ง ระดับ CHoCH ล่าสุดจึงอาจยังไม่ใช่ยอด/ก้นล่าสุดที่ตาเห็น
3. **ไม่ repaint ย้อนหลัง** (ไม่ใช้ `request.security` / lookahead) แต่บนแท่งที่ยังไม่ปิด ป้ายอาจโผล่แล้วหายได้ตาม close ที่ยังวิ่งอยู่ ใช้ "Once Per Bar Close" ตอนตั้ง Alert
4. **Show\* ไม่กระทบ logic:** ปิดการแสดง IDM แล้ว BOS ก็ยังต้องรอ IDM อยู่
5. **จำนวน object จำกัด 500 เส้น / 500 label** กราฟยาวมาก ๆ ของเก่าจะหายไป
6. `os` เริ่มที่ 0 (Bearish) ช่วงต้นกราฟก่อน CHoCH แรกจึงมีพฤติกรรมเป็นขาลง

---

## 7. ไอเดียการแก้ไข (ใน `editable.pine`)

| อยากได้ | แก้ตรงไหน |
|---|---|
| BOS แบบไม่ต้องรอ IDM | ลบ `and sbtm_crossed` ใน `cond_bosBull` และ `and stop_crossed` ใน `cond_bosBear` |
| CHoCH ต้องปิดทะลุแบบมี buffer | `close > topy + ta.atr(14) * 0.1` ใน `cond_chochBullBreak` |
| IDM ต้องปิดทะลุ ไม่ใช่แค่ไส้ | เปลี่ยน `low < sbtmy` เป็น `close < sbtmy` ใน `cond_idmBull` |
| Sweep ต้องมี IDM ก่อนด้วย | เพิ่ม `and sbtm_crossed` ใน `cond_sweepBull` |
| แยก period ของ swing high/low | เรียก `swings()` ด้วยค่า len ต่างกัน แล้วใช้คนละชุด |
| ใช้เป็นสัญญาณเทรด | ใช้ตัวแปร `cond_*` ได้เลย เช่น เข้า Long เมื่อ `cond_bosBull` และ Stop ที่ `sbtmy` |
| Alert | มี `alertcondition` ให้ 8 ตัวท้ายไฟล์ (CHoCH/BOS/IDM/Sweep ทั้งสองฝั่ง) |

> หมายเหตุ: `editable.pine` ยังไม่ได้ทดสอบ compile บน TradingView (environment นี้รัน Pine ไม่ได้) ถ้าเจอ error ให้เทียบกับ `original.pine` ได้เลย เพราะโครงสร้างตรงกันทุกบล็อก
