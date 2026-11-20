# TradingView Indicators

| Indicator | ไฟล์ | คำอธิบาย |
|---|---|---|
| ⭐ **SFP Pro X — MAIN v2** (ไฟล์หลัก · อัปเดต 2026-11-20) | [indicators/sfp/main/sfp-pro-x-main-v2.pine](indicators/sfp/main/sfp-pro-x-main-v2.pine) | MSS = swing ที่ส่งราคาไป sweep · ปิดเนื้อเทียนเท่านั้น (ไส้ = ยกเลิก) · OB/Demand-Supply กรองด้วย LTF · TP = liquidity ตั้งตอนเข้า · ไม่เข้า = ไม่นับ SL · sweep เฉพาะ swing ≤ 25 แท่ง · แท่ง sweep ต้องเว้นห่าง swing ≥ 2 แท่ง · MSS ตาม swing ที่ถูก sweep · SBR/RBS = body แท่ง swing ที่ระดับ MSS · แท่ง MSS ต้องมี FVG · แตะ POI ก่อนถึง Fib แล้วทำ H/L ใหม่ = ยกเลิก · ไส้เลยแท่ง sweep ก่อน MSS = ยกเลิก · Fib 0.618 คัดโซน (ปรับได้) · [รายละเอียด](indicators/sfp/README.md) |
| SFP Pro X — DEV (ไฟล์สำรองสำหรับแก้) | [indicators/sfp/sfp-pro-sweep-mss.pine](indicators/sfp/sfp-pro-sweep-mss.pine) | สำเนาเดียวกับ MAIN v2 ล่าสุด (2026-11-20) · ใช้แก้/ทดลองเงื่อนไขใหม่ก่อนย้ายเข้าไฟล์หลัก · [รายละเอียด](indicators/sfp/README.md) |
| ⭐ **HTF CE0.5 — MAIN** (ไฟล์หลัก · 2026-11-20) | [indicators/htf-ce05/main/htf-ce05-main.pine](indicators/htf-ce05/main/htf-ce05-main.pine) | Hidden Liquidity (MSS → Key → hidden liQ → IDM) + Single Liquidity กับ CE 0.5 ของไส้ HTF · กล่อง Entry/SL/TP · เซ็ตอัพที่จบแล้วเป็นสีเทา · [รายละเอียด](indicators/htf-ce05/README.md) |
| HTF CE0.5 — DEV (ไฟล์สำรองสำหรับแก้) | [indicators/htf-ce05/htf-ce05-dev.pine](indicators/htf-ce05/htf-ce05-dev.pine) | สำเนาเดียวกับ MAIN (2026-11-20) · ใช้แก้/ทดลองก่อนย้ายเข้าไฟล์หลัก · [รายละเอียด](indicators/htf-ce05/README.md) |
