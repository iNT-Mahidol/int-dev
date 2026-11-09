# Open Questions

จุดที่ข้อมูลขัดกันหรือไม่พอ พร้อมทางที่เลือกไว้ก่อน ให้ Product Owner ปิดทีละข้อ

| # | เรื่อง | ที่มา | ทางที่เลือกไว้ก่อน |
| --- | --- | --- | --- |
| 1 | `source/10` บรรทัด 5 บอกว่า INT ไม่เขียน skill ของตัวเอง แต่ plugin นี้มี 2 skill | `source/10` หลักการ | ทำตามคำขอสร้าง plugin เพราะ 2 skill นี้ไม่ซ้ำกับ ECC ควรแก้ถ้อยคำใน source |
| 2 | คำขอให้ CLAUDE.md มี 6 หัวข้อ แต่ source ให้ Commands, Verifying, Workflow, Things Claude gets wrong อยู่ใน AGENTS.md | `source/10` ส่วนบังคับ | Product Owner เลือกตาม source: CLAUDE.md มี `@AGENTS.md` + Subagents + Session |
| 3 | lefthook ตัวอย่างใช้ `pnpm test:coverage` แต่ AGENTS.md ใช้ `pnpm test` ทั้งที่บอกว่าเป็นคำสั่งชุดเดียวกัน | `source/10` Quality gate, `21-rules-nextjs/testing.md` | ใช้ `{{TEST_CMD}}` ค่าเริ่มต้น `pnpm test` เพราะ `testing.md` ระบุว่า coverage threshold อยู่ใน c8 |
| 4 | `actions.md` อ้าง "Golden reference ใน AGENTS.md" แต่ source ไม่มีเนื้อหานี้ | `21-rules-nextjs/actions.md` | ไม่แต่งเพิ่ม ต้องเติมใน AGENTS.md ของแต่ละระบบ หรือแก้ rule |
| 5 | `DESIGN.md` อ้าง `tokens.css`, `components/src/*` และ `bundle.js` ซึ่งไม่อยู่ใน source | `30-design-int/DESIGN.md` | คัดเฉพาะ DESIGN.md และ design-tokens.json ตามคำขอ ต้องหาแหล่งของไฟล์เหล่านี้ |
| 6 | `source/10` อ้าง "แท็บ Profile" และ "Stack Profile" ซึ่งไม่อยู่ใน source | `source/10` | ใช้ `21-rules-nextjs/nextjs.md` แทน |
| 7 | marketplace ของ ECC ใน source คือ `affaan-m/everything-claude-code` แต่เครื่องที่สร้าง plugin ลงจาก `affaan-m/ECC.git` | `source/10` การติดตั้ง | คงตาม source เพราะ repo นี้ยังเข้าถึงได้ |
| 8 | deny กันแค่ `Read(.env)` ไม่กัน Edit/Write ทั้งที่กฎบอกห้ามแตะ `.env` | `source/10` การติดตั้ง | คง template ตาม source เพื่อไม่แต่งกฎเพิ่ม แนะนำให้เพิ่ม `Edit` และ `Write` ใน source |
| 9 | `permissions` และ lefthook ผูกกับ pnpm แต่ `stack=none` อาจไม่ใช้ pnpm | `source/10` | `settings.json` คงเดิมทุก key ส่วน lefthook ตัดงาน `lint` เมื่อ `stack=none` |
| 10 | ขนาด S: ตารางบอก "ไม่มี plan" แต่หัว plan มีช่องขนาด S และระยะ Plan ให้ข้าม plan เฉพาะงานที่อธิบาย diff ได้ในประโยคเดียว | `source/10` | `rules/int.md` ใช้เงื่อนไขที่แคบกว่า: ข้าม plan ได้เฉพาะงานที่อธิบาย diff ได้ในประโยคเดียว |
| 11 | "ระบุขนาดไว้ในบรรทัด Complexity ของ plan" แต่ template ของ plan ไม่มีบรรทัด Complexity | `source/10` | ใช้ช่อง `ขนาด` ในบรรทัดหัว plan |
| 12 | สถานะ plan ใน source คือ `approved` แต่คำขอ doctor เขียนว่า "อนุมัติ" | `source/10`, คำขอ | doctor ตรวจทั้งสองคำ |
| 13 | คำขอให้ gitignore มีไฟล์ของ ck แต่ ck เก็บข้อมูลที่ `~/.claude/ck/` ไม่มีไฟล์ใน repo | `source/10` ck, skill `ck` ของ ECC 2.2.3 | ไม่เพิ่ม entry ของ ck |
| 14 | ขั้นตอนหลัง init ในคำขอให้รัน `pnpm install` แต่ `lefthook` ไม่อยู่ใน dependencies ทำให้ `prepare` fail | คำขอ, `source/10` | init แจ้งให้รัน `pnpm add -D lefthook --allow-build=lefthook` และติดตั้ง `gitleaks` ก่อน `pnpm install` (ทดสอบกับ pnpm 12.10 ถ้าไม่มี flag จะ exit 1 และไม่ติดตั้ง hook) |
| 15 | grep ข้อ 4 "ทั้ง repo" เจอชื่อคำสั่ง ECC แบบสั้นใน `source/10:43` ซึ่งห้ามแก้ และในไฟล์ PROMPT | คำขอ | grep โดยยกเว้น `source/` และ `int-dev-source-2026-11-09/` |
| 16 | ชื่อ deploy ใน deny คือ `pnpm deploy:prod` แต่ source ให้เปลี่ยนตามระบบ | `source/10` | init เตือนในสรุปให้ปรับ `.claude/settings.json` ตามคำสั่งจริง |
| 17 | `nextjs.md` มีหัวข้อ "ยังไม่ได้ตัดสิน" (env schema, cache, naming, timezone/พ.ศ., logging/PDPA, git workflow, deployment/migration, lint/format) | `21-rules-nextjs/nextjs.md` | ไม่ตัดสินแทน คัด verbatim |
