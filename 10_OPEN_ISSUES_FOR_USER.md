# 10 — Open Issues for User Decision
Version: 1.0 | Date: 2026-11-12 | Source: review of 00–09 spec files

วิธีใช้: แต่ละข้อมี "ค่าที่แนะนำ" อยู่แล้ว
- ถ้าเห็นด้วย ให้ใส่ `[x] ตามที่แนะนำ`
- ถ้าไม่เห็นด้วย ให้เขียนคำตอบของคุณในช่อง `คำตอบ:`

เมื่อตอบครบแล้ว ผมจะอัปเดต `00_MASTER.md` และ `09_DECISIONS_AND_CHANGELOG.md` ให้ตรงกัน แล้วเริ่มขั้นตอนตาม Master §11

---

## A. ไฟล์ในสเปกที่ขัดกันเอง

### A1. มีไฟล์ Module 06 สองไฟล์
- ปัญหา: `06_CONFLUENCE_ALERTS_UI.md` ยังเขียนว่า "Disable IDM until its definition is approved" ซึ่งล้าสมัยแล้ว และนิยามโปรไฟล์ Balanced ไม่ตรงกับ `06_ENTRY_CONFLUENCE_ALERTS.md`
- ค่าที่แนะนำ: ใช้ `06_ENTRY_CONFLUENCE_ALERTS.md` เป็นหลัก และยกเลิกหรือลบ `06_CONFLUENCE_ALERTS_UI.md`
- [x ] ตามที่แนะนำ
- คำตอบ:

### A2. มีไฟล์ 09 สองไฟล์
- ปัญหา: `09_DECISIONS_TO_CONFIRM.md` ข้อ 9 เสนอชุด TF ตายตัว (H4/H1/M15/M5/M1) ซึ่งขัดกับ Master §4 ที่ห้าม hard-code TF ส่วนข้อ 1, 5, 12 ได้รับการอนุมัติไปแล้ว
- ค่าที่แนะนำ: ใช้ `09_DECISIONS_AND_CHANGELOG.md` เป็นหลัก ส่วนข้อ 2, 3, 4, 6, 8 ของไฟล์เก่าให้ใช้ค่าเริ่มต้นตาม Master §5 แล้วยกเลิกไฟล์เก่า
- [x ] ตามที่แนะนำ
- คำตอบ:

### A3. SFP กับ Liquidity Sweep ใช้เงื่อนไขเดียวกัน
- ปัญหา: ทั้งสองแบบคือไส้ทะลุระดับแล้วปิดกลับ ถ้าไม่จัดการ แท่งเดียวกันจะเกิดสองเหตุการณ์ที่ไม่เชื่อมกัน
- ค่าที่แนะนำ: ให้ SFP เป็นประเภทย่อยของ sweep (ใช้ event ID เดียวกันแต่ติดป้าย SFP เพิ่ม) ส่วน alert ยังแยก Sweep กับ SFP ตามสเปก
- [ ] ตามที่แนะนำ
- คำตอบ:Sfp คือในกรณีที่มีแท่งขึ้นไปสวีปไฮแล้วกลับมาทะลุโลของไฮนั้นด้วยเนื้อเทียนถ้าทะลุด้วยไส้จะไม่นับเป็นSfpแต่ในกรณีที่ทะลุด้วยไส้แล้วแท่งถัดจากนั้นเป็นแท่งที่มีวอลุ่มรุนแรงก็จะนับเป็นSfp

---

## B. Logic ที่ยังกำกวม (กระทบผลลัพธ์โดยตรง)

### B1. MSS ต้องทะลุระดับไหน
- ปัญหา: 02 §7 เขียนแค่ว่า "the relevant internal/major structure level"
- ค่าที่แนะนำ: Bullish MSS = แท่งปิดทะลุ **internal swing high ล่าสุดที่ยืนยันแล้วก่อน IDM sweep** ตาม break mode ที่เลือก ส่วน Bearish ทำกลับด้าน และมี input ให้เลือก `Internal / Major` โดยค่าเริ่มต้นคือ Internal
- [ x] ตามที่แนะนำ
- คำตอบ:

