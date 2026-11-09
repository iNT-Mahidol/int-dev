---
description: ตรวจว่าโปรเจกต์ยังตรงมาตรฐาน INT แล้วรายงานเป็นตารางพร้อมวิธีแก้ ไม่แก้ไฟล์ใดๆ
disable-model-invocation: true
allowed-tools:
  - Read
  - Grep
  - Glob
  - Bash(git rev-parse *)
  - Bash(jq *)
  - Bash(diff *)
  - Bash(ls *)
  - Bash(test *)
---

# /int:doctor

**ห้ามแก้ไฟล์ใดๆ** ห้ามใช้ Write หรือ Edit ห้ามรันคำสั่งที่เปลี่ยนสถานะ และห้ามอ่าน `.env` รันทีละคำสั่ง ไม่ใช้ pipe หรือ `&&`

รัน `git rev-parse --show-toplevel` ถ้าไม่สำเร็จให้แจ้งว่า "ต้องรันใน git repo" แล้วหยุด ทุก path ด้านล่างนับจากรากของ repo

อ่าน `AGENTS.md` เอาค่า `int-dev:`, `stack:` และ `design:` จากส่วนบนของไฟล์ ถ้าไม่มี AGENTS.md ให้ถือว่าไม่ทราบค่า และแสดงข้อที่ต้องใช้ค่านั้นเป็น ⚠️

## รายการตรวจ

### 1. ไฟล์บังคับ

`CLAUDE.md`, `AGENTS.md`, `lefthook.yml`, `.claude/settings.json`, `.claude/rules/int.md`, `.claude/design/DESIGN.md`

แต่ละไฟล์ที่ขาดเป็น ❌ วิธีแก้: รัน `/int:init`

### 2. เวอร์ชัน

เทียบ `int-dev:` กับ `version` ใน `${CLAUDE_PLUGIN_ROOT}/.claude-plugin/plugin.json` แบบ semver

| กรณี | ผล | วิธีแก้ |
| --- | --- | --- |
| เท่ากัน | ✅ | |
| ใน AGENTS.md เก่ากว่า | ⚠️ | รัน `/int:init` ซ้ำ |
| ใน AGENTS.md ใหม่กว่า | ⚠️ | อัปเดต plugin ด้วย `claude plugin update int@int-dev` |
| ไม่มี หรือยังเป็น `{{VERSION}}` | ❌ | รัน `/int:init` |

### 3. ECC และ superpowers

อ่าน `.claude/settings.json` และ `${CLAUDE_CONFIG_DIR:-$HOME/.claude}/plugins/installed_plugins.json`

| ตรวจ | ไม่ผ่าน | วิธีแก้ |
| --- | --- | --- |
| `enabledPlugins["ecc@ecc"]` เป็น `true` | ❌ | ตั้งค่าใน `.claude/settings.json` |
| มี entry ของ `ecc@ecc` ใน `installed_plugins.json` | ❌ | `claude plugin marketplace add affaan-m/everything-claude-code` แล้ว `claude plugin install ecc@ecc` |
| `enabledPlugins["superpowers@claude-plugins-official"]` เป็น `false` | ❌ | ตั้งค่าใน `.claude/settings.json` |
| `env.ECC_HOOK_PROFILE` เป็น `standard` | ❌ | ตั้งค่าใน `.claude/settings.json` |
| มี `.claude/rules/ecc/common/` | ⚠️ | ติดตั้ง ECC แล้วรัน `/int:init` ซ้ำ |

### 4. permissions.deny

```bash
jq -n --slurpfile tpl "${CLAUDE_PLUGIN_ROOT}/templates/base/.claude/settings.json" --slurpfile cur .claude/settings.json '$tpl[0].permissions.deny - ($cur[0].permissions.deny // [])'
```

ผลเป็น `[]` คือ ✅ ถ้ามีรายการเป็น ❌ และแสดงรายการที่ขาดทั้งหมด วิธีแก้: เพิ่มรายการเหล่านั้นใน `permissions.deny` ถ้า `jq` error เป็น ❌ พร้อมข้อความ error (ไฟล์หายให้ระบุว่าดูข้อ 1)

รายการ `Bash(pnpm deploy:prod *)` เปลี่ยนชื่อได้ตามคำสั่ง deploy จริงของระบบ ถ้าขาดแต่รายการนี้และมีรายการ deny ที่ขึ้นต้น `Bash(` และมีคำว่า `deploy` ให้เป็น ⚠️ และให้ผู้ใช้ยืนยันว่าเป็นคำสั่งจริง

### 5. lefthook

รัน `git rev-parse --git-path hooks/pre-commit` และ `git rev-parse --git-path hooks/pre-push` แล้วอ่านทั้งสองไฟล์ ทั้งคู่มีคำว่า `lefthook` คือ ✅ ถ้าไฟล์ใดไม่มีหรือไม่มีคำนี้เป็น ❌ พร้อมชื่อ hook วิธีแก้: `pnpm add -D lefthook` แล้ว `pnpm install` หรือ `pnpm exec lefthook install`

### 6. design-quality

เมื่อ `design: int` เท่านั้น: ถ้ามี `.claude/rules/ecc/web/design-quality.md` เป็น ❌ วิธีแก้: ลบไฟล์นี้ เพราะขัดกับ Corporate Identity กรณี `design: client` แสดง ✅ พร้อมหมายเหตุว่าไม่เกี่ยว

### 7. stack rules

เมื่อ `stack: nextjs`: เทียบทีละไฟล์ใน `${CLAUDE_PLUGIN_ROOT}/templates/stacks/nextjs/.claude/rules/stack/` ทั้ง 8 ไฟล์

```bash
diff -u "${CLAUDE_PLUGIN_ROOT}/templates/stacks/nextjs/.claude/rules/stack/<ไฟล์>" ".claude/rules/stack/<ไฟล์>"
```

ไฟล์ขาดหรือถูกแก้เป็น ⚠️ แสดง diff ไม่เกิน 10 บรรทัดต่อไฟล์ วิธีแก้: รัน `/int:init` ซ้ำแล้วเลือกทับ กรณี `stack: none` แสดง ✅ พร้อมหมายเหตุว่าไม่เกี่ยว

### 8. plan ที่อนุมัติแล้ว

หา `.claude/plans/*.plan.md` แล้วอ่านบรรทัดที่ขึ้นต้น `สถานะ:` ของแต่ละไฟล์ แยกช่องด้วย `·`

เมื่อสถานะเป็น `approved` หรือ `อนุมัติ` ช่อง `อนุมัติโดย:` และ `ผู้ขอยืนยัน:` ต้องมีค่าจริง ไม่ว่าง และไม่ใช่ข้อความ template เช่น `<PO>` หรือ `<ชื่อ>` ถ้าขาดเป็น ❌ พร้อมชื่อไฟล์และช่องที่ขาด วิธีแก้: กรอกช่องนั้นก่อนเริ่มงานต่อ

ไม่มี plan หรือไม่มี plan ที่อนุมัติแล้วแสดง ✅

## รายงาน

แสดงตารางเดียว เรียงตามข้อ 1 ถึง 8 แล้วปิดท้ายด้วยจำนวน ✅ / ⚠️ / ❌

```markdown
| รายการ | ผล | รายละเอียดและวิธีแก้ |
| --- | --- | --- |
```

ต่อด้วย diff จากข้อ 7 ถ้ามี
