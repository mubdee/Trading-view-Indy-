# SFP Pro X — Sweep + MSS + OB Confluence

ไฟล์สคริปต์: [`sfp-pro-sweep-mss.pine`](./sfp-pro-sweep-mss.pine) (Pine Script v6)

## วิธีใช้
1. เปิดไฟล์ `.pine` → กด **Raw** → คัดลอกทั้งหมด
2. TradingView → Pine Editor → วางโค้ด → **Add to chart**

## ลำดับเงื่อนไข (ตัวอย่างฝั่ง Bearish)
1. **Liquidity** — swing high, EQH, PDH, Asia High หรือ HTF High
2. **Sweep** — ไส้เลยระดับแล้วปิดกลับลงมา → "SFP ?" จาง ๆ
   - แท่ง sweep ต้องเว้นห่างจากแท่ง swing อย่างน้อย 1 แท่ง (แท่งที่ติดกับ swing กวาด = ไม่นับ) · ปรับที่ "เว้นแท่งระหว่าง swing กับแท่ง Sweep อย่างน้อย" (0 = ปิด) · ใช้กับ Swing, EQH/EQL, Asia H/L
3. **MSS** — ภายใน 20 แท่ง ต้องมีแท่งที่ราคาปิด (เนื้อเทียน) ทะลุ swing low ล่าสุด
4. **Entry (OB + Confluence)** — หา OB ในขา MSS แล้วหาระดับที่ **ซ้อนอยู่ใน OB**
   - **A-level** (MSNR) — ยอดของ line chart ที่ยัง Fresh
   - **SBR** — body ของแท่ง Sweep (ก้น body ฝั่ง Bear / หัว body ฝั่ง Bull) = แนวรับเดิมที่กลายเป็นแนวต้าน · ไฟล์สำรอง (DEV) ใช้นิยามนี้แล้ว (ไฟล์หลักยังเป็นแบบเดิม: body ของแท่งที่เป็นระดับ MSS)
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

---

## DEV — Zone Filter (OB / Demand-Supply)
ไฟล์ `sfp-pro-sweep-mss.pine` (ตัว DEV) เพิ่ม:
- **อายุ Liquidity** (ค่าเริ่มต้น 10 แท่ง) — นับ SFP เฉพาะตอนกวาด swing / EQH / EQL ที่เกิดภายใน 10 แท่งก่อนหน้า · PDH/PWH/Asia/HTF sweep ได้ตลอด (เลือกให้นับอายุด้วยได้) · swing เก่ายังใช้เป็นเป้า TP
- **Demand / Supply** — base (body ≤ 0.5 ATR, 1–3 แท่ง) + impulse (body ≥ 1 ATR) · ระบุชนิด RBD/DBD/DBR/RBR
- **ตัวกรองโซน** (ทั้ง OB และ D/S): Fresh · Displacement · Premium/Discount ของขา · ขนาดไม่เกิน 3 ATR
- **คะแนน ★** = Fresh + Displacement + P/D + ขนาดพอดี + อยู่ที่จุด Sweep + ระดับ A/V·SBR·QM ในโซน (สูงสุด 3) + OB ซ้อน D/S
- เลือกโซนที่ ★ สูงสุด (ต้อง ≥ 3) แล้วเข้าที่ระดับ confluence ในโซนนั้น เช่น `Entry · Supply RBD + OB ★7 + QM + A`
- **TP = Liquidity เท่านั้น** — ไม่ใช้ RR คงที่แล้ว · เป้าคือ liquidity ฝั่งตรงข้ามที่ยังไม่ถูกกวาด: Swing, EQH/EQL, PDH/PDL, PWH/PWL, Asia, HTF, internal swing (iH/iL) · เลือก `ใกล้สุด` หรือ `Key liquidity ก่อน` · ข้ามเป้าที่ใกล้จน RR < ค่าที่ตั้ง (0 = ไม่กรอง) · ไม่มีเป้า = ไม่เข้าออเดอร์
- **โซนกรองด้วย LTF** — แท่ง impulse ที่ออกจากโซนต้องมี FVG บน LTF (ตั้งเป็นเงื่อนไขบังคับได้) · LTF volume delta ตามทิศได้ ★ · Leg-out ≥ 1× ความสูงโซน · คะแนนขั้นต่ำ 4 ★ (สูงสุด 12)
- **Fib คัดโซน (MAIN v2)** — ลาก Fib จากปลายไส้ sweep (1) ไปยอดขา MSS (0) · ยอดขาเลื่อนตาม high/low ใหม่จนกว่าจะเข้า Entry · ค่าเริ่มต้น = ตัวกรองที่ **0.618**: โซน OB / Demand-Supply ต้องอยู่ต่ำกว่า 0.618 (Bull) / สูงกว่า (Bear) · ตอน MSS ยังไม่ถึง → `รอ Fib` จนขาวิ่งต่อแล้วโซนผ่าน · ราคาแตะโซนก่อนผ่าน = ยกเลิก · เลือก `แตะ` / `ทั้งโซนเลย` · โหมด `คะแนน ★` หรือ Off ได้
- **สถิติเฉพาะที่เข้าจริง** — setup ที่ยังไม่ถึง Entry ใช้ได้จนกว่าจะปิดเลยไส้ sweep / มี setup ใหม่ฝั่งเดียวกันมาแทน / เก่าเกิน 150 แท่ง (ตั้งเวลาจำกัดได้ · ไม่ได้เข้า = ไม่นับ SL) · TP/SL ตั้งตอนราคาเข้า Entry จริงเท่านั้น (TP = liquidity ที่ยังไม่ถูกกวาด ณ ตอนเข้า · ไม่มีเป้า = ยกเลิก) · ลำดับ Entry → SL/TP เช็คด้วยแท่ง LTF · ไม่มี market entry
- **SL = Fixed** (ค่าเริ่มต้น) — ตั้งระยะ SL เองจาก Entry (Points / Price / ATR, ค่าเริ่มต้น 300 points)
- **SL = Zone** — SL เลยขอบนอกของโซนเข้า (OB/Supply/Demand) + buffer · ไม่มีโซน → ปลายไส้ sweep · ป้ายบอกระยะเป็น points
- **SL สูงสุด** — ถ้าระยะ Entry→SL เกินค่าที่ตั้ง (Points / Price / ATR) = ไม่เข้าออเดอร์ ลบ setup ทิ้ง (0 = ปิด)
- **MSS** — ระดับ = swing low ที่ส่งราคาขึ้นไป sweep (Bear) / swing high ที่ส่งราคาลงไป sweep (Bull): จุดต่ำ/สูงสุดหลัง swing ฝั่งตรงข้ามล่าสุด (ขาสั้นกว่า 1 ATR ย้อนไป swing ก่อนหน้า) · ตัดสินที่แท่งแรกที่ราคาไปถึงเลยระดับ: ปิดเนื้อเทียนเลย = MSS, ทะลุแค่ไส้ = ยกเลิก setup แล้วรอ sweep ใหม่ · sweep ใหม่ที่ไม่เลยไส้เดิมไม่ทับ SFP ที่รออยู่
- **เก็บ setup ที่จบแล้ว** (ค่าเริ่มต้นเปิด, ฝั่งละ 5) — ไม่ลบทิ้ง แต่ทำให้จางและแปะผล: `TP ✔ +2.4R` · `SL ✖ -1R` · `Miss · ไม่ถึง Entry` · `✖ ยกเลิก · ปิดเลยไส้` · `หมดอายุ` · setup จบเมื่อโดน SL ด้วย (หลัง Entry fill)