### B2. Displacement กับ MSS ต้องเป็นแท่งเดียวกันหรือไม่
- ค่าที่แนะนำ: เพิ่ม input "Displacement window N bars" ค่าเริ่มต้น N = 0 ซึ่งหมายถึงแท่งที่ทะลุ MSS ต้องเป็นแท่ง displacement เอง
- [ x] ตามที่แนะนำ
- คำตอบ:

### B3. นิยาม "Protected major swing"
- ค่าที่แนะนำ: Protected low (ขาขึ้น) = major swing low ที่ยืนยันแล้ว ซึ่งเป็นจุดเริ่มต้นของขาที่ทำ bullish BOS/CHoCH ล่าสุด ถ้ายังไม่มี break ให้ใช้ major swing low ล่าสุดที่ยืนยันแล้ว ส่วน Protected high ทำกลับด้าน
- เพิ่มเติม: ตัวเลือก "dealing range model" ใน Master §7 ข้อ 1 ยังไม่มีนิยาม จึงเสนอให้ตัดออกในเวอร์ชันแรก
- [x ] ตามที่แนะนำ
- คำตอบ:

### B4. ขั้น `CONTEXT_FOUND` เกิดขึ้นเมื่อไร
- ปัญหา: Master §8 มีสถานะนี้ แต่ไม่มีนิยาม
- ค่าที่แนะนำ: เกิดเมื่อมี protected major swing ที่ใช้ได้ฝั่งนั้น และ HTF bias ไม่ขัดตามโปรไฟล์ (ถ้าปิด HTF ให้ข้ามเงื่อนไขนี้)
- [ x] ตามที่แนะนำ
- คำตอบ:

### B5. ขั้น ZONE_WAIT ต้อง retest โซนไหน
- ค่าที่แนะนำ: เฉพาะ OB/FVG ที่เกิดจากขา displacement ของ MSS นั้น ซึ่งผูกกันด้วย source break ID ไม่ใช่โซนใดก็ได้ที่ยัง active
- [ x] ตามที่แนะนำ
- คำตอบ:

### B6. การทะลุครั้งแรกตอน bias ยังเป็น Neutral
- ปัญหา: 01 ข้อ 5 กำหนดให้ "Define first break from neutral explicitly"
- ค่าที่แนะนำ: ให้นับเป็น BOS แล้วตั้ง bias ตามทิศที่ทะลุ
- [ ] ตามที่แนะนำ
- คำตอบ:ถ้าทะสุด้วยใส้เทียนให้นับเป็นบอสหลอกถ้าทะลุด้วยเนื้อเทียนให้นับเป็นบอสจริง

### B7. ขอบเขต Rejection Block (ค้างจาก 09 ข้อ 1)
- ปัญหา: คำว่า "rejection-side wick-to-body area" ตีความได้สองแบบ
  - (a) เฉพาะไส้ฝั่ง rejection: Bearish = `[max(open,close), high]` / Bullish = `[low, min(open,close)]`
  - (b) ไส้รวมตัวแท่ง: Bearish = `[min(open,close), high]` / Bullish = `[low, max(open,close)]`
- ค่าที่แนะนำ: (a) เป็นค่าเริ่มต้น โดย input ยังเลือก Wick / Body / Full ได้ และ invalidation = ปิดเลยปลายไส้ + buffer
- [x ] ตามที่แนะนำ
- คำตอบ:

### B8. Entry trigger และนิยาม "Rejection Candle" (ค้างจาก 09 ข้อ 2)
- ค่าที่แนะนำ: ค่าเริ่มต้น = `Rejection Candle`
  - Bullish Rejection Candle = แท่งที่ low แตะโซน แล้วปิดเหนือ proximal ของโซน และปิดในครึ่งบนของ range
  - Bearish ทำกลับด้าน
  - ตัวเลือกอื่น (Internal MSS / SFP / Any enabled) ยังเลือกได้
- [ x] ตามที่แนะนำ
- คำตอบ:

### B9. หนึ่ง setup ให้ entry ได้กี่ครั้ง
- ค่าที่แนะนำ: 1 ครั้ง แล้วรีเซ็ตเป็น NONE (reason = `ENTRY_DONE`)
- [x ] ตามที่แนะนำ
- คำตอบ:

