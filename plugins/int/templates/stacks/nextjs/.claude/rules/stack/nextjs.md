# Stack: Next.js (ที่มา: Profile: Next.js)

สถาปัตยกรรม Domain-oriented Modular Monolith with per-domain Clean Architecture ใช้โครงสร้างเต็มทุกระบบ ไม่มีระดับ Lite
กฎใน .claude/rules/stack/ ชนะ .claude/rules/ecc/ เมื่อขัดกัน

## Stack (ห้ามเปลี่ยนโดยไม่มี ADR)
- Next.js App Router, React 19, TypeScript strict; Next.js 16+ ใช้ proxy.ts ไม่ใช่ middleware.ts
- Prisma 7, schema แยกไฟล์ตามโดเมนใน prisma/schema/
- Better Auth (ห้าม NextAuth): production ใช้ ADFS OAuth, local ใช้ DEV_AUTH_ENABLED + DEV_AUTH_ALLOW_EMAILS
- config อยู่ที่ src/domains/identity/infrastructure/auth/better-auth.ts, route อยู่ที่ src/app/api/auth/[...all]/route.ts
- shadcn/ui + Tailwind v4, @tanstack/react-table v8, Recharts ผ่าน shadcn charts, Zustand, Lexical
- Zod สำหรับ validation
- STORAGE_DRIVER = localfs | s3 | seaweedfs
- pnpm เท่านั้น (ห้าม npm/yarn), Node 20+

## ลำดับ phase ใน plan
domain → use-case → db-functions → route → UI (ลบ phase ที่ไม่มีงาน)

## Commands
- Check: pnpm check:all (ห้ามปิดหรือข้ามตัวใด)
- Test: pnpm test, coverage 80% ขึ้นไป
- Build: pnpm build

## ยังไม่ได้ตัดสิน: ถาม Product Owner ก่อน ห้ามเลือกเอง
env schema, cache, naming, timezone/พ.ศ., logging/PDPA, git workflow, deployment/migration, lint/format
