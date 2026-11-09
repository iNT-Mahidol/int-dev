# int-dev

Claude Code plugin marketplace ของ INT มี plugin เดียวชื่อ `int` ซึ่งทำแค่ 2 อย่าง คือวางไฟล์มาตรฐาน INT ลงในโปรเจกต์ และตรวจว่าโปรเจกต์ยังตรงมาตรฐาน

plugin นี้ไม่ใช่ fork ของ ECC และไม่เพิ่ม agent, hook หรือ command ที่ซ้ำกับ ECC แหล่งข้อมูลของมาตรฐานอยู่ใน `source/` จุดที่ข้อมูลยังขัดกันอยู่ใน [OPEN-QUESTIONS.md](OPEN-QUESTIONS.md)

## ติดตั้ง

```bash
claude plugin marketplace add iNT-Mahidol/int-dev
claude plugin install int@int-dev
```

หลังรัน `/int:init` และ commit `.claude/settings.json` แล้ว คนที่ clone repo จะได้รับการเสนอ marketplace `int-dev` และ `ecc` โดยไม่ต้องเพิ่มเอง

## /int:init

```text
/int:init stack=nextjs design=int
```

| อาร์กิวเมนต์ | ค่า | ความหมาย |
| --- | --- | --- |
| `stack` | `nextjs` หรือ `none` | `nextjs` เพิ่ม stack rules และ ECC rules ของ typescript, react, web |
| `design` | `int` หรือ `client` | `int` คัด DESIGN ของ INT, `client` สร้าง `DESIGN.md` เปล่าให้กรอกแบรนด์ลูกค้า |

ถ้าไม่ระบุ skill จะถาม และจะถามคำสั่ง Check, Test, Build เสมอ

ไฟล์ที่มีอยู่แล้วและต่างจากเดิมจะแสดง diff แล้วถามทีละไฟล์ ไฟล์ที่เหมือนเดิมทุกไบต์จะไม่แตะ skill นี้ไม่ commit, push หรือรัน install และไม่อ่าน `.env`

หลังรันเสร็จ ให้ทำตามลำดับ:

1. ติดตั้ง [gitleaks](https://github.com/gitleaks/gitleaks)
2. `pnpm add -D lefthook --allow-build=lefthook` (ต้องทำก่อน `pnpm install` เพราะ `prepare` เรียก `lefthook` และ pnpm บล็อก build script ถ้าไม่ระบุ flag นี้)
3. `pnpm install`
4. `/ecc:ck init`
5. `claude plugin list` ตรวจว่า `ecc@ecc` เปิดอยู่
6. ปรับ `pnpm deploy:prod` ใน `.claude/settings.json` ให้ตรงคำสั่งจริงของระบบ
7. `/int:doctor`

## /int:doctor

```text
/int:doctor
```

แสดงตาราง ✅ / ⚠️ / ❌ พร้อมวิธีแก้ ตรวจไฟล์บังคับ, เวอร์ชัน, การเปิดปิด plugin, `permissions.deny`, lefthook, `design-quality.md`, stack rules และช่องอนุมัติของ plan

`allowed-tools` ของ skill นี้แค่ข้ามการขอสิทธิ์ของคำสั่งอ่าน ไม่ได้จำกัดเครื่องมืออื่น การไม่แก้ไฟล์จึงบังคับด้วยคำสั่งใน skill เท่านั้น

## ออกเวอร์ชันใหม่

1. แก้ `version` ใน `plugins/int/.claude-plugin/plugin.json` (ไม่ระบุใน `marketplace.json` เพื่อให้มีที่เดียว)
2. เพิ่มรายการใน [CHANGELOG.md](CHANGELOG.md)
3. ตรวจ:

   ```bash
   claude plugin validate --strict plugins/int
   claude plugin validate --strict .
   diff -r source/21-rules-nextjs plugins/int/templates/stacks/nextjs/.claude/rules/stack
   ```

4. รวมเข้า `main` แล้วติด tag ด้วย `claude plugin tag plugins/int`
5. ผู้ใช้อัปเดตด้วย `claude plugin update int@int-dev` แล้วรัน `/int:init` ซ้ำในแต่ละโปรเจกต์ `/int:doctor` จะเตือนโปรเจกต์ที่ยังเก่า

เมื่อ `source/` เปลี่ยน ต้องคัดไฟล์ไปยัง `plugins/int/templates/` และ `plugins/int/design-profiles/` ใหม่ด้วย
