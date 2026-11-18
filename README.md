# TradingView Indicators

| Indicator | ไฟล์ | คำอธิบาย |
|---|---|---|
| ⭐ **SFP Pro X — MAIN v2** (เวอร์ชันหลักล่าสุด) | [indicators/sfp/main/sfp-pro-x-main-v2.pine](indicators/sfp/main/sfp-pro-x-main-v2.pine) | MSS = swing ที่ส่งราคาไป sweep · ปิดเนื้อเทียนเท่านั้น (ไส้ = ยกเลิก) · OB/Demand-Supply กรองด้วย LTF · TP = liquidity ตั้งตอนเข้า · ไม่เข้า = ไม่นับ SL · sweep เฉพาะ swing ≤ 10 แท่ง · [รายละเอียด](indicators/sfp/README.md) |
| SFP Pro X — MAIN v1 (เวอร์ชันเดิม) | [indicators/sfp/main/sfp-pro-x-main-v1.pine](indicators/sfp/main/sfp-pro-x-main-v1.pine) | ตัวหลักที่ล็อกไว้ ไม่แก้ไข · Sweep → MSS → OB + A/V, SBR/RBS, QM · [รายละเอียด](indicators/sfp/README.md) |
| SFP Pro X — Dev (ตัวทดลองเพิ่มเงื่อนไข) | [indicators/sfp/sfp-pro-sweep-mss.pine](indicators/sfp/sfp-pro-sweep-mss.pine) | ตัวทดลอง: + Demand/Supply และตัวกรองโซน (Fresh, Displacement, P/D, ขนาด, คะแนน ★) · [รายละเอียด](indicators/sfp/README.md) |
