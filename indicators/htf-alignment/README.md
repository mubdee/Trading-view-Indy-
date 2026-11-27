# HTF Alignment PRO — แท่ง HTF ทางขวา + Quarterly + SMT

- ไฟล์หลัก: [`main/htf-alignment-main.pine`](./main/htf-alignment-main.pine)
- ไฟล์สำรองสำหรับแก้: [`htf-alignment-dev.pine`](./htf-alignment-dev.pine) (ชื่อบนกราฟ `HTF Alignment PRO DEV`)

สร้างใหม่ตาม "HTF Alignment [CHAFFEY]" จากคลิปรีวิว (ต้นฉบับเป็น protected source) · 2026-11-27

1. **แท่ง HTF ทางขวากราฟ** — 6 ช่อง (ค่าเริ่ม 15m · 30m · 1H · 4H · D · W) ติ๊กเปิด/ปิด + ตั้งจำนวนแท่งได้ · หัวกลุ่มแสดงชื่อ TF + เวลาที่เหลือ (`n/a` เมื่อตลาดปิด / ดูย้อนหลัง)
2. **Daily Reset Alignment** — แท่ง HTF ระหว่างวันนับจากเวลาเปิดวันเทรด (18:00 NY สำหรับ Futures)
3. **HTF Quarterly Alignment** — กลุ่ม 6H (Daily Cycles) + 90m Cycles · ป้าย Q1–Q4 (Above/Below · Label Candles) · Quarterly Alignment = เริ่มกลุ่มที่ Q1
4. **SMT 2 สินทรัพย์** (ค่าเริ่ม YM + ES สำหรับ MNQ)
   - บนกราฟ: เส้นเชื่อม High/Low + ป้าย `SMT w/ YM | 4H, 6H` หรือ `SMT w/ 2 Assets | D` (TF ที่ใช้สวิงเดียวกันรวมในป้ายเดียว)
   - บนแท่ง HTF: เส้นสีส้มระหว่างแท่ง
5. **HTF FVG** สีเทา (เฉพาะที่ยังไม่ถูกเติม)
6. **CISD** บนแท่ง HTF (แดง = Bear · น้ำเงิน = Bull) + ลากลงบนกราฟตาม TF ที่เลือก (ค่าเริ่ม D)
7. **เส้น High/Low ของ HTF ใหญ่สุด** (W) จากจุดที่ทำ High/Low ไปถึงแท่ง HTF

หมายเหตุ: ใช้กับกราฟ TF เล็กกว่ากลุ่ม HTF (กลุ่มที่ TF ≤ กราฟจะถูกซ่อน) · ถ้าใช้กับทอง ให้เปลี่ยน Compare Asset เป็นคู่ที่วิ่งไปด้วยกัน เช่น XAGUSD
