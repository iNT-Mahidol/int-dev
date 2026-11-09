# Prompt: สร้าง plugin int-dev จาก Claude Project Standard

> วิธีใช้: แตก zip นี้ไว้ในโฟลเดอร์ repo ใหม่ชื่อ `int-dev` (ให้มีโฟลเดอร์ `source/` อยู่ที่ root) เปิด Claude Code ใน WSL ที่โฟลเดอร์นั้น แล้ววางข้อความในกรอบด้านล่างทั้งหมด

---

```text
สร้าง Claude Code plugin marketplace ชื่อ "int-dev" ใน repo นี้ ให้มี plugin เดียวชื่อ "int" (version 0.1.0)
หน้าที่ของ plugin มีแค่ 2 อย่าง: (1) วางไฟล์มาตรฐาน INT ลงในโปรเจกต์ (2) ตรวจว่าโปรเจกต์ยังตรงมาตรฐาน
plugin นี้ไม่ใช่ fork ของ ECC และไม่เพิ่ม agent, hook หรือ command ที่ซ้ำกับ ECC

## แหล่งข้อมูล (source of truth เดียว)
อ่านทุกไฟล์ใน source/ ให้ครบก่อนเริ่ม:
- source/10-claude-project-standard.md   มาตรฐานกระบวนการ (หลักการ, 6 ระยะ, gate, plan template, lefthook, settings.json, CLAUDE.md/AGENTS.md/rules/int.md, subagent)
- source/21-rules-nextjs/*.md            stack rules ของ Next.js 8 ไฟล์ (ใช้ verbatim)
- source/30-design-int/                   DESIGN.md + design-tokens.json ของ INT
ห้ามแต่งกฎเพิ่มเอง ถ้าข้อมูลขัดกันหรือไม่พอ ให้จดลง OPEN-QUESTIONS.md แล้วเลือกทางที่เข้มกว่าไว้ก่อน

## ข้อเท็จจริงของ Claude Code ที่ต้องเคารพ
- plugin แจก rules หรือ CLAUDE.md โดยตรงไม่ได้ จึงต้องใช้วิธี "skill คัดลอก template ลงโปรเจกต์"
- plugin settings.json รองรับแค่ key agent และ subagentStatusLine ห้ามใส่ permissions ใน plugin
- ใช้ ${CLAUDE_PLUGIN_ROOT} อ้าง path ใน plugin
- ไม่ใช้ dependencies ข้าม marketplace ให้ template settings.json ประกาศทั้ง 2 marketplace เอง
- ก่อนเขียน manifest ให้เปิดเอกสาร plugin ล่าสุดของ Claude Code (plugins reference) ยืนยัน schema ก่อน

## โครงสร้างที่ต้องได้
int-dev/
├── .claude-plugin/marketplace.json          # marketplace "int-dev" → plugin "int" source ./plugins/int
├── plugins/int/
│   ├── .claude-plugin/plugin.json           # name "int", version 0.1.0, description ภาษาไทย
│   ├── skills/
│   │   ├── init/SKILL.md                    # /int:init
│   │   └── doctor/SKILL.md                  # /int:doctor
│   ├── templates/
│   │   ├── base/                            # ทุก stack
│   │   │   ├── CLAUDE.md
│   │   │   ├── AGENTS.md
│   │   │   ├── lefthook.yml
│   │   │   ├── gitignore.append
│   │   │   └── .claude/
│   │   │       ├── settings.json
│   │   │       ├── rules/int.md
│   │   │       ├── plans/.gitkeep
│   │   │       └── prds/.gitkeep
│   │   ├── docs/adr/.gitkeep, docs/lessons/.gitkeep
│   │   └── stacks/nextjs/.claude/rules/stack/*.md   # คัดจาก source/21-rules-nextjs/ ทั้ง 8 ไฟล์ verbatim
│   └── design-profiles/int/{DESIGN.md, design-tokens.json}
├── source/                                  # เก็บไว้ตามเดิม ห้ามแก้
├── README.md                                # ภาษาไทย: ติดตั้ง, ใช้ init/doctor, วิธีออกเวอร์ชันใหม่
├── CHANGELOG.md
└── OPEN-QUESTIONS.md

## เนื้อหา template (ดึงจาก source/10 ตามหัวข้อ)
- .claude/settings.json: ตามหัวข้อ "การติดตั้ง" ครบทุก key (extraKnownMarketplaces ecc, enabledPlugins ecc=true และ superpowers=false, env ECC_HOOK_PROFILE=standard, permissions allow/deny ทั้งหมด) และเพิ่ม marketplace int-dev + enabledPlugins "int@int-dev": true
- CLAUDE.md: ตาม "ส่วนบังคับใน CLAUDE.md" (Commands, Verifying your work, Workflow, Things Claude gets wrong, Subagents, Session) ใช้ placeholder {{CHECK_CMD}} {{TEST_CMD}} {{BUILD_CMD}}
- AGENTS.md: ตามหัวข้อ CLAUDE.md/AGENTS.md มีบรรทัด "int-dev: {{VERSION}}" และ "stack: {{STACK}}" "design: {{DESIGN}}" ไว้บนสุด
- .claude/rules/int.md: ไม่มี frontmatter paths (โหลดทุก session) ไม่เกินราว 80 บรรทัด ครอบคลุม gate 1/2/production, header plan (สถานะ · ขนาด S|M|L · อนุมัติโดย · ผู้ขอยืนยัน · Task), ชื่อไฟล์ plan <YYYY-MM-DD>-<slug>.plan.md, ขนาด S/M/L, conditional reviewers, ผลจาก subagent ไม่เกิน 10 ข้อพร้อม file:line, Corporate Identity, coverage 80%, ห้าม push main/force push, ห้ามแตะ .env และ secret, production deploy มนุษย์เป็นคนกด
- lefthook.yml: ตามตัวอย่างใน "Quality gate บนเครื่อง" ใช้ placeholder คำสั่งเดียวกับ CLAUDE.md ตรวจ syntax ของ gitleaks ให้ตรงกับเวอร์ชันปัจจุบัน
- gitignore.append: รายการที่ต้องเพิ่มต่อท้าย .gitignore (เช่น .claude/settings.local.json, ไฟล์ของ ck ที่ไม่ควร commit) ไม่ ignore plans/prds

## skill /int:init
frontmatter: description ภาษาไทย, disable-model-invocation: true, argument-hint "[stack=nextjs|none] [design=int|client]"
ขั้นตอน:
1. ตรวจว่าอยู่ใน git repo ถ้าไม่ใช่ให้หยุด
2. รับ stack และ design จาก argument ถ้าไม่ระบุให้ถาม (design=client หมายถึงโปรเจกต์ลูกค้า ไม่คัด DESIGN ของ INT แต่สร้าง .claude/design/DESIGN.md เปล่าพร้อมหัวข้อให้กรอกแบรนด์ลูกค้า)
3. ถามคำสั่ง Check/Test/Build (nextjs ให้ค่าเริ่มต้นจาก source/21-rules-nextjs/nextjs.md)
4. คัด templates/base และ templates/stacks/<stack> ลงโปรเจกต์ ไฟล์ที่มีอยู่แล้วให้แสดง diff และถามก่อนทับทุกไฟล์ ห้ามทับเงียบ
5. คัด rules ของ ECC จาก plugin ECC ที่ติดตั้งอยู่ (หา path จาก ~/.claude/plugins) ลง .claude/rules/ecc/ เฉพาะ common และภาษาที่ stack ใช้ (nextjs = typescript + react/web) ยกเว้น web/design-quality.md ทุกกรณี ถ้าหา ECC ไม่เจอให้บอกวิธีติดตั้งแล้วข้ามขั้นนี้
6. design=int คัด design-profiles/int ลง .claude/design/
7. แทน placeholder ทั้งหมด, append gitignore.append โดยไม่ซ้ำบรรทัดเดิม, เพิ่ม "prepare": "lefthook install" ใน package.json ถ้ามี
8. สรุปเป็นตาราง: ไฟล์ที่สร้าง / ทับ / ข้าม และบอกขั้นตอนที่ผู้ใช้ต้องทำเอง: pnpm install, /ecc:ck init, claude plugin list, แล้วรัน /int:doctor
ห้าม: commit, push, รัน install เอง, อ่านหรือแก้ .env

## skill /int:doctor
frontmatter: description ภาษาไทย, disable-model-invocation: true, allowed-tools เฉพาะอ่าน (Read, Grep, Glob, Bash แบบอ่านอย่างเดียว)
ตรวจแล้วแสดงตาราง ✅ / ⚠️ / ❌ พร้อมวิธีแก้ทีละข้อ:
- ไฟล์บังคับครบ (CLAUDE.md, AGENTS.md, lefthook.yml, .claude/settings.json, .claude/rules/int.md, .claude/design/DESIGN.md)
- เวอร์ชันใน AGENTS.md เทียบกับ plugin ปัจจุบัน (เก่ากว่า = ⚠️ แนะนำรัน /int:init ซ้ำ)
- ecc เปิด, superpowers ปิด, ECC_HOOK_PROFILE=standard
- permissions.deny มีครบตาม template (บอกรายการที่ขาด)
- lefthook ติดตั้งแล้ว (.git/hooks/pre-commit มาจาก lefthook)
- ไม่มี .claude/rules/ecc/web/design-quality.md ในโปรเจกต์ design=int
- stack rules ตรงกับ template (ไฟล์ขาดหรือถูกแก้ = ⚠️ แสดง diff สั้นๆ)
- .claude/plans/ ไม่มี plan สถานะ "อนุมัติ" ที่ขาดช่อง อนุมัติโดย/ผู้ขอยืนยัน
doctor ห้ามแก้ไฟล์ใดๆ

## การตรวจงาน (ต้องผ่านทั้งหมดก่อนบอกว่าเสร็จ)
1. claude plugin validate --strict ทั้ง plugins/int และ marketplace
2. สร้าง repo ทดสอบใน /tmp (git init + package.json เปล่า) ติดตั้ง marketplace แบบ local แล้วรัน /int:init stack=nextjs design=int ตามด้วย /int:doctor ผลต้องไม่มี ❌ นอกจากรายการที่ต้องติดตั้งเอง (lefthook, ECC)
3. รัน /int:init ซ้ำในโปรเจกต์เดิม ต้องถามก่อนทับทุกไฟล์
4. grep ทั้ง repo ต้องไม่เจอชื่อคำสั่ง ECC แบบสั้น (เช่น "/plan " หรือ "/code-review") ใช้ /ecc: เต็มเสมอ
5. diff templates/stacks/nextjs/.claude/rules/stack/ กับ source/21-rules-nextjs/ ต้องเหมือนกันทุกไบต์

## วิธีทำงาน
- ทำเป็น plan ก่อน (/ecc:plan) แล้วรอฉันยืนยัน
- commit ทีละขั้น: scaffold → templates → init → doctor → README/CHANGELOG ห้าม push
- ตอบและเขียนเอกสารเป็นภาษาไทย ศัพท์เทคนิคคงภาษาอังกฤษ
```

---

## หมายเหตุสำหรับหม่อน

- ใน doc มีหลักการว่า "INT ไม่เขียน skill ของตัวเอง" plugin นี้มีแค่ skill วางไฟล์และตรวจไฟล์ ซึ่ง ECC ไม่มี ถ้าจะใช้จริงควรแก้ถ้อยคำหลักการใน doc ให้ตรงกันด้วย
- `source/21-rules-nextjs/` ร่างจากเนื้อหาแท็บ ".claude rules" ที่เราเขียนกันใน doc ถ้าหม่อนแก้แท็บนั้นภายหลัง ให้ copy มาทับไฟล์ใน source/ ก่อนรัน prompt
