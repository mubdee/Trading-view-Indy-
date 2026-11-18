# SFP Pro X — Sweep + MSS + OB Confluence

ไฟล์สคริปต์: [`sfp-pro-sweep-mss.pine`](./sfp-pro-sweep-mss.pine) (Pine Script v6)

## วิธีใช้
1. เปิดไฟล์ `.pine` → กด **Raw** → คัดลอกทั้งหมด
2. TradingView → Pine Editor → วางโค้ด → **Add to chart**

## ลำดับเงื่อนไข (ตัวอย่างฝั่ง Bearish)
1. **Liquidity** — swing high, EQH, PDH, Asia High หรือ HTF High
2. **Sweep** — ไส้เลยระดับแล้วปิดกลับลงมา → "SFP ?" จาง ๆ
3. **MSS** — ภายใน 20 แท่ง ต้องมีแท่งที่ราคาปิด (เนื้อเทียน) ทะลุ swing low ล่าสุด
4. **Entry (OB + Confluence)** — หา OB ในขา MSS แล้วหาระดับที่ **ซ้อนอยู่ใน OB**
   - **A-level** (MSNR) — ยอดของ line chart ที่ยัง Fresh
   - **SBR** — แนวรับเดิม (body ของ swing low ที่ถูก MSS ทะลุ) กลายเป็นแนวต้าน
   - **QM** — ไหล่ซ้ายของ Quasimodo (body ของ swing high ก่อน low ที่ถูกทะลุ)
   - เส้น Entry เขียนว่ามีอะไรซ้อนกันบ้าง เช่น `Entry · OB + QM + A`
   - ถ้าไม่มีระดับไหนอยู่ใน OB → ใช้ขอบ OB ด้านใกล้ราคา · ถ้าไม่มี OB → ใช้ระดับที่ใกล้ราคาที่สุด
   - เมื่อไส้แตะ → เปลี่ยนเป็น **Tested** + alert
5. **TP / SL** — SL เหนือไส้ sweep, TP = RR ที่ตั้ง (คิดจากราคา Entry)
6. **ลบออก** — เมื่อราคาปิดเลยปลายไส้, ถึง TP, หรือเก่าเกิน 150 แท่ง

ฝั่ง Bullish กลับด้าน (V-level, RBS, QM ฝั่งล่าง)

## ตัวกรองเสริม (ปิดไว้เป็นค่าเริ่มต้น เพื่อให้สัญญาณไม่น้อยเกิน)
| Setting | ค่าเริ่มต้น | เปิดแล้วจะ |
|---|---|---|
| Displacement | Off | แท่ง MSS ต้อง body ≥ x ATR หรือทิ้ง FVG |
| HTF Bias | Off | EMA / Premium-Discount บน HTF |
| Session | Off | เฉพาะ London / NY |
| ลึกไม่เกิน | 0 (ปิด) | ตัด sweep ที่ลึกเกิน x ATR |
| ใช้ Liquidity | All | `Key only` = เฉพาะ EQH/EQL, PDH/PDL, Asia, HTF |
| Volume (LTF) | Off | ตัด SFP ที่ volume เลยระดับสูงเกิน |

## โซนเข้า (Entry mode)
`OB + Confluence` (default) · `MSNR` · `MSNR + FVG` · `OB + FVG` · `Sweep Wick`

## Dashboard
HTF Bias · Session · SFP ล่าสุด + ชนิด liquidity · Liquidity/MSS · Entry · Status · สถิติ W / L / Miss, Win rate, Net R · Active

> สถิติคิดแบบ limit order ที่ราคา Entry: ถ้าราคาไม่กลับมาแตะ Entry แต่วิ่งถึง TP ก่อน = **Miss** · ถ้าแท่งเดียวแตะทั้ง Entry และ SL = แพ้

## Alerts
- Bullish / Bearish Sweep (รอ MSS)
- Bullish / Bearish SFP + MSS (มี Entry / SL / TP ในข้อความ)
- Retest Bull / Bear Entry (MSNR / OB / FVG)
