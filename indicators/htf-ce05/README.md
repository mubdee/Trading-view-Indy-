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

## 3) Key Level = POI (2026-11-20 · ค่าเริ่มต้นเปิด)
**Key Level = ระดับ MSNR ที่ซ้อนอยู่ใน OB / Demand-Supply ทุก TF ที่เปิด** (กลุ่มตั้งค่า "Key Level")
- **ระดับ MSNR (TF กราฟ):** QML (ไหล่ซ้าย Quasimodo) · SBR/RBS (body ของ swing ที่ถูกปิดทะลุ) · A/V (body ของ swing) · Swing length (MSNR) = 3
- **โซน:** OB = แท่งสวนทางสุดท้ายก่อนแท่งที่ปิดทะลุมัน · D/S = base (body ≤ 0.5 ATR) + impulse (body ≥ 1 ATR) · โซนที่ถูกปิดทะลุ = ใช้ไม่ได้
- **TF ที่ซ้อน:** TF กราฟ (ปิด) · TF 1 = H1 (เปิด) · TF 2 = H4 (ปิด) · `ทุก TF ที่เปิด` หรือ `อย่างน้อย 1 TF`
- **Fresh:** ระดับที่ราคาแตะแล้ว = ถูกใช้ ไม่นับอีก · **เลือกโซนสุดท้าย** = ระดับลึกสุดใกล้ต้นขา (หรือ "ใกล้ราคาสุด")
- **ช่วงที่หา Key:** Hidden = Origin → High/Low ของขา MSS · Single = ปลายไส้ HTF → single liQ
- **LQ:** ต้องมี swing ย่อยก่อนถึง Key อย่างน้อย 1 (ตั้ง 0 = ปิด)
- **เข้า:** เงื่อนไขเดิมครบ + ราคาแตะ Key → **Limit ที่ Key Level** (หรือปิดแท่งสัญญาณ) · ไม่มี Key = `ไม่มี Key Level` ไม่เข้า
- เส้นชมพู = Key Level พร้อมป้ายชนิด + โซนที่ซ้อน เช่น `Key QML · OB H1`

## การแสดงผล
- เส้นมีสี = เซ็ตอัพที่ยังรอ · **สีเทาจาง** = จบแล้ว (ใช้แล้ว / Invalid / หมดอายุ / ถูกแทนที่)
- กล่องเขียว (TP) / แดง (SL) + ป้าย Entry · SL · TP · RR · ลูกศร = แท่งเข้า
- ออเดอร์จบ → `TP ✓` / `SL ✗` / `หมดเวลา` แล้วกล่องเป็นสีเทา
- Dashboard: สถานะ + Level ของ Hidden ▲▼ / Single ▲▼ · เตือนถ้า HTF ไม่สูงกว่า TF กราฟ

## Alerts
Hidden BUY / SELL · Single BUY / SELL · Any signal
