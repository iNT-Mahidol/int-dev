# กฎของ INT

ใช้ของ ECC ก่อนเสมอ และเรียกด้วยชื่อเต็มที่มี `ecc:` นำหน้า ไฟล์นี้รวมกฎที่ ECC ไม่รู้ และเป็นข้อตรวจตอน `/ecc:code-review`

## Gate
- Gate 1: ไม่มีโค้ดก่อน plan ผ่าน Gate 1 ผู้ขอยืนยัน Requirements แล้ว Product Owner อนุมัติและเป็นคน commit plan
- Gate 2: ไม่ push ก่อน Gate 2 git hook ตอน push ผ่าน และผู้พัฒนาอ่านผล review กับผลรันจริงแล้ว
- Production: Agent เตรียม release ได้ แต่คนเป็นผู้อนุมัติและกดเองทุกครั้ง
- Development: agent deploy ได้เอง Staging: deploy ได้เมื่อ check ทั้งหมดผ่าน
- ถ้า Product Owner เป็นผู้พัฒนาด้วย release ขึ้น production ต้องมีผู้อนุมัติอย่างน้อยหนึ่งคนที่ไม่ใช่ผู้พัฒนา
- ถ้า design ทำให้ Requirements เปลี่ยน ให้แก้ plan แล้วขอ Gate 1 ใหม่ ทำต่างจาก plan ต้องแก้ plan ใน commit เดียวกัน

## Plan
- ที่เก็บ: `.claude/plans/<YYYY-MM-DD>-<slug>.plan.md` 1 change = 1 ไฟล์
- บรรทัดหัว plan:

```markdown
สถานะ: draft | approved | done · ขนาด: S | M | L · อนุมัติโดย: <PO> <วันที่> · ผู้ขอยืนยัน: <ชื่อ> · Task: <ลิงก์>
```

- ข้าม plan ได้เฉพาะงานที่อธิบาย diff ได้ในประโยคเดียว

## ขนาดงาน
ตัดสินก่อนเริ่มทุกครั้ง ถ้าไม่แน่ใจให้ถาม อย่าเลือกขนาดใหญ่ไว้ก่อน

| ขนาด | ตัวอย่าง | ใช้อะไร |
| --- | --- | --- |
| S | แก้คำผิด, แก้ไฟล์เดียว, ปรับ style | session เดียว ไม่เรียก agent ผ่าน lefthook ก่อน commit |
| M | ฟีเจอร์เล็กในโดเมนเดียว | `/ecc:plan` → `tdd-workflow` → `/ecc:code-review` ครั้งเดียว |
| L | ฟีเจอร์ข้ามโดเมน, migration, auth, หน้าจอใหม่ | ครบทุกขั้น + reviewer เฉพาะทาง + `e2e-testing` |

## Reviewer เฉพาะทาง (เรียกเมื่อมีเหตุเท่านั้น)
- `security-reviewer`: diff แตะ auth, API หรือข้อมูลส่วนบุคคล
- `database-reviewer`: diff แตะ `prisma/` หรือ `db-functions/`
- `a11y-architect`: มีหน้าจอใหม่หรือ component ที่มีการโต้ตอบ
- กรณีอื่นใช้ `/ecc:code-review` ตัวเดียว

## ผลจาก subagent
- ตอบเป็นรายการปัญหา แต่ละข้อมี `file:line` ไม่เกิน 10 ข้อ
- ห้ามแปะโค้ด log หรือเนื้อไฟล์กลับมา
- เลือก subagent และ effort ตาม `.claude/rules/ecc-agents.md`

## ข้อตรวจตอน review
- ตรงกับ Requirements ของ plan, ไม่มีงานนอกขอบเขต, ไม่มี secret หรือข้อมูลส่วนบุคคลใน log
- Important คือสิ่งที่ทำให้ระบบพัง ข้อมูลรั่ว หรือผิดนโยบาย เรื่องชื่อและสไตล์เป็น Nit
- coverage 80% ขึ้นไป

## Corporate Identity
- ลำดับแหล่งอ้างอิง: INT Brand Guidelines → INT System Standard → `.claude/design/DESIGN.md` ถ้าขัดกัน ชั้นบนชนะ
- UI ใช้ token และ component ตาม `.claude/design/DESIGN.md` เท่านั้น ห้ามใส่ค่าสี ฟอนต์ หรือระยะตรงๆ และตรวจกับ Nielsen's 10 Heuristics
- ห้ามคัดลอก `rules/web/design-quality.md` ของ ECC ลงระบบของ INT
- pattern หน้าจอใหม่ที่ยังไม่มีในมาตรฐาน ทำ mockup ให้ Product Owner อนุมัติก่อน
- ระบบที่จะออกนอก Corporate Identity ต้องมี ADR บอกเหตุผล
- งานของลูกค้าภายนอกใช้ `DESIGN.md` ตามแบรนด์ลูกค้า ไม่ใช้ Corporate Identity ของ INT

## ห้าม
- ห้าม push เข้า main และห้าม force push
- ห้ามข้าม git hook ด้วย `--no-verify`
- ห้ามแตะ `.env` และ secret
- ห้าม deploy production เอง
