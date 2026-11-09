---
paths:
  - "src/domains/**"
  - "src/composition/**"
  - "src/shared/**"
---
# Domains (ที่มา: Profile: Next.js → กฎกำกับสถาปัตยกรรม)

## โครงของโดเมน
- src/domains/<domain>/: index.ts (ทางเข้าเดียว), domain/, application/, infrastructure/, ui/<aggregate>/
- domain/types.ts บังคับ; <domain>-machine.ts เมื่อ entity มีสถานะมากกว่า 2; policies/ เมื่อมีการคำนวณ
- ห้ามสร้างโฟลเดอร์ว่างไว้เผื่อ

## ทิศทาง dependency
- ui → application → domain และ infrastructure → domain ไม่มีอะไรออกจาก domain/
- domain/ ต้อง pure: ห้าม import framework, Prisma หรือ I/O
- domains/ ห้าม import composition/
- shared/ ห้าม import domains/
- identity ห้าม import โดเมนอื่น
- กราฟ dependency ระหว่างโดเมนห้ามมีวงวน

## ข้ามโดเมน
- อ่าน: ผ่าน @/domains/<x> (index.ts) เท่านั้น
- เขียน: ผ่าน use-case ของโดเมนเจ้าของเท่านั้น ห้ามเข้า infrastructure/ ของโดเมนอื่น
- แชร์ type ด้วย export type คู่กับ import 'server-only'
- composition/ ไม่มีตารางของตัวเอง: queries/ อ่านอย่างเดียว, flows/ เขียนผ่าน use-case ของเจ้าของ

## โดเมนใหม่
ต้องผ่านครบ 3 ข้อ: เป็นเจ้าของ aggregate root ผู้เดียว, มี lifecycle ของตัวเอง, ลบทิ้งแล้วส่วนที่เหลือยัง compile ผ่าน
ไม่ผ่าน → อยู่ในโดเมนเดิม (identity บังคับมีทุกระบบ ไม่ต้องทดสอบ)
การเพิ่มโดเมนต้องมี ADR

บังคับด้วย: pnpm check:domain-boundary, pnpm check:shared-purity (Promotion Test ยังไม่มี check)
