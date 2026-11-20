# Sweep + MSS — ยืนยันโครงสร้างอย่างเดียว

- ไฟล์หลัก: [`main/sweep-mss-main.pine`](./main/sweep-mss-main.pine)
- ไฟล์สำรองสำหรับแก้: [`sweep-mss-dev.pine`](./sweep-mss-dev.pine) (ชื่อบนกราฟ `Sweep + MSS DEV`)

ไม่มีจุดเข้า / SL / TP · แสดงเฉพาะโครงสร้าง

1. **Sweep** — ไส้เลย swing แล้วปิดกลับ (swing ของ TF กราฟ ภายใน 60 แท่ง + swing ของ TF ใหญ่ เช่น H1) · แท่ง sweep ต้องห่างจาก swing ≥ 2 แท่ง · ป้ายบอก TF ที่ถูกกวาด เช่น `Sweep M30,H1`
2. **ระดับ MSS** — จุดสูง/ต่ำสุดของขาที่ส่งราคาไป sweep (ระหว่าง swing ที่ถูกกวาดกับแท่ง sweep)
3. **MSS ✓** — ปิดทะลุระดับภายใน 20 แท่ง แล้วแท่งถัดไปต้องวิ่งต่อ (ปิดเลย High/Low แท่งทะลุ) · ปิดกลับ = แค่ sweep · ทำ High/Low ใหม่เลยไส้ sweep ก่อน = ยกเลิก
4. Sweep ที่ไม่เกิด MSS: ลบทิ้ง (เลือกแสดงจางได้)

Alert: Bullish / Bearish Sweep · Bullish / Bearish MSS ✓ (หรือ "Any alert() function call")
