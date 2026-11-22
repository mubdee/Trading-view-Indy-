# MSNR × SMC — POI MSNR (H1/H4) → Sweep → MSS → Limit ที่เลเวล MSNR

- ไฟล์หลัก: [`main/msnr-smc-main.pine`](./main/msnr-smc-main.pine)
- ไฟล์สำรองสำหรับแก้: [`msnr-smc-dev.pine`](./msnr-smc-dev.pine) (ชื่อบนกราฟ `MSNR × SMC DEV`)

เปิดกราฟเล็ก (M5 / M15) แล้วเห็นเลเวลและแท่งเทียนของ H1 / H4 บนกราฟเดียว

1. **POI** — เลเวล MSNR ของ H1/H4 ที่ยัง Fresh (A/V · RBS/SBR · OCL/Gap · ต้องตรง Storyline H4)
2. **Sweep (TF ชาร์ต)** — ไส้ทะลุ swing แล้วปิดกลับ ใกล้เลเวล HTF · swing ≤ 25 แท่ง · เว้น ≥ 1 แท่งคั่น · นับ Key liquidity (PDH/PDL · Asia H/L · EQH/EQL)
3. **MSS** — ปิดเนื้อเทียนทะลุ high/low ของขาที่ส่งไป sweep ภายใน 20 แท่ง · ไส้เลยปลายไส้ sweep ก่อน = ยกเลิก · ตัวเลือก Displacement / FVG
4. **Score 0-6** — Key · Disp · FVG · Entry ทับ HTF · Discount/Premium · Killzone → เกรด A+ / A / B / C · ตั้ง Limit เฉพาะ Score ≥ ที่กำหนด
5. **Entry** — Limit ที่เลเวล MSNR ในขา MSS (RBS/SBR = body แท่ง swing ที่ระดับ MSS, V/A/QM/OCL ของ TF ชาร์ต) · SL เลยปลายไส้ sweep · TP = เลเวล HTF Fresh ฝั่งตรงข้าม (≥ 1.5R)
6. **แท่งเทียน TF ใหญ่** ทางขวาของกราฟ — H4 4 แท่ง + H1 6 แท่ง (live) · เส้น Open · ตัว S = sweep บนแท่ง TF ใหญ่ · เวลาที่เหลือก่อนแท่งปิด
7. Dashboard: Storyline · Next R/S · Setup · SL/TP · Score · Stats (ทั้งหมด + เฉพาะ A/A+)

Alert: MSS → Buy/Sell Limit · Limit Filled · หรือ "Any alert() function call" (มี Entry / SL / TP / RR / Score ในข้อความ)

**DEV (2026-11-22 · ยังไม่ย้ายเข้า MAIN):** เส้น SMT บนกราฟ (swing ทองเทียบ swing เงินเวลาเดียวกัน · Sweep ที่เงินไม่ยืนยันขึ้นป้าย `Sweep · SMT`) · SMT divergence ทอง/เงิน + PO3 (Entry ฝั่งถูกของ Open H4) → Score เต็ม 8 (A+ ≥7) · บริหารไม้ — TP1 (Roadblock = เลเวล MSNR ฝั่งตรงข้ามตัวแรก หรือ 1R) ปิด 50% แล้วเลื่อน SL ไป Entry + 30 points · Stats คำนวณตามโหมด · Lot ตามความเสี่ยง USD (XAUUSD 1 lot = 100 oz) · SL ไกลสุด (points) แบบ SFP SBR/RBS · Dashboard แถว Lot
