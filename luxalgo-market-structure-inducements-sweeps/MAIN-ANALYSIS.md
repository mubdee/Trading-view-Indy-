# วิเคราะห์เงื่อนไขไฟล์หลัก (branch `main`, commit `65e16f6`)

เอกสารนี้วิเคราะห์ `original.pine` บน branch `main` หลังจากที่แก้ใน commit
`65e16f6 Update Pine Script version and input syntax`
**ไม่มีการแก้ไขไฟล์ใด ๆ บน `main`** เอกสารนี้อยู่แยกใน branch `claude/main-analysis`

License: CC BY-NC-SA 4.0 © LuxAlgo

---

## 1. สิ่งที่เปลี่ยนใน `main` (เทียบกับเวอร์ชันจาก LuxAlgo Library)

| บรรทัด | ก่อน | หลัง (ไฟล์หลักปัจจุบัน) |
|---|---|---|
| 4 | `//@version=6` | `//@version=5` |
| 5 | ชื่อ `"Market Structure with Inducements & Sweeps"` | `"Market Structure with Inducements & Sweeps [LuxAlgo]"`, short title `"LuxAlgo - Market Structure with Inducements & Sweeps"` |
| 9–26 | `input.int(...)`, `input.color(...)`, `input.bool(...)` | `input(...)` แบบรวม (Pine อนุมานชนิดจากค่าเริ่มต้น: 50 → int, #089981 → color, true → bool) |
| ท้ายไฟล์ | ไม่มี newline | มี newline |

**บรรทัด 29–216 (logic ทั้งหมด) ไม่เปลี่ยนแม้แต่ตัวอักษรเดียว** ตรวจด้วย `git diff`
ดังนั้นเงื่อนไข CHoCH / IDM / BOS / Sweep ทำงานเหมือนเดิมทุกประการ

### ผลของการเปลี่ยนเป็น v5

| ประเด็น | ผล |
|---|---|
| `if top` / `if btm` (บรรทัด 75, 78) | ใน v5 ค่า float แปลงเป็น bool อัตโนมัติ (`na` = false, มีค่า = true) จึงใช้ได้ ส่วน v6 ไม่ยอมแปลงอัตโนมัติ เวอร์ชัน v6 เดิมจึงอาจ compile ไม่ผ่านบน TradingView (ใน `editable.pine` ใช้ `not na(top)` แทนแล้ว) |
| `input(...)` | ใช้ได้ทั้ง v5 และ v6 ผลเท่ากับ `input.int` / `input.color` / `input.bool` |
| `and` / `or` | v5 ประเมินทุกฝั่ง ส่วน v6 หยุดประเมินเมื่อรู้ผลแล้ว (lazy evaluation) แต่เงื่อนไขในสคริปต์นี้ไม่มี side effect ผลจึงเหมือนกัน |
| method syntax `ext_choch.set_xy1(...)` (บรรทัด 181–209) | รองรับใน v5 |

---

## 2. Settings ของไฟล์หลัก

| บรรทัด | Input | ค่าเริ่มต้น | ใช้ใน |
|---|---|---|---|
| 9 | CHoCH Detection Period (`len`) | 50 | swing ใหญ่ → ระดับ CHoCH |
| 10 | IDM Detection Period (`shortLen`) | 3 | swing เล็ก → ระดับ IDM |
| 13–14 | Bullish / Bearish Elements | #089981 / #ff5252 | สี CHoCH, BOS |
| 16–17 | Show CHoCH / Show BOS | true | แสดงผลเท่านั้น |
| 19–20 | Show Inducements + สี | true, gray | แสดงผลเท่านั้น |
| 22–23 | Show Sweeps + สี | true, gray | แสดงผลเท่านั้น |
| 25 | Show Swings | true | วงกลม swing |

> ปิด Show\* แล้ว logic ยังคำนวณเหมือนเดิม เช่น ปิด IDM แล้ว BOS ก็ยังต้องรอ IDM

---

## 3. เงื่อนไขทั้งหมด (อ้างอิงบรรทัดในไฟล์หลัก)

### 3.1 Swing (บรรทัด 41–46)

```pine
os := high[len] > upper ? 0 : low[len] < lower ? 1 : os[1]   // 41
top = os == 0 and os[1] != 0 ? high[len] : na                 // 43
btm = os == 1 and os[1] != 1 ? low[len]  : na                 // 46
```
- swing high: high ของแท่งย้อนหลัง `len` แท่ง สูงกว่าทุกแท่งหลังจากนั้น
- ยืนยันช้า `len` แท่งเสมอ (50 แท่งสำหรับ CHoCH, 3 แท่งสำหรับ IDM)

### 3.2 CHoCH (บรรทัด 75–103)

| # | เงื่อนไข | บรรทัด | ผล |
|---|---|---|---|
| C1 | มี swing ใหญ่ใหม่ (`if top` / `if btm`) | 75, 78 | จำระดับ `topy` / `btmy` และรีเซ็ต crossed |
| C2 | `close > topy and not top_crossed` | 83 | `os := 1` (ขาขึ้น) |
| C3 | `close < btmy and not btm_crossed` | 86 | `os := 0` (ขาลง) |
| C4 | `os != os[1]` | 91 | **วาด CHoCH** และเริ่ม leg ใหม่: `max = high`, `min = low` และล้าง IDM |

### 3.3 IDM (บรรทัด 113, 132)

| ฝั่ง | เงื่อนไข | ตัดสินด้วย |
|---|---|---|
| ขาขึ้น (113) | `low < sbtmy and not sbtm_crossed and os == 1 and sbtmy != btmy` | ไส้ล่าง |
| ขาลง (132) | `high > stopy and not stop_crossed and os == 0 and stopy != topy` | ไส้บน |

### 3.4 BOS (บรรทัด 121, 140)

| ฝั่ง | เงื่อนไข | ตัดสินด้วย |
|---|---|---|
| ขาขึ้น (121) | `close > max and sbtm_crossed and os == 1` | ราคาปิด และ **ต้องมี IDM ก่อน** |
| ขาลง (140) | `close < min and stop_crossed and os == 0` | ราคาปิด และ **ต้องมี IDM ก่อน** |

เกิด BOS แล้วล้าง IDM (บรรทัด 126, 145) → BOS ครั้งถัดไปต้องรอ IDM ใหม่

### 3.5 Sweep (บรรทัด 150, 154)

| ฝั่ง | เงื่อนไข |
|---|---|
| ขาขึ้น (150) | `high > max and close < max and os == 1 and n - max_x1 > 1 and showSweeps` |
| ขาลง (154) | `low < min and close > min and os == 0 and n - min_x1 > 1 and showSweeps` |

ไส้ทะลุ extreme ของ leg แต่ปิดกลับเข้ามา ไม่ต้องมี IDM และไม่เปลี่ยน state

### 3.6 Trailing max/min (บรรทัด 159–165)

อัปเดตหลังเช็ก BOS/Sweep เสมอ ทำให้ `max` / `min` ที่ใช้เช็กคือค่าของ **แท่งก่อนหน้า**

### 3.7 Extensions (บรรทัด 178–211, เฉพาะแท่งสุดท้าย)

| os = 1 (ขาขึ้น) | os = 0 (ขาลง) |
|---|---|
| ระดับ CHoCH ↓ ที่ `btmy` | ระดับ CHoCH ↑ ที่ `topy` |
| ระดับ BOS ↑ ที่ `max` | ระดับ BOS ↓ ที่ `min` |
| IDM ที่ `sbtmy` ถ้ายังไม่ถูกแทง (ยื่นไปข้างหน้า 15 แท่ง) | IDM ที่ `stopy` ถ้ายังไม่ถูกแทง |

---

## 4. Flow สรุป

```
ขาขึ้น:  CHoCH↑ (close > swing high ใหญ่)
            → IDM (ไส้ low < swing low เล็ก)
            → BOS↑ (close > high สูงสุดของ leg)
            → IDM → BOS↑ → ...
          ระหว่างทาง: x = ไส้ทะลุ high สูงสุดแต่ปิดกลับ
          จบเมื่อ: CHoCH↓ (close < swing low ใหญ่)
ขาลง:    กลับด้านทั้งหมด
```

ลำดับต่อแท่ง: Swing → CHoCH → IDM↑ → BOS↑ → IDM↓ → BOS↓ → Sweep → อัปเดต max/min → Extensions → Plot

---

## 5. ข้อสังเกตในไฟล์หลัก

1. **สี IDM ขาลงยัง hard-code เป็น `color.gray`** (บรรทัด 134–135) เปลี่ยนสี IDM ใน Settings แล้วฝั่งขาลงไม่เปลี่ยนตาม (ฝั่งขาขึ้นบรรทัด 115–116 ใช้ `idmCss` ถูกต้อง)
2. **Lag ของ swing:** CHoCH อิง swing ที่ยืนยันช้า 50 แท่ง
3. **ไม่ repaint ย้อนหลัง** แต่บนแท่งที่ยังไม่ปิด ป้ายอาจขึ้นแล้วหายได้
4. **object จำกัด 500 เส้น / 500 label** (บรรทัด 5)
5. `os` เริ่มที่ 0 ช่วงต้นกราฟก่อน CHoCH แรกจึงทำงานเป็นขาลง

## 6. ความสอดคล้องระหว่างไฟล์ (ใน branch `claude/main-analysis`)

ปรับ `editable.pine` และ `ANALYSIS.md` ให้ตรงกับไฟล์หลักแล้ว **ไม่ได้แก้ `original.pine` และไม่ได้แตะ `main`**

| ไฟล์ | สถานะ |
|---|---|
| `original.pine` | ไฟล์หลัก v5 ไม่แก้ |
| `editable.pine` | เปลี่ยนเป็น v5 และใช้ `input(...)` เหมือนไฟล์หลัก ชื่อ indicator ตามไฟล์หลักต่อท้าย "(Editable)" ให้แยกกันได้บนกราฟ logic เหมือนเดิม |
| `ANALYSIS.md` | แก้คำอธิบายเวอร์ชันเป็น v5 และระบุว่า `original.pine` คือไฟล์หลัก |

สิ่งที่ `editable.pine` ยังต่างจากไฟล์หลักโดยตั้งใจ: แยกเงื่อนไขเป็น `cond_*`, สี IDM ขาลงใช้ `idmCss`, `if not na(top)` และเพิ่ม `alertcondition` 8 ตัว
