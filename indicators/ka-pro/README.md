# KA PRO — Key Alchemist PRO (TS × CISD × MSNR)

- ไฟล์หลัก: [`main/ka-pro-main.pine`](./main/ka-pro-main.pine) (ชื่อบนกราฟ `KA PRO`)
- ไฟล์สำรองสำหรับแก้: [`ka-pro-dev.pine`](./ka-pro-dev.pine) (ชื่อบนกราฟ `KA PRO DEV`) · แก้/ทดลองในไฟล์นี้ก่อน แล้วค่อยย้ายเข้าไฟล์หลัก

อ้างอิงเซ็ตอัพ Key Alchemist (amirxau77) + MSNR (SL 10 PIPS) · ใช้กับ XAUUSD M5 (TS = H1 · Bias = H4)

## เซ็ตอัพ
1. **TS (Turtle Soup)** — ไส้กวาด KEY/.LVL ของ TF ใหญ่ (swing H1/H4 · CE 0.5 · PDH/PDL · PWH/PWL · Asia H/L · โมเดล 3 แท่ง) แล้วปิดกลับ → CISD / MSS → Limit ใน OTE 0.5–0.79 ที่เลเวลซ้อนกัน
2. **KR (Key Alchemist Model)** — หลัง TS รอ pullback ทำ IDM → ราคากวาด IDM → Limit ที่ KEY [CE 0.5] / จุดสุด (Hidden liQ)
3. **RE (Re-entry)** — BOS ไปทางเดียวกัน → Limit ที่ key level ใหม่
4. **CT (Continuation)** — Bias ชัด + BOS บน TF ชาร์ต → Limit ที่ต้นขา BOS ที่ทับเลเวล MSNR (SBR/RBS/QM/OCL/EOB/TL)

## Filter / ความลื่น
- Bias 2 ชั้น (H4 + H1 ต้องตรงกัน) · โครงสร้าง MTF H4 → H1 → M15 → M5 → M1 (ต้องตรงทิศ ≥ 3/5)
- Score 0-8 · Killzone เวลา New York (London 02-05 · NY 08:30-11 · Silver Bullet 03-04 / 10-11 / 14-15)
- PO3: Premium/Discount เทียบ NY Midnight Open
- คุมไม้ทั้งวัน: ขาดทุนรายวัน −2R · แพ้ติด 2 · 4 ไม้/วัน · พักหลัง SL 6 แท่ง · Bias ต้องนิ่ง 6 แท่ง · ไม่เปิดไม้สวนทาง · ไม่ตั้งซ้ำ · Bias เปลี่ยน → ยกเลิก Limit ที่สวน

## ช่วงวิเคราะห์ (จากแท่งไหน → ถึงแท่งไหน) · DEV v9.3
- ตั้งค่า → กลุ่ม **ช่วงวิเคราะห์** → `วิเคราะห์ช่วง`
  - **ย้อนหลัง N แท่งล่าสุด** — แบบเดิม (ถึงแท่งล่าสุด)
  - **เลือกวันเวลา (จาก → ถึง)** — ใส่วันเวลาเริ่ม/จบ · ติ๊ก `ถึง = แท่งล่าสุด` ถ้าอยากให้วิ่งถึงปัจจุบัน
  - **นับแท่งย้อนหลัง (จาก → ถึง)** — เช่น จาก 800 ถึง 300 = วิเคราะห์แท่งที่ 800 ถึง 300 นับย้อนจากแท่งล่าสุด
- หา Setup เฉพาะในช่วง · จบช่วง → ยกเลิก Limit ที่ยังไม่โดน (ปิดได้) ไม้ที่เข้าแล้วปล่อยจนปิด · ช่วงถูกแรเงาบนกราฟ
- Dashboard แถว "ช่วงวิเคราะห์" บอกช่วงจริง + จำนวนแท่ง · เลือกได้ไม่เกิน 2500 แท่งที่กราฟโหลด

## หน้าจอ
- เส้นทุกเส้นยึดเวลาแท่งเทียนจริง (xloc.bar_time) · ป้ายขวารวมกันอัตโนมัติไม่ทับกัน
- ตาราง MTF (บนขวา) · Dashboard (ล่างขวา) · **ตารางแผนตั้งไม้ (ล่างซ้าย)**: ✅ ตั้งเลย · 📍 ถืออยู่ · ⏳ รอเงื่อนไข · ✗ ห้ามเทรด + Entry / SL / TP
- ความเร็ว: calc_bars_count 2500 · หา Setup ย้อนหลัง 1200 แท่ง · ถ้าช้าให้ลดค่าหรือปิด MTF

## ประวัติ
- **v9.9 DEV (2026-11-25)** — KEY swing M15 + M30 · Reversal: M5 MSS + KEY ซ้อน ≥ 2 · KR เข้าที่ QM key
- **v9.10 DEV (2026-11-25)** — Quarterly Theory (AMDX · Daye): Quarter วัน / 90 นาที · True Open · Manipulation Q1 → คะแนน/บังคับ · ไม่เทรด Q1 Asia · KEY High/Low Q1 · ไม้ REV ไม่ติด filter MTF
- **v9.8 DEV (2026-11-25)** — ตาม ICT guide: EQH/EQL · Opening Range London/NY · OB (ICT) + Breaker · NY AM 07-10 · London Close · Macro · NY 9:30 · SMT กับ DXY
- **v9.7 DEV (2026-11-25)** — ไม่ตั้ง BE (ค่าเริ่มต้น) · โครงสร้าง MTF / Bias ใช้แท่ง TF ใหญ่ที่ปิดแล้ว (ไม่ repaint)
- **v9.6 DEV (2026-11-25)** — SMT ทอง×เงิน (+1 คะแนน / บังคับได้) · แท่งกวาดต้องปฏิเสธชัด · Entry FVG ของขา displacement
- **v9.5 DEV (2026-11-25)** — ไม้เงา (จำลองไม้ที่ถูกข้าม → filter ไหนตัดไม้ดี/เสียกี่ R) · Reversal: สวน Bias ได้ถ้า M15+M5 กลับทิศ (ค่าเริ่มต้นปิด)
- **v9.4 DEV (2026-11-25)** — Bias ใช้โครงสร้าง MTF H4 + H1 (ตรงกับตาราง) · Dashboard แถว "ข้ามเพราะ" (MAIN = v9.2)
- **v9.3 DEV (2026-11-25)** — เลือกช่วงวิเคราะห์ จาก → ถึง (ยังอยู่ใน DEV · MAIN = v9.2)
- **v9.2 (2026-11-25)** — ตารางแผนตั้งไม้ · ป้ายกล่องบอก SELL/BUY LIMIT ราคา (MAIN = DEV)
- v9.1 — เร็วขึ้น (ลดแท่งที่คำนวณ / MTF / ลูป)
- v9 — Smooth: Killzone NY · คุมไม้ทั้งวัน
- v7 — ไล่โครงสร้าง H4 → M1 · v6 — เส้นตรงแท่งเทียน · PO3 · Asia H/L
- v2–v5 — Bias · OTE · KR/RE/CT · ไม้ต่อไป
