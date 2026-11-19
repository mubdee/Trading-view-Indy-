# TradingView Indicators

| Indicator | ไฟล์ | คำอธิบาย |
|---|---|---|
| ⭐ **SFP Pro X — MAIN v2** (ไฟล์หลัก · ห้ามแก้) | [indicators/sfp/main/sfp-pro-x-main-v2.pine](indicators/sfp/main/sfp-pro-x-main-v2.pine) | MSS = swing ที่ส่งราคาไป sweep · ปิดเนื้อเทียนเท่านั้น (ไส้ = ยกเลิก) · OB/Demand-Supply กรองด้วย LTF · TP = liquidity ตั้งตอนเข้า · ไม่เข้า = ไม่นับ SL · sweep เฉพาะ swing ≤ 10 แท่ง · แท่ง sweep ต้องเว้นห่าง swing ≥ 1 แท่ง · Fib 0.618 คัดโซน (ปรับได้) · [รายละเอียด](indicators/sfp/README.md) |
| SFP Pro X — DEV (ไฟล์สำรองสำหรับแก้) | [indicators/sfp/sfp-pro-sweep-mss.pine](indicators/sfp/sfp-pro-sweep-mss.pine) | สำเนาของ MAIN v2 ล่าสุด (รวม Fib คัดโซน + เว้นแท่ง sweep) · ใช้แก้/ทดลองเงื่อนไขใหม่ก่อนย้ายเข้าไฟล์หลัก · [รายละเอียด](indicators/sfp/README.md) |
