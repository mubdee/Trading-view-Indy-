# SFP Pro — Sweep + MSS

ไฟล์สคริปต์: [`sfp-pro-sweep-mss.pine`](./sfp-pro-sweep-mss.pine) (Pine Script v6)

## วิธีใช้
1. เปิดไฟล์ `.pine` → กด **Raw** → คัดลอกทั้งหมด
2. TradingView → Pine Editor → วางโค้ด → **Add to chart**

## เงื่อนไข (ตัวอย่างฝั่ง Bearish)
1. **Sweep** — ไส้เลย swing high เดิม แล้วปิดกลับลงมาใต้ระดับ → ขึ้น "SFP ?" จาง ๆ และเส้นประที่ swing low ล่าสุด
2. **MSS** — ภายใน N แท่ง (ค่าเริ่มต้น 20) ต้องมีแท่งที่ **ราคาปิด (เนื้อเทียน) ทะลุ swing low ล่าสุด** → SFP สีเต็ม + เส้น MSS + โซน
3. **ยกเลิก** — ราคาปิดเหนือปลายไส้ sweep หรือไม่ทะลุภายในเวลา → ลบทิ้ง

ฝั่ง Bullish กลับด้านกัน (กวาด swing low → ปิดทะลุ swing high ล่าสุด)

## ค่าที่ควรรู้
| Setting | ค่าเริ่มต้น | ความหมาย |
|---|---|---|
| Swing (liquidity) Left / Right | 5 / 1 | swing ที่ใช้เป็น liquidity |
| Swing ล่าสุด Left / Right | 2 / 1 | swing ที่ต้องทะลุเพื่อ MSS |
| ทะลุด้วย | Close | `Full Body` = ทั้ง open และ close ต้องเลยระดับ |
| แท่ง Sweep ต้องปิดกลับด้วย | Close | `Body` = ทั้งเนื้อเทียนต้องอยู่ในระดับ |
| แสดง SFP ล่าสุด (ต่อฝั่ง) | 1 | กราฟสะอาด เหลือเฉพาะตัวที่ใช้ได้ |
| ใช้ Volume กรอง SFP | ปิด | กรองด้วย volume จาก lower timeframe |

## Alerts
- Bullish / Bearish Sweep (รอ MSS)
- Bullish / Bearish SFP + MSS (confirmed)
- Retest Bull / Bear SFP Zone
