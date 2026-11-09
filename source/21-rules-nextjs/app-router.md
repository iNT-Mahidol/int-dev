---
paths:
  - "src/app/**"
---
# App Router (ที่มา: Profile: Next.js → โครงสร้างโฟลเดอร์)

- src/app/ มีไฟล์ route เท่านั้น: page, layout, loading, error, not-found, template, default, route, sitemap, robots, icon
- ห้าม _components/ หรือ utils ใน src/app/ ย้าย component ไปตามกฎใน components.md
- route group: (public) ไม่ต้อง login, (app) ผู้ใช้ที่ login, (admin) ผู้ดูแลระบบ, (dev) เครื่องมือ dev เท่านั้น
- api/ ใช้เฉพาะเมื่อ Server Action ทำไม่ได้
- page เรียกข้อมูลผ่าน index.ts ของโดเมน หรือ composition/queries ห้ามมี business logic

บังคับด้วย: pnpm check:app-purity
