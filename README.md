# TradingView Indicators

| Indicator | ไฟล์ | คำอธิบาย |
|---|---|---|
| ⭐ **SFP Pro X — MAIN v2** (ไฟล์หลัก · ห้ามแก้) | [indicators/sfp/main/sfp-pro-x-main-v2.pine](indicators/sfp/main/sfp-pro-x-main-v2.pine) | MSS = swing ที่ส่งราคาไป sweep · ปิดเนื้อเทียนเท่านั้น (ไส้ = ยกเลิก) · OB/Demand-Supply กรองด้วย LTF · TP = liquidity ตั้งตอนเข้า · ไม่เข้า = ไม่นับ SL · sweep เฉพาะ swing ≤ 25 แท่ง · แท่ง sweep ต้องเว้นห่าง swing ≥ 2 แท่ง · MSS ตาม swing ที่ถูก sweep · SBR = body แท่ง sweep · ไส้เลยแท่ง sweep ก่อน MSS = ยกเลิก · Fib 0.618 คัดโซน (ปรับได้) · [รายละเอียด](indicators/sfp/README.md) |
| SFP Pro X — DEV (ไฟล์สำรองสำหรับแก้) | [indicators/sfp/sfp-pro-sweep-mss.pine](indicators/sfp/sfp-pro-sweep-mss.pine) | สำเนาเดียวกับ MAIN v2 ล่าสุด (2026-11-19) · ใช้แก้/ทดลองเงื่อนไขใหม่ก่อนย้ายเข้าไฟล์หลัก · [รายละเอียด](indicators/sfp/README.md) |
| **SMC CB** (ใหม่) | [indicators/smc-cb/smc-cb.pine](indicators/smc-cb/smc-cb.pine) | Bias D1 → BOS/CHoCH ปิดเนื้อเทียน ไล่ H4 → M15 → Confirm TF (Body = TP Low · Wick = TP POI) → IDM → OB/Supply M5 คัดด้วย Fib 0.618 / 0.786 · แดชบอร์ดแบบ SFP Pro X · [รายละเอียด](indicators/smc-cb/README.md) |
