# SFP Pro X — Liquidity Sweep + MSS

ไฟล์สคริปต์: [`sfp-pro-sweep-mss.pine`](./sfp-pro-sweep-mss.pine) (Pine Script v6)

## วิธีใช้
1. เปิดไฟล์ `.pine` → กด **Raw** → คัดลอกทั้งหมด
2. TradingView → Pine Editor → วางโค้ด → **Add to chart**

## ลำดับเงื่อนไข (ตัวอย่างฝั่ง Bearish)
1. **Liquidity** — swing high, EQH, PDH, Asia High หรือ HTF High
2. **Sweep** — ไส้เลยระดับแล้วปิดกลับลงมา (ลึกไม่เกิน 1.5 ATR) → "SFP ?" จาง ๆ
3. **ตัวกรอง** — ต้องไม่สวน HTF Bias, อยู่ใน Session (ถ้าเปิด), Volume ผ่าน (ถ้าเปิด)
4. **MSS** — ภายใน 10 แท่ง ต้องปิดทะลุ swing low ล่าสุดด้วยเนื้อเทียน **และมีแรง** (body ≥ 0.8 ATR หรือทิ้ง FVG)
5. **Entry** — วาด OB + FVG ในขา MSS พร้อม TP (2R) / SL (เหนือไส้ sweep)
6. **ลบออก** — เมื่อราคาปิดเลยปลายไส้, ถึง TP, หรือเก่าเกิน 150 แท่ง

ฝั่ง Bullish กลับด้านกัน

## ค่าเริ่มต้นสำคัญ
| Setting | ค่าเริ่มต้น | หมายเหตุ |
|---|---|---|
| ใช้ Liquidity | All | `Key only` = เฉพาะ EQH/EQL, PDH/PDL, Asia, HTF |
| ลึกไม่เกิน | 1.5 ATR | 0 = ปิด |
| Displacement | Body ATR หรือ FVG | Off = ทะลุอย่างเดียวพอ |
| ต้องทะลุภายใน | 10 แท่ง | |
| HTF Bias | EMA 50 บน H4 | Off / EMA / Premium-Discount / ทั้งคู่ |
| Session filter | ปิด | London 07–10, NY 12:30–15:30 (UTC) |
| โซนเข้า | OB + FVG | หรือ Sweep Wick แบบเดิม |
| RR | 2 | ใช้ทั้งเส้น TP และสถิติ |

## Dashboard
HTF Bias · Session · SFP ล่าสุด + ชนิด liquidity · ระดับ Liquidity/MSS · Status · สถิติ W/L, Win rate, Net R · จำนวน setup ที่ active

> สถิติคำนวณแบบง่าย: เข้าที่ราคาปิดของแท่ง MSS, SL เหนือ/ใต้ไส้ sweep, TP = RR ที่ตั้ง — ถ้าแท่งเดียวแตะทั้ง TP และ SL นับเป็นแพ้

## Alerts
- Bullish / Bearish Sweep (รอ MSS)
- Bullish / Bearish SFP + MSS (มีราคา SL/TP ในข้อความ)
- Retest Bull / Bear Entry Zone (OB/FVG)
