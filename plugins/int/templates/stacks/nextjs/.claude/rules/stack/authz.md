---
paths:
  - "src/domains/**/application/**"
  - "src/domains/identity/**"
  - "src/app/(admin)/**"
---
# Authorization (ที่มา: Profile: Next.js → Authorization (RBAC))

- Better Auth ใช้ยืนยันตัวตนเท่านั้น สิทธิ์อยู่ในโดเมน identity
- ตรวจสิทธิ์ใน use-case เสมอ ด้วย can(actor, "<domain>:<action>") → ไม่มีสิทธิ์คืน err("FORBIDDEN")
- ห้ามตรวจ role ตรงๆ เช่น actor.roles.includes("admin")
- default deny: permission ที่ไม่ได้ระบุถือว่าไม่มีสิทธิ์
- ตาราง role → permission อยู่ที่ src/domains/identity/domain/permissions.ts ไฟล์เดียว ฐานข้อมูลเก็บแค่ user ↔ role (และ unitId ถ้ามี)
- permission ใหม่ต้องเพิ่มใน permissions.ts และระบุใน plan ทุกครั้ง
- ทุกระบบต้องมี role admin ส่วน role อื่นกำหนดตามระบบ
- สิทธิ์ระดับเจ้าของข้อมูล (เช่น แก้ได้เฉพาะร่างของตัวเอง) เป็น policy ใน domain/policies/ ตรวจหลัง can()
- UI ใช้ can() ซ่อนปุ่มได้ layout ของ (admin)/ กันการเข้าหน้าได้ แต่ไม่แทนการตรวจใน use-case
- ทุก use-case ที่มี permission ต้องมี test ทั้งกรณีมีสิทธิ์และไม่มีสิทธิ์ (ได้ FORBIDDEN)

Review: แก้ permission, การตรวจสิทธิ์ หรือไฟล์ใน identity = เรียก security-reviewer ตอน review
บังคับด้วย: test ใน pnpm test (ยังไม่มี check ที่ตรวจว่าทุก use-case เรียก can())
