# Subagent ของ ECC และ effort

model มาจาก frontmatter ของ ECC แต่ ECC ไม่ได้กำหนด effort ไว้ ระบุ `effort` ตามตารางทุกครั้งที่เรียก subagent คำสั่ง ECC ที่เรียก agent ให้เอง เช่น `/ecc:code-review` ให้ตั้ง `/effort` ของ session ให้ตรงตารางก่อนรัน

| ระยะ | Subagent | model (ECC) | effort | เรียกเมื่อ |
| --- | --- | --- | --- | --- |
| 1. Plan | `planner` | opus | high | `/ecc:plan` งานขนาด M และ L |
| 2. Design | `architect` | opus | high | มีการตัดสินใจเชิงโครงสร้าง ก่อนเขียน ADR |
| 2. Design | `a11y-architect` | sonnet | medium | มีหน้าจอใหม่หรือ component ที่มีการโต้ตอบ |
| 3. Build | `tdd-guide` | sonnet | medium | งานขนาด M และ L |
| 3. Build | `build-error-resolver` | sonnet | low | build หรือ type check พัง |
| 4. Test | `code-reviewer` | sonnet | medium | ทุกงาน M และ L ผ่าน `/ecc:code-review` |
| 4. Test | `security-reviewer` | sonnet | high | diff แตะ auth, API หรือข้อมูลส่วนบุคคล |
| 4. Test | `database-reviewer` | sonnet | high | diff แตะ `prisma/` หรือ `db-functions/` |
| 4. Test | `typescript-reviewer`, `react-reviewer` | sonnet | medium | diff แตะ `.ts` หรือ `.tsx` |
| 4. Test | `e2e-runner` | sonnet | medium | งานขนาด L |
| 5. Deploy | `doc-updater` | haiku | low | `/ecc:update-docs` |
| 6. Maintain | `refactor-cleaner` | sonnet | low | `/ecc:refactor-clean` |
| ทุกระยะ | Explore (มากับ Claude Code) | - | low | ค้น codebase |

- effort ของ session หลักตามขนาดงาน: S ใช้ low, M ใช้ medium, L ใช้ high
- ห้ามใช้ `xhigh` หรือ `max` เป็นค่าเริ่มต้น ยกขึ้นได้ทีละขั้นเมื่อรอบก่อนพลาด และบันทึกเหตุผลใน plan
- ห้ามเรียก subagent ที่ไม่อยู่ในตารางนี้ ถ้าจำเป็นให้เพิ่มเข้ามาตรฐานก่อน
