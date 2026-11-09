---
paths:
  - "src/domains/**/infrastructure/**"
  - "src/composition/queries/**"
  - "prisma/**"
---
# Data และ Prisma (ที่มา: Profile: Next.js → Prisma relation, Single write gateway, Result และ error)

- Prisma ปรากฏได้เฉพาะ infrastructure/db-functions/ และ composition/queries/ (อ่านอย่างเดียว)
- schema: schema.prisma มีแค่ datasource + generator, _shared.prisma สำหรับ enum ข้ามโดเมน, 1 ไฟล์ต่อ 1 โดเมน ชื่อตรงกับโดเมน
- relation ข้ามโดเมนชี้ขึ้นต้นน้ำทางเดียว ทุกโดเมนอ้าง identity ได้ แต่ identity ไม่อ้างใคร
- single write gateway: 1 mutation มีฟังก์ชันเขียนทางเดียว ห้ามเขียนข้ามมัน (เช่น postStockMovement())
- entity เดียวใช้ได้ทุกทีม ห้ามแตกตารางตามทีม
- ค่าคำนวณและรายงานอยู่ใน view หรือ snapshot ไม่ปนกับตารางดิบ
- db-functions แปลง error ที่คาดได้: P2002 → CONFLICT, P2025 → NOT_FOUND ที่เหลือ throw ต่อ
- ห้ามแก้ src/generated/prisma/ (สร้างใหม่ด้วย postinstall)
- ห้ามรัน prisma migrate reset, migrate deploy หรือ db push เอง

Review: แก้ไฟล์ในโฟลเดอร์นี้ = เรียก database-reviewer ตอน review
บังคับด้วย: pnpm check:prisma-boundary (ทิศ relation และ single write gateway ยังไม่มี check)
