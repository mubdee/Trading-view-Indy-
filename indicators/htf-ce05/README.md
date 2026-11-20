# HTF CE0.5 — Hidden Liquidity + Single Liquidity (CE 0.5)

- ไฟล์หลัก (ห้ามแก้): [`main/htf-ce05-main.pine`](./main/htf-ce05-main.pine)
- ไฟล์สำรองสำหรับแก้: [`htf-ce05-dev.pine`](./htf-ce05-dev.pine) (สำเนาเดียวกัน · ชื่อบนกราฟ `HTF CE0.5 DEV`)

## วิธีใช้
1. เปิดไฟล์ `.pine` → กด **Raw** → คัดลอกทั้งหมด
2. TradingView → Pine Editor → วางโค้ด → **Add to chart**
3. แนะนำ: กราฟ **M5** · HTF = **60 (H1)** · SL buffer ทอง 0.3–0.5

## 1) Hidden Liquidity
1. **MSS** — ราคาปิดทะลุ swing (มี FVG ยืนยัน · ปิดได้)
2. **Key** — swing ต้นขาที่ทำ MSS
3. **hidden liQ** — low/high ของแท่งถัดจาก Key (หรือแท่งต้น FVG)
4. **IDM** — swing ย่อยตอนราคาย่อ ต้องถูกกินก่อน (ปิดได้)
5. **Entry** — ราคากวาด hidden liQ และไม่ปิดเลย Key → เข้าที่ราคาปิดแท่งสัญญาณ
6. **SL** = Key (+ buffer) · **TP** = High/Low หลัง MSS หรือ RR คงที่

## 2) Single Liquidity + CE 0.5 (HTF)
1. บน HTF หาแท่งไส้ยาวที่เป็น swing → **CE 0.5** = กึ่งกลางไส้
2. **single liQ** = low/high ของแท่งถัดไป (ต้องอยู่ก่อนถึง CE)
3. **Entry** — ราคากวาด single liQ แล้วถึง CE และไม่ปิดเลยปลายไส้
4. **SL** = ปลายไส้ HTF (+ buffer) · **TP** = RR คงที่

## การแสดงผล
- เส้นมีสี = เซ็ตอัพที่ยังรอ · **สีเทาจาง** = จบแล้ว (ใช้แล้ว / Invalid / หมดอายุ / ถูกแทนที่)
- กล่องเขียว (TP) / แดง (SL) + ป้าย Entry · SL · TP · RR · ลูกศร = แท่งเข้า
- ออเดอร์จบ → `TP ✓` / `SL ✗` / `หมดเวลา` แล้วกล่องเป็นสีเทา
- Dashboard: สถานะ + Level ของ Hidden ▲▼ / Single ▲▼ · เตือนถ้า HTF ไม่สูงกว่า TF กราฟ

## Alerts
Hidden BUY / SELL · Single BUY / SELL · Any signal
