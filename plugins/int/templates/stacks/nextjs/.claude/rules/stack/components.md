---
paths:
  - "src/**/*.tsx"
---
# UI component (ที่มา: Profile: Next.js → ตำแหน่งของ UI component)

## ตำแหน่ง: นับจำนวนโดเมนที่ component import type หรือ DTO
- 0 → shared/ui/components/
- 1 → domains/<x>/ui/<aggregate>/ (จัดกลุ่มตาม aggregate ห้ามวางแบน)
- 2 ขึ้นไป → composition/ui/

## shadcn/ui
- shared/ui/primitives/ เป็นปลายทางของ shadcn CLI ห้ามแก้ไฟล์ที่ generate
- จะปรับธีมให้ใช้ CSS variables หรือห่อด้วย wrapper ใน shared/ui/components/

## หน้าตาและพฤติกรรม
- ใช้ token และ component ตาม .claude/design/DESIGN.md เท่านั้น ห้ามใส่สี ฟอนต์ หรือระยะตรงๆ
- ทุกหน้าที่แสดงข้อมูลมีสถานะ loading, empty และ error
- ฟอร์มแสดง fieldErrors ใต้ช่องที่ผิด และ message ด้านบนฟอร์ม ใช้ schema เดียวกับ use-case ตรวจฝั่ง client
- ตาราง: @tanstack/react-table v8, กราฟ: Recharts ผ่าน shadcn charts, rich text: Lexical ใน shared/rich-text/
- Zustand ใช้เมื่อ state ข้ามหลาย component จริงเท่านั้น

Review: หน้าจอใหม่หรือ component ที่มีการโต้ตอบ (ฟอร์ม, dialog, menu) = เรียก a11y-architect ตอน review
บังคับด้วย: pnpm check:design-tokens, pnpm check:datetime-format (ตำแหน่ง component ยังไม่มี check)
