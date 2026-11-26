# MTF Supply & Demand / Orderblocks — โซน OB 3 แท่ง (TF ชาร์ต + Timeframe 2)

- ไฟล์หลัก: [`main/mtf-sd-ob-main.pine`](./main/mtf-sd-ob-main.pine)
- ไฟล์สำรองสำหรับแก้: [`mtf-sd-ob-dev.pine`](./mtf-sd-ob-dev.pine) (ชื่อบนกราฟ `MTF S&D OB DEV`)

สร้างใหม่ตามเงื่อนไขของ JP7FX "Supply & Demand / Orderblocks - Multi TimeFrame" (ต้นฉบับเป็น protected source)

1. **แท่ง OB → แท่งกลับตัว → แท่ง 3 ทิ้ง imbalance**
   - Demand: แท่ง 2 เขียว · Low แท่ง 3 > High แท่ง OB (Wick→Wick) หรือใช้ตัวเนื้อแท่ง 3 (Wick→Open)
   - Supply: แท่ง 2 แดง · High แท่ง 3 < Low แท่ง OB (กลับด้าน)
2. **กรอง** — 2 แท่งก่อนหน้าห้ามทำไส้ต่ำกว่า (Demand) / สูงกว่า (Supply) แท่ง OB · แท่ง OB สีอะไรก็ได้ (ติ๊ก "สีตรงข้าม" = แบบ JP7FX)
3. **วาดโซน** จากไส้ หรือ ตัวเนื้อ ของแท่ง OB · ไม่มีขอบกล่อง (เห็นไส้เทียน) · เส้น 50% (ลบเมื่อราคาแตะ)
4. **สถานะ** — Fresh → Mitigated (แตะ) → ลบเมื่อไส้ทะลุทั้งโซน / ปิดเลยโซน · เลือกแสดงเฉพาะ Fresh ได้
5. **TF ชาร์ต + Timeframe 2** (ค่าเริ่ม H4) · ไม่ repaint · ระบายสีแท่ง OB
6. Alert: OB ใหม่ / OB ถูกแตะ ("Any alert() function call")
