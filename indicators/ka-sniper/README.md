# KA SNIPER — Top-down H4 → M1 · ออเดอร์เดียวที่ดีที่สุด

ไฟล์: [`ka-sniper.pine`](./ka-sniper.pine) (ชื่อบนกราฟ `KA SNIPER`) · แยกจาก KA PRO

## ทำงานยังไง
- Engine คำนวณบนข้อมูล **M5 เสมอ** (+ H4 / H1 / M15 / M1 / D) → ออเดอร์เดียวกันไม่ว่ากราฟจะเป็น M1 · M5 · M15
- ทีละ 1 ออเดอร์:
  1. **Bias** = Trend H4 + H1 ตรงกัน (BOS / CHoCH ต้องปิดด้วยเนื้อเทียน)
  2. **KEY**: swing H4 · H1 · M15 · PDH/PDL · Asia H/L · London H/L
  3. **กวาด KEY** ฝั่งตรงข้าม Bias (ไส้เลยแล้วปิดกลับภายใน 3 แท่ง M5)
  4. **CISD**: แท่งปิดทะลุด้วยเนื้อเทียน + Displacement / FVG
  5. **Limit ที่ OTE M5** (0.62–0.79 · เลือกเลเวลที่ซ้อน FVG / CISD / KEY) · SL เลยไส้ · TP = Liquidity ฝั่งตรงข้าม (RR 2–5)
- Score 6 ข้อ (M15 · M1 · KEY แข็ง · Killzone · Premium/Discount · Entry ซ้อน) · ค่าเริ่มต้นต้อง ≥ 3 · ยืนยันใน Killzone · แพ้ 2 ไม้/วัน หยุด
- อ่านย้อนหลัง 5 วันเทรด (~1 สัปดาห์) · ไม่ repaint (ใช้แท่ง M5 ที่ปิดแล้ว)
