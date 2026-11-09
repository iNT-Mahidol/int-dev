---
paths:
  - "src/domains/**/application/**"
---
# Use-case, Action, Result, Validation (ที่มา: Profile: Next.js)

## ลำดับงาน
- action: requireActor() → เรียก use-case → revalidate ห้ามตรวจสิทธิ์หรือ validate เอง
- use-case: validate → authorize → ตรวจ invariant → เรียก db-functions → คืน Result<T>
- "use server" อยู่ได้เฉพาะ application/actions/ ใช้กับ mutation เท่านั้น เพราะทุก export เป็น endpoint สาธารณะ
- use-case เป็นฟังก์ชันธรรมดา signature: (actor: Actor, input: unknown) => Promise<Result<T>>

## Result และ error (src/shared/result.ts)
- คืน ok(value) หรือ err(code, message?) ห้าม throw สำหรับ error ที่คาดไว้
- รหัสมีแค่: VALIDATION, UNAUTHENTICATED, FORBIDDEN, NOT_FOUND, CONFLICT, INVARIANT, UNEXPECTED (เพิ่มต้องมี ADR)
- INVARIANT ต้องเขียน message ภาษาไทยเฉพาะเรื่องเอง รหัสอื่นใช้ข้อความเริ่มต้นจาก src/shared/errors/messages.ts ได้
- message ห้ามมี stack, SQL หรือข้อความจาก Prisma
- ข้อมูลที่ผู้ใช้ไม่มีสิทธิ์เห็น คืน NOT_FOUND ไม่ใช่ FORBIDDEN
- error ที่ไม่ได้คาด throw ต่อให้ error.tsx หรือ wrapper ของ route handler
- route handler ใน api/ ใช้ toResponse(result) แปลงเป็น HTTP status

## Validation
- ใช้ Zod เท่านั้น schema อยู่ที่ application/validation/<use-case>.ts export schema + type (z.infer)
- ไฟล์ validation ห้าม import server-only, Prisma หรือโค้ดฝั่ง server เพราะฟอร์มใช้ schema เดียวกัน
- use-case เรียก safeParse เป็นบรรทัดแรกเสมอ แปลงผลด้วย validationError(zodError)
- ข้อความ error ภาษาไทยเขียนไว้ใน schema
- schema ตรวจรูปแบบเท่านั้น กฎธุรกิจอยู่ใน domain/policies/ และคืน INVARIANT

ตัวอย่างต้นแบบ: Golden reference ใน AGENTS.md
บังคับด้วย: pnpm check:server-action (ลำดับงานและรหัส error ยังไม่มี check ใช้ type ของ Result คุม)
