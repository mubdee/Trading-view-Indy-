# MSNR × SMC + OB — MSNR × SMC + โซน MTF Supply & Demand / Orderblocks

- ไฟล์หลัก: [`main/msnr-smc-ob-main.pine`](./main/msnr-smc-ob-main.pine)
- ไฟล์สำรองสำหรับแก้: [`msnr-smc-ob-dev.pine`](./msnr-smc-ob-dev.pine) (ชื่อบนกราฟ `MSNR SMC OB DEV`)

ที่มา: [MSNR × SMC MAIN](../msnr-smc/main/msnr-smc-main.pine) (2026-11-24) ทั้งไฟล์ + [MTF S&D OB](../mtf-sd-ob/main/mtf-sd-ob-main.pine) ต่อท้ายเป็น MODULE

- ทุกอย่างของ MSNR × SMC เหมือนเดิม (เลเวล MSNR H1/H4 · Sweep → MSS → Limit · Score · SMT · แท่งเทียน TF ใหญ่ · Dashboard)
- **เพิ่มโซน OB** (TF ชาร์ต + Timeframe 2 ค่าเริ่ม H4) — เงื่อนไขเดียวกับ [MTF S&D OB](../mtf-sd-ob/README.md) · ไม่มีขอบกล่อง · ไม่มีตาราง Stats ของ OB
- โซน OB **ยังไม่ผูกกับ Setup / Score** (แสดงผลอย่างเดียว)
- OB คำนวณเฉพาะช่วง "คำนวณย้อนหลัง" เหมือน Engine (ไม่ให้สคริปต์ช้า) · ค่าเริ่ม 6 โซนต่อฝั่งต่อ TF
- Settings หมวด `OB · การวาด & ป้าย` · `OB · ตัวกรอง` · `OB · TF ชาร์ต / Timeframe 2` อยู่ท้ายสุด
- Alert "Any alert() function call" รวม OB ใหม่ / OB ถูกแตะ (ขึ้นต้น `MSNR×SMC OB:`)
