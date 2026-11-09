---
paths:
  - "src/**/*.test.ts"
  - "src/**/*.test.tsx"
  - "src/test-utils/**"
---
# Testing (ที่มา: Profile: Next.js → Testing)

- Node built-in test runner + sinon + c8 (E2E ใช้ Playwright)
- วางไฟล์ test ข้างไฟล์ที่ทดสอบ: create-project.ts คู่กับ create-project.test.ts
- บังคับ test สำหรับ logic ใน domain/ และ use-case ที่มี invariant
- coverage ขั้นต่ำ 80% ทั้ง repo รวม actions/ และ ui/
- fixture ที่ใช้ร่วมกันอยู่ใน src/test-utils/
- test ที่ fail ห้ามแก้หรือลบเพื่อให้ผ่าน ให้แก้โค้ด

บังคับด้วย: pnpm test (coverage threshold ใน c8)
