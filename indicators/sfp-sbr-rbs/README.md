# SFP SBR/RBS — Sweep + MSS → Limit Entry ที่ SBR / RBS

- ไฟล์หลัก (ห้ามแก้): [`main/sfp-sbr-rbs-main.pine`](./main/sfp-sbr-rbs-main.pine)
- ไฟล์สำรองสำหรับแก้: [`sfp-sbr-rbs-dev.pine`](./sfp-sbr-rbs-dev.pine) (สำเนาเดียวกัน · ชื่อบนกราฟ `SFP SBR/RBS DEV`)
- แตกมาจาก SFP Pro X (ใช้ Liquidity / Sweep / MSS / Dashboard ชุดเดียวกัน) แต่เข้าออเดอร์ที่ SBR / RBS อย่างเดียว

## วิธีใช้
1. เปิดไฟล์ `.pine` → กด **Raw** → คัดลอกทั้งหมด
2. TradingView → Pine Editor → วางโค้ด → **Add to chart**

## ลำดับเงื่อนไข (ตัวอย่างฝั่ง Bearish · Bull กลับด้าน)
1. **Liquidity** — swing high (5/2, อายุ ≤ 25 แท่ง), EQH, PDH, PWH, Asia High (HTF ปิดไว้)
2. **Sweep** — ไส้เลยระดับแล้วปิดกลับ · แท่ง sweep ต้องห่างแท่ง swing ≥ 2 แท่ง
3. **รอ MSS** (≤ 20 แท่ง) — ไส้เลยปลายไส้ sweep / แตะ MSS แค่ไส้ / ปิดเหนือไส้ sweep = ยกเลิก
4. **MSS** — ปิดเนื้อเทียนทะลุ low ของ swing ที่ส่งราคาขึ้นไป sweep · แท่ง MSS ต้องสร้าง FVG
5. **Entry = SBR** — ก้น body ของแท่ง swing low ที่ถูก MSS ทุบ → Sell Limit ทันที (Bull: RBS = หัว body ของ swing high → Buy Limit)
   - SBR ต้องอยู่ระหว่างไส้ sweep กับราคาปิด ไม่งั้น = `✖ ไม่มีจุดเข้า SBR/RBS`
6. **SL** — 1000 points (XAUUSD 1 point = 0.01 → $10 ที่ 0.01 lot = 10% ของทุน $100) · ถ้าปลายไส้ sweep ห่าง Entry **ไม่ถึง** 1000 points → SL ที่ปลายไส้ sweep (+ buffer ได้)
7. **TP** — liquidity ฝั่งตรงข้ามที่ยังไม่ถูกกวาด (Key liquidity ก่อน · ข้ามเป้าที่ RR < 1) ตั้งตอนราคาเข้า Entry จริง · ไม่มีเป้า = ยกเลิก
8. **ไม่ได้เข้า** — ราคาปิดเลยไส้ sweep ก่อนถึง Entry / มี setup ใหม่ฝั่งเดียวกัน / เก่าเกิน 150 แท่ง = ไม่นับ W/L

## Input สำคัญ (หมวด 7 Trade & Risk)
`SL ตั้งจาก` (Points / USD) · `SL (points)` 1000 · `1 point = ราคาเท่าไร` 0.01 · `Lot` 0.01 · `Contract size` 100 · `ทุน` 100 · `SL ไม่ถึงระยะที่ตั้ง → ใช้ปลายไส้ sweep` · `buffer เลยปลายไส้ (points)`

## Dashboard
Status · Entry (SBR/RBS · Buy/Sell Limit) · W/L · Net R และ $ (คิดจาก SL จริงแต่ละไม้) · Funnel · Risk / ออเดอร์