### B10. โปรไฟล์ Strict ยอมให้ HTF เป็น Neutral หรือไม่
- ค่าที่แนะนำ: Strict ต้องตรงทิศ (neutral ถือว่าไม่ผ่าน) ส่วน Balanced ยอมให้เป็น neutral ได้
- [ x] ตามที่แนะนำ
- คำตอบ:

### B11. ยืนยันการ reclaim ของ IDM (ค้างจาก 09 ข้อ 3)
- ค่าที่แนะนำ: ค่าเริ่มต้นคือต้องปิดกลับบนแท่ง sweep เอง ส่วน "within N bars" เป็น input ที่ปิดไว้เป็นค่าเริ่มต้น (ตรงกับ 02 §5)
- [x ] ตามที่แนะนำ
- คำตอบ:

---

## C. ขอบเขตด้านเทคนิคของ Pine

### C1. Timeframe แยกรายโมดูล 8 ตัว
- ปัญหา: การรันตรรกะที่มีสถานะ (pivot / zone / state machine) บน TF อื่นผ่าน `request.security()` ทำได้แต่หนักมาก และ TF ที่ต่ำกว่ากราฟใช้กับตรรกะโครงสร้างไม่ได้จริง
- ค่าที่แนะนำ (เฟส 1):
  - ทุกโมดูลรองรับ `Chart TF` และ TF ที่ **สูงกว่าหรือเท่ากับ** กราฟ
  - ถ้าเลือก TF ต่ำกว่ากราฟ ให้ปิดโมดูลนั้นและเตือนบน dashboard (สเปก §4 อนุญาตไว้)
  - การรองรับ LTF ผ่าน `request.security_lower_tf()` เลื่อนไปเฟส 2
- [x ] ตามที่แนะนำ
- คำตอบ:

### C2. PDH/PDL/PWH/PWL บนกราฟ TF สูง
- ค่าที่แนะนำ: ถ้า chart TF ≥ D ให้ปิด PDH/PDL อัตโนมัติ ถ้า ≥ W ให้ปิด PWH/PWL และแจ้งบน dashboard
- [ x] ตามที่แนะนำ
- คำตอบ:

---

## D. ค่าเริ่มต้นที่สเปกยังไม่ได้ระบุ (ทุกค่าเป็น input ที่ปรับได้)

| ค่า | ค่าเริ่มต้นที่แนะนำ | คำตอบ |
|---|---|---|
| Break buffer | 0 | |
| Sweep buffer | 0 | |
| OB/FVG invalidation buffer | 0 | |
| Mitigation threshold | Touch | |
| Zone max age | 300 แท่ง | |
| IDM candidate max bars | 100 แท่ง | |
| Liquidity pool max age | 500 แท่ง | |
| SMT timestamp tolerance | 2 แท่ง | |
| Max boxes / lines / labels | 500 / 500 / 500 (เพดานของ Pine) | |

- [ x] ใช้ค่าตามที่แนะนำทั้งหมดส่วนsmt ให้จับคู่กับคู่ที่คิดว่าเหมาะสมได้เลย

---

## E. ข้อมูลที่ขาด

### E1. สัญลักษณ์ที่ใช้เทียบ SMT (ค้างจาก 09 ข้อ 5)
- ต้องระบุ: สัญลักษณ์และ exchange/broker เช่น `OANDA:XAGUSD`
- เพิ่มเติม: ถ้าใช้คู่ที่สวนทางกัน (เช่น DXY กับทอง) ต้องเพิ่ม input "Inverse correlation" ซึ่งสเปกยังไม่มี
- สัญลักษณ์สำหรับ XAUUSD:
- สัญลักษณ์สำหรับ Forex (ระบุทีละคู่ หรือเลือกเองใน input):
- เพิ่ม input Inverse correlation: [ ] ใช่ [ ] ไม่

### E2. ชุด Timeframe เริ่มต้น (ค้างจาก 09 ข้อ 4)
- ค่าที่แนะนำ: ไม่ hard-code โดยค่าเริ่มต้นทุกตัว = `Chart TF` ยกเว้น HTF Context = `Off` และ Setup TF = `Off`
- [ x] ตามที่แนะนำ
- คำตอบ:
