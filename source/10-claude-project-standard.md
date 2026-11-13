# Claude Project Standard

## หลักการ

มาตรฐานนี้กำหนดว่าทุกระบบของ INT พัฒนาด้วย Claude Code อย่างไร **SDLC ตาม AI-Native SDLC Playbook ของ Anthropic เป็นกระบวนการ** ที่บอกว่าต้องผ่านด่านไหน ส่งมอบอะไร และใครอนุมัติ ส่วน **ECC (Everything Claude Code) เป็นเครื่องมือทำงานแบบเต็มรูปแบบ** INT ไม่เขียน agent, skill หรือ hook ของตัวเอง เพราะไม่มีคนดูแลระยะยาว INT ดูแลแค่ไฟล์กติกาไม่กี่ไฟล์ในแต่ละ repo

1. **ทุกระยะจบด้วย artifact ที่ commit ลง git** ระยะถัดไปเริ่มจากการอ่าน artifact นั้น ห่วงโซ่ของ commit คือ audit trail ว่าใครขออะไร agent ทำอะไร และใครอนุมัติ
2. **คนยืนอยู่เหนือวงจร** คนเป็นผู้เริ่ม กำกับ และอนุมัติที่ gate ส่วนงานระหว่าง gate ให้ agent ทำ
3. **กฎที่ต้องเป็นจริงเสมอบังคับด้วยเครื่องมือ** git hook (lefthook) บังคับทุกคน permission ใน `settings.json` บังคับ Claude ส่วน `CLAUDE.md`, rule และ skill เป็นคำแนะนำ
4. **Agent ทำได้ถึงประตู production แต่ผ่านเองไม่ได้** การ release ขึ้น production ต้องมีคนอนุมัติและกดเองทุกครั้ง
5. **ใช้ของ ECC ก่อนเสมอ** ถ้า ECC มีของที่ทำงานนั้นแล้ว ห้ามเขียนของ INT ซ้ำ

มาตรฐานนี้ใช้กับทุกระบบของ INT คู่กับแท็บ Profile ของ stack ที่กำหนดโครงสร้างโค้ดและคำสั่งตรวจ

## วงจร 6 ระยะ

SDLC แบบ AI-native เป็นวงวน ไม่ใช่เส้นตรง ปัญหาที่พบในระยะ Maintain จะเริ่ม `/ecc:plan` รอบใหม่ แล้ววงจรเริ่มรอบถัดไปเอง

&#91;embedded content: วงจร SDLC · 6 ระยะ\]

| ระยะ | Artifact ที่ commit | ผู้อนุมัติที่ gate | ระยะถัดไปเริ่มเมื่อ |
| --- | --- | --- | --- |
| 1. Plan | plan ใน `.claude/plans/` (หัวข้อ Requirements) | ผู้ขอยืนยัน Requirements แล้ว Product Owner อนุมัติ (Gate 1) | Product Owner commit plan ที่อนุมัติแล้ว |
| 2. Design | ADR ใน `docs/adr/`, mockup (เฉพาะงานที่ต้องมี) | Product Owner | ADR และ mockup ได้รับอนุมัติ |
| 3. Build | test ที่ fail, โค้ด | ไม่มี gate ใหม่ ทำตาม plan ที่ผ่าน Gate 1 | test ผ่านทั้งหมด |
| 4. Test | ผล review และผลรันจริง | git hook ผ่าน แล้วผู้พัฒนายืนยัน (Gate 2) | push ได้ |
| 5. Deploy | release, เอกสารที่อัปเดต | ผู้รับผิดชอบ release อนุมัติและกดเอง | release สำเร็จ |
| 6. Maintain | `docs/lessons/`, plan รอบใหม่ | ผู้ดูแลระบบคัดกรอง | กลับไปที่ Plan |

**บทบาท:** หม่อนเป็น Product Owner ของทุกระบบ ผู้ขอระบบยืนยันว่าหัวข้อ Requirements ของ plan ตรงกับที่ต้องการก่อน Product Owner อนุมัติที่ Gate 1 เมื่อ Product Owner เป็นผู้พัฒนาด้วย ให้ git hook และ `/ecc:code-review` เป็นผู้ตรวจที่ไม่ได้เขียนงาน และให้ release ขึ้น production มีผู้อนุมัติอย่างน้อยหนึ่งคนที่ไม่ใช่ผู้พัฒนา

### ECC ในแต่ละระยะ

| ระยะ | ใช้จาก ECC |
| --- | --- |
| 1. Plan | `/ecc:plan`, งานใหญ่ `/ecc:plan-prd` แล้ว `/ecc:plan <prd>`, skill `intent-driven-development`, `search-first` |
| 2. Design | agent `architect`, skill `architecture-decision-records`, `design-system` (audit token), `accessibility` และ agent `a11y-architect` |
| 3. Build | skill `tdd-workflow`, `/ecc:build-fix`, `/ecc:ck` |
| 4. Test | `/ecc:code-review` (เรียก `security-reviewer`, `database-reviewer`, `typescript-reviewer`, `react-reviewer` ตามเงื่อนไข), skill `verification-loop`, `/ecc:test-coverage`, skill `e2e-testing`, `/ecc:security-scan` |
| 5. Deploy | `/ecc:update-docs`, `/ecc:pr` (เมื่อใช้ remote) |
| 6. Maintain | `/ecc:learn`, `/ecc:evolve`, `/ecc:refactor-clean` |

ใช้ชื่อเต็มที่มี `ecc:` นำหน้าเสมอ เพราะ `/plan` และ `/code-review` ชนกับคำสั่งของ Claude Code เอง ใน ECC 2.2.3 คำสั่ง `/tdd`, `/verify`, `/e2e` และ `/orchestrate` ถูกเลิกแล้ว ให้ใช้ skill ตามตารางแทน ห้ามเปิด legacy shim หลังติดตั้งให้พิมพ์ `/ecc:` ดูรายการจริง ถ้าชื่อใดไม่ตรง ให้แก้เอกสารนี้

## รายละเอียดแต่ละระยะ

### 1. Plan: requirement อยู่ใน plan

- ผู้ขอเล่าปัญหาด้วยภาษาของตัวเองให้ Claude ฟัง แล้วรัน `/ecc:plan "<ความต้องการ>"` planner ทวน requirement ประเมินความเสี่ยง แตก phase และเขียน Success Criteria แล้วหยุดรอยืนยัน
- requirement กำกวมหรือเสี่ยงสูง (เงิน ข้อมูลส่วนบุคคล migration) ใช้ skill `intent-driven-development` ก่อน
- งานใหญ่ที่ส่งมอบหลายรอบ ใช้ `/ecc:plan-prd` ได้ `.claude/prds/<name>.prd.md` แล้วรัน `/ecc:plan .claude/prds/<name>.prd.md` ECC จะบันทึก plan ลง `.claude/plans/` ให้เอง
- ซักถาม plan: อะไรอาจพัง ขั้นไหนเสี่ยงที่สุด และทางเลือกที่ไม่เลือก วนจนคนที่ไม่เคยเห็นบทสนทนาลงมือตามได้
- ผู้ขอยืนยัน Requirements แล้ว Product Owner อนุมัติ บันทึกเป็น `.claude/plans/<YYYY-MM-DD>-<slug>.plan.md` พร้อมบรรทัดสถานะ แล้ว Product Owner เป็นคน commit (Gate 1) ไม่มีโค้ดก่อน commit นี้
- งานที่อธิบาย diff ได้ในประโยคเดียว เช่น แก้คำผิด ข้าม plan ได้

```markdown
# Implementation Plan: <ชื่อเรื่อง>
สถานะ: draft | approved | done · ขนาด: S | M | L · อนุมัติโดย: <PO> <วันที่> · ผู้ขอยืนยัน: <ชื่อ> · Task: <ลิงก์>
## Overview
## Requirements
## Architecture Changes
## Implementation Steps
### Phase 1: <ตามลำดับใน Stack Profile>
## Testing Strategy
## Risks & Mitigations
## Success Criteria
- [ ] Check ผ่าน
- [ ] Test ผ่าน, coverage 80% ขึ้นไป
```

Phase ที่ไม่มีงานให้ลบทิ้ง ไม่ต้องเก็บหัวข้อว่างไว้

### 2. Design: ADR และ mockup

- ใช้ agent `architect` เมื่อมีการตัดสินใจเชิงโครงสร้าง แล้วบันทึกเป็น ADR ด้วย skill `architecture-decision-records` (`docs/adr/NNNN-*.md`, สถานะ proposed → accepted → superseded) ถ้า plan เปลี่ยนการตัดสินใจเดิม ให้เขียน ADR ใหม่ที่ supersede ฉบับเดิมใน commit เดียวกัน
- งานที่มีผู้ใช้: ใช้ ISO 9241-210 เป็นแกน เริ่มด้วย Jobs To Be Done และ Service Blueprint แล้วเขียนผลลงหัวข้อ Requirements
- งานหน้าจอ: ทำ mockup ตาม `.claude/design/DESIGN.md` ตรวจกับ Nielsen's 10 Heuristics แล้วให้ Product Owner ยืนยันก่อนเข้า Build
- จุดที่น่ากังวลหรือนโยบายขัดกัน ระบุในหัวข้อ Risks และปิดกับเจ้าของนโยบายก่อนลงมือ
- ถ้า design ทำให้ Requirements เปลี่ยน ให้แก้ plan แล้วขอ Gate 1 ใหม่

### 3. Build: test ก่อน แล้วทำตาม plan

- เริ่ม session ด้วย `/ecc:ck resume`
- skill `tdd-workflow`: test ที่ fail ต้อง commit ก่อน แล้วจึงเขียนโค้ดให้ผ่าน ห้ามแก้หรือลบ test เพื่อให้ผ่าน
- ลงมือทีละ phase ตามลำดับใน plan จบแต่ละ phase ให้ `/ecc:ck save` ถ้า build พังใช้ `/ecc:build-fix`
- ทำต่างจาก plan ต้องแก้ plan ใน commit เดียวกัน
- UI ใช้ token และ component ตาม `.claude/design/DESIGN.md` เท่านั้น

### 4. Test: ตรวจด้วยหลักฐาน ไม่ใช่คำยืนยัน

- คำสั่ง Check, Test, Build ต้องรันได้ด้วยคำสั่งเดียวและคืนค่า non-zero เมื่อพัง (กำหนดใน AGENTS.md และแท็บ Profile)
- ทุกข้อใน Success Criteria ต้องตรวจด้วยคำสั่งได้
- `/ecc:code-review` ใน context ใหม่ แก้ CRITICAL และ HIGH ทั้งหมด ถ้าแตะ auth, API หรือข้อมูลส่วนบุคคล เพิ่ม `/ecc:security-scan`
- skill `verification-loop` และ `/ecc:test-coverage` (80% ขึ้นไป) งานหน้าจอใช้ skill `e2e-testing` และเทียบ screenshot ทั้ง light และ dark กับ mockup
- Gate 2: git hook ตอน push ผ่าน และผู้พัฒนาอ่านผล review กับผลรันจริงก่อน push รายงานว่าเสร็จได้เมื่อแนบผลการรันจริงเท่านั้น

### 5. Deploy: คนกดเอง

- คำสั่ง deploy production ถูก deny ไว้ใน `settings.json` Claude เตรียม release ได้ แต่ผู้รับผิดชอบ release เป็นคนอนุมัติและรันเอง
- rollback ต้องเป็นคำสั่งเดียวที่ซ้อมบน staging แล้ว
- หลัง release ใช้ `/ecc:update-docs` ทำเอกสารให้ตรงกับโค้ด อัปเดต Decision Log หรือ ADR และเปลี่ยนสถานะ plan เป็น done

| สภาพแวดล้อม | Agent ทำได้ |
| --- | --- |
| Development | deploy ได้เอง |
| Staging | deploy ได้เมื่อ check ทั้งหมดผ่าน |
| Production | เตรียม release ได้ แต่คนเป็นผู้อนุมัติและกด |

### 6. Maintain: ปิดวงจร

- เหตุการณ์ (error, ticket ใน SC - IT Request Tracking) ให้ Claude วินิจฉัยแบบอ่านอย่างเดียว แล้วเริ่ม `/ecc:plan` รอบใหม่จากผลนั้น
- แก้ได้เฉพาะผ่าน plan ใหม่ หรือ runbook ที่อนุมัติไว้ล่วงหน้า เช่น rollback
- บันทึกบทเรียนใน `docs/lessons/` แล้วใช้ `/ecc:learn` สกัด pattern และ `/ecc:evolve` ยกระดับเป็น skill ถ้าเกิดจาก Claude ทำผิดซ้ำ ให้เพิ่มกฎใต้ `## Things Claude gets wrong` ใน AGENTS.md
- ทุกเดือนใช้ `/ecc:refactor-clean` ลบโค้ดที่ไม่ใช้

## Quality gate บนเครื่อง

รอบนี้ยังไม่ใช้ GitHub บังคับกติกา ตัวบังคับอยู่บนเครื่องของผู้พัฒนา 3 ชั้น

| ชั้น | ใช้อะไร | กันอะไร |
| --- | --- | --- |
| Git hook | `lefthook.yml` | ทุกคน ทั้งคนและ Claude: commit หรือ push ไม่ได้ถ้า check ไม่ผ่าน |
| Permission | `deny` ใน `.claude/settings.json` | Claude ข้าม hook, push เข้า main, force push และ deploy production เองไม่ได้ |
| ECC hook | `ECC_HOOK_PROFILE=standard` | Claude ตรวจ format และ type ก่อนจบงาน |

```yaml
# lefthook.yml: คำสั่งชุดเดียวกับใน AGENTS.md
pre-commit:
  parallel: true
  commands:
    lint:
      glob: "*.{ts,tsx,js,jsx,css}"
      run: pnpm exec eslint {staged_files}
    secrets:
      run: gitleaks ...   # สแกน secret ในไฟล์ที่ stage
pre-push:
  commands:
    no-main:
      run: test "$(git rev-parse --abbrev-ref HEAD)" != "main"
    check:
      run: pnpm check:all
    test:
      run: pnpm test:coverage
    build:
      run: pnpm build
```

ใส่ `"prepare": "lefthook install"` ใน `package.json` เพื่อให้ hook ติดตั้งเองตอน `pnpm install` ข้อจำกัดคือคนยังพิมพ์ `--no-verify` ข้ามได้ แต่ Claude ถูก deny ไว้ เมื่อทีมโตขึ้นค่อยเพิ่ม CI บน GitHub โดยใช้คำสั่งชุดเดียวกัน

## Corporate Identity

- ลำดับแหล่งอ้างอิง: INT Brand Guidelines → INT System Standard → `.claude/design/DESIGN.md` ของแต่ละ repo ถ้าขัดกัน ชั้นบนชนะ
- UI ของระบบ INT ใช้ token และ component ตาม `DESIGN.md` เท่านั้น ห้ามใส่ค่าสี ฟอนต์ หรือระยะตรงๆ ในโค้ด บังคับด้วย lint ใน `check:all`
- ห้ามคัดลอก `rules/web/design-quality.md` ของ ECC ลงระบบของ INT เพราะกฎนั้นดันให้ UI แต่ละชิ้นดูไม่ซ้ำ ซึ่งขัดกับ Corporate Identity
- pattern หน้าจอใหม่ที่ยังไม่มีในมาตรฐาน ทำ mockup ให้ Product Owner อนุมัติก่อน แล้วเพิ่มเข้า INT System Standard
- ระบบที่จะออกนอก Corporate Identity ต้องมี ADR บอกเหตุผล
- งานของลูกค้าภายนอกใช้ `DESIGN.md` ตามแบรนด์ลูกค้า ไม่ใช้ Corporate Identity ของ INT

## โครงสร้างโฟลเดอร์

✏️ INT ดูแล · 📦 คัดลอกจาก ECC · 🤖 ECC สร้างระหว่างทำงาน

```text
<repo>/
├── CLAUDE.md                 # ✏️ @AGENTS.md + เรื่องเฉพาะ Claude Code
├── AGENTS.md                 # ✏️ คำสั่ง Check / Test / Build (ชุดเดียวกับ lefthook), เวอร์ชัน ECC
├── lefthook.yml              # ✏️ quality gate บนเครื่อง
├── package.json              # "prepare": "lefthook install"
├── .mcp.json                 # MCP ที่ทีมใช้ร่วม (ถ้ามี)
├── .worktreeinclude          # ไฟล์ gitignore ที่ต้องคัดลอกไป worktree เช่น .env
├── .claude/
│   ├── settings.json         # ✏️ เปิด ECC, ปิด superpowers, deny, ECC_HOOK_PROFILE
│   ├── settings.local.json   # ค่าส่วนตัว (gitignore)
│   ├── rules/
│   │   ├── ecc/              # 📦 common, web, stack (ไม่เอา design-quality.md) ห้ามแก้
│   │   ├── int.md            # ✏️ gate, ที่เก็บ plan, Corporate Identity, ข้อตรวจตอน review
│   │   └── ecc-agents.md     # ✏️ subagent ของ ECC และ effort ในแต่ละระยะ
│   ├── design/DESIGN.md      # ✏️ Corporate Identity ของ repo นี้
│   ├── prds/                 # 🤖 /ecc:plan-prd
│   └── plans/                # 🤖 /ecc:plan: 1 change = 1 ไฟล์
└── docs/
    ├── adr/                  # การตัดสินใจที่อยู่ยาว
    └── lessons/              # บทเรียนหลังเกิดเหตุ
```

`.gitignore` ต้องมี `.claude/settings.local.json`, `.claude/worktrees/` และ `CLAUDE.local.md` ส่วน `.claude/plans/` และ `.claude/prds/` ต้อง commit เพราะเป็นหลักฐานของ gate

### เลือกที่วางให้ถูก

| ต้องการ | วางที่ | บังคับได้จริงไหม |
| --- | --- | --- |
| บริบทและคำสั่งที่ใช้ทุก session | `CLAUDE.md` / `AGENTS.md` | ไม่ (คำแนะนำ) |
| กฎของ INT และข้อตรวจตอน review | `.claude/rules/int.md` | ไม่ |
| subagent ของ ECC และ effort | `.claude/rules/ecc-agents.md` | ไม่ |
| ขั้นตอนการทำงาน | skill และ command ของ ECC | ไม่ |
| ห้าม Claude รันคำสั่งหรืออ่านไฟล์ | `settings.json` → `permissions.deny` | ใช่ (เฉพาะ Claude) |
| check ที่ต้องผ่านก่อน commit และ push | `lefthook.yml` | ใช่ (ทุกคน) |

ไฟล์ใน `~/.claude/` ของแต่ละคนเป็นของส่วนตัว ห้ามใส่กฎที่ทีมต้องใช้ร่วมไว้ที่นั่น

## การติดตั้ง

```json
{
  "extraKnownMarketplaces": {
    "ecc": { "source": { "source": "github", "repo": "affaan-m/everything-claude-code" } }
  },
  "enabledPlugins": {
    "ecc@ecc": true,
    "superpowers@claude-plugins-official": false
  },
  "env": { "ECC_HOOK_PROFILE": "standard" },
  "permissions": {
    "allow": ["Bash(pnpm check:*)", "Bash(pnpm test *)", "Bash(pnpm build)", "Bash(git status)", "Bash(git diff *)", "Bash(git log *)"],
    "deny": [
      "Read(./.env)", "Read(./.env.*)", "Read(./secrets/**)",
      "Bash(git commit --no-verify *)", "Bash(git commit -n *)", "Bash(git push --no-verify *)",
      "Bash(git push * main)", "Bash(git push --force *)", "Bash(git push -f *)", "Bash(git reset --hard *)",
      "Bash(pnpm deploy:prod *)"
    ]
  }
}
```

1. commit `.claude/settings.json` คนที่ clone repo จะได้ ECC เวอร์ชันเดียวกันโดยไม่ต้องติดตั้งเอง
2. คัดลอก rules ของ ECC ไปไว้ที่ `.claude/rules/ecc/` เพราะ plugin แจก rule ให้ไม่ได้ (ยกเว้น `web/design-quality.md`)
3. ตรวจด้วย `claude plugin list` ว่า `ecc@ecc` เปิดอยู่ แล้วบันทึกเวอร์ชันใน AGENTS.md
4. ทุกเครื่องลงทะเบียน hook ของ ck ใน `~/.claude/settings.json` ครั้งเดียว แล้วรัน `/ecc:ck init` ครั้งเดียวต่อ repo
5. อัปเดต ECC ทุกไตรมาส ลองกับ repo เดียวก่อน แล้วค่อยขยายไประบบอื่น

ชื่อคำสั่ง deploy ใน deny ให้เปลี่ยนตามที่แต่ละระบบใช้จริง ถ้า hook ของ ECC ตัวไหนรบกวน ปิดเป็นรายตัวด้วย `ECC_DISABLED_HOOKS`

### Plugin ที่ซ้อนกับ ECC

Claude เลือก skill จากคำอธิบาย ถ้ามีสอง plugin ที่ทำงานเดียวกัน Claude อาจหยิบของอีกชุดมาใช้แทน ECC

| Plugin | ใน repo INT | เหตุผล |
| --- | --- | --- |
| superpowers | ปิด | มี workflow วางแผน → TDD → review ครบชุดเหมือน ECC |
| impeccable, ui-ux-pro-max, design | เปิดได้ | งานดีไซน์ที่ ECC ไม่มี |
| engineering, product-management | เปิดได้ | เรียกด้วยชื่อเต็ม ไม่ชน แต่ review ต้องผ่าน `/ecc:code-review` เสมอ |

### ck: ความต่อเนื่องระหว่าง session

ck (Context Keeper) มากับ ECC เรียกด้วย `/ecc:ck <คำสั่ง>` เก็บว่าทำถึงไหน ขั้นต่อไป และ blocker ไว้ที่ `~/.claude/ck/` ของเครื่องคนทำ จึงเป็นสมุดจดส่วนตัว ไม่ใช่ source of truth การตัดสินใจที่มีผลเกิน session ต้องลง plan หรือ ADR ด้วย

| คำสั่ง | ใช้เมื่อ |
| --- | --- |
| `/ecc:ck init` | ครั้งแรกที่ทำงานกับ repo นั้น |
| `/ecc:ck resume` | เริ่ม session ใหม่ |
| `/ecc:ck save` | ก่อน `/clear`, ก่อนปิด session และหลังจบแต่ละ phase |
| `/ecc:ck list`, `/ecc:ck info` | ดูภาพรวมทุก repo |

ไม่ใช้ `/ecc:save-session` และ `/ecc:resume-session` คู่กับ ck เพื่อไม่ให้มีสองที่เก็บที่ขัดกัน

### สภาพแวดล้อม WSL

- Claude Code, ECC และ repo ทั้งหมดอยู่ฝั่ง WSL plugin อยู่ที่ `~/.claude/plugins` ของ Linux แยกจาก Claude ฝั่ง Windows
- เก็บ repo ไว้ใน filesystem ของ Linux เช่น `~/projects/` ห้ามไว้ใต้ `/mnt/c/` เพราะช้าและ file watching ไม่นิ่ง
- skill `e2e-testing` ต้องติดตั้ง browser ของ Playwright ใน WSL ด้วย `pnpm exec playwright install --with-deps chromium`
- เปิด repo ใน VS Code ผ่าน WSL extension เพื่อให้ editor กับ Claude Code เห็นไฟล์ชุดเดียวกัน

## CLAUDE.md, AGENTS.md และ rules/int.md

AGENTS.md เป็นกฎชุดเดียวที่ agent ทุกตัวอ่าน `CLAUDE.md` มีแค่บรรทัด `@AGENTS.md` กับพฤติกรรมเฉพาะ Claude Code

- เริ่มด้วย `/init` แล้วตัดให้เหลือเฉพาะสิ่งที่คนมาใหม่ต้องรู้ในวันแรก ยาวไม่เกิน 200 บรรทัด
- ทุกบรรทัดต้องผ่านคำถาม "ถ้าลบบรรทัดนี้ Claude จะทำผิดไหม" ถ้าไม่ ให้ลบ
- Claude ทำผิดเรื่องเดียวกันครั้งที่สอง ให้เพิ่มการแก้ไขเข้า AGENTS.md ทันที
- ใช้ `IMPORTANT` เน้นได้ไม่เกินหนึ่งหรือสองบรรทัด และแก้ไฟล์เหล่านี้ผ่าน commit เหมือนโค้ด

| ใส่ | ไม่ใส่ |
| --- | --- |
| คำสั่ง build, test, lint ที่ Claude เดาไม่ได้ | สิ่งที่อ่านจากโค้ดแล้วรู้เอง |
| กฎที่ต่างจากค่าปกติของภาษา | convention มาตรฐานที่ Claude รู้อยู่แล้ว |
| การตัดสินใจทางสถาปัตยกรรมเฉพาะโปรเจกต์ | เอกสาร API ละเอียด (ใส่ลิงก์แทน) |
| ข้อควรระวังที่ไม่ชัดเจน | ข้อมูลที่เปลี่ยนบ่อย |

### ส่วนบังคับใน AGENTS.md

```markdown
## Commands
- Check: pnpm check:all (ต้องผ่านทุกตัว)
- Test: pnpm test (เขียวทั้งหมด ห้ามข้ามหรือลบ test ที่ fail)
- Build: pnpm build
## Verifying your work
รันทั้งสามคำสั่งก่อนรายงานว่างานเสร็จ และแนบผลลัพธ์ ถ้า test fail ให้แก้โค้ด ไม่ใช่แก้ test
## Workflow
อ่าน .claude/plans/<YYYY-MM-DD>-<slug>.plan.md ก่อนเริ่ม และอัปเดตเมื่อทำต่างจาก plan
## Things Claude gets wrong
- (เพิ่มตามกฎทำผิดสองครั้ง)
```

### rules/int.md

ไฟล์เดียวที่รวมกฎของ INT ซึ่ง ECC ไม่รู้ และ `/ecc:code-review` ใช้เป็นข้อตรวจ

- gate: ไม่มีโค้ดก่อน plan ผ่าน Gate 1 และไม่ push ก่อน Gate 2
- ที่เก็บ plan และบรรทัดหัว plan: `สถานะ · ขนาด (S/M/L) · อนุมัติโดย · ผู้ขอยืนยัน · Task`
- ขนาดงาน S/M/L ตัดสินก่อนเริ่มทุกครั้งตามตารางในหัวข้อถัดไป ถ้าไม่แน่ใจให้ถาม อย่าเลือกขนาดใหญ่ไว้ก่อน
- UI ตาม `.claude/design/DESIGN.md` และ Nielsen's 10 Heuristics
- coverage 80% ขึ้นไป
- ข้อตรวจตอน review: ตรงกับ Requirements ของ plan, ไม่มีงานนอกขอบเขต, ไม่มี secret หรือข้อมูลส่วนบุคคลใน log Important คือสิ่งที่ทำให้ระบบพัง ข้อมูลรั่ว หรือผิดนโยบาย เรื่องชื่อและสไตล์เป็น Nit
- reviewer เฉพาะทางเรียกเมื่อมีเหตุเท่านั้น: `security-reviewer` (auth, API, ข้อมูลส่วนบุคคล), `database-reviewer` (`prisma/`, `db-functions/`), `a11y-architect` (หน้าจอใหม่) rule ใน `rules/stack/` เตือนซ้ำตอน Claude แตะไฟล์ในโฟลเดอร์นั้น

### ส่วนบังคับใน CLAUDE.md

```markdown
@AGENTS.md

# Claude Code only
## Subagents
- ใช้ subagent เพื่อกรอง context: งานที่อ่านเยอะแต่ผลสั้น (ค้น codebase ใช้ Explore)
- ทุกครั้งที่ส่งงาน กำหนดรูปแบบผลลัพธ์: รายการไม่เกิน 10 ข้อ มี file:line ห้ามแปะโค้ด log หรือเนื้อไฟล์
- ห้ามส่งงานต่อกันเป็นทอด และห้ามเรียก reviewer ที่ไม่มีเหตุตาม rules
## Session
- เริ่มด้วย /ecc:ck resume จบด้วย /ecc:ck save
- ตอน compact ให้เก็บรายชื่อไฟล์ที่แก้, path ของ plan และคำสั่ง test
```

### ใช้ agent ให้คุ้ม token (ส่วนหนึ่งของ rules/int.md)

ระบบหลาย agent ใช้ token มากกว่าแชทปกติประมาณ 15 เท่า (ตามที่ Anthropic วัดจากระบบของตัวเอง) และคุ้มกับงานที่แยกทำพร้อมกันได้ ไม่คุ้มกับงานที่ทุกส่วนต้องรู้บริบทเดียวกัน จึงใช้ subagent เป็นตัวกรอง context ไม่ใช่การเพิ่มคนในทีม

**1. แบ่งระดับงานตามขนาด** ระบุขนาดไว้ในบรรทัด Complexity ของ plan

| ขนาด | ตัวอย่าง | ใช้อะไร |
| --- | --- | --- |
| S | แก้คำผิด, แก้ไฟล์เดียว, ปรับ style | session เดียว ไม่มี plan ไม่เรียก agent ผ่าน lefthook ก่อน commit |
| M | ฟีเจอร์เล็กในโดเมนเดียว | `/ecc:plan` → `tdd-workflow` → `/ecc:code-review` ครั้งเดียว |
| L | ฟีเจอร์ข้ามโดเมน, migration, auth, หน้าจอใหม่ | ครบทุกขั้น + reviewer เฉพาะทางตามข้อ 2 + `e2e-testing` |

**2. เรียก reviewer เฉพาะทางเมื่อมีเหตุเท่านั้น**

- `security-reviewer`: diff แตะ auth, API หรือข้อมูลส่วนบุคคล
- `database-reviewer`: diff แตะ `prisma/` หรือ `db-functions/`
- `a11y-architect`: มีหน้าจอใหม่หรือ component ที่มีการโต้ตอบ
- กรณีอื่นใช้ `/ecc:code-review` ตัวเดียว

**3. subagent ส่งผลกลับสั้นเสมอ**

- ตอบเป็นรายการปัญหา แต่ละข้อมี `file:line` ไม่เกิน 10 ข้อ
- ห้ามแปะโค้ด log หรือเนื้อไฟล์กลับมา เพราะผลที่ส่งกลับเข้า context หลักทันที
- งานค้น codebase ใช้ subagent Explore ที่มากับ Claude Code

**4. Subagent และ effort ในแต่ละระยะ (ไฟล์ `.claude/rules/ecc-agents.md`)**

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

**วัดผล:** เก็บ token ต่อ plan แยกตามขนาด S/M/L จาก hook `cost-tracker` ของ ECC แล้วทบทวนกติกานี้ทุกเดือนพร้อมตัวชี้วัดอื่น

## แนวปฏิบัติประจำวัน

ข้อจำกัดหลักของ Claude คือ context window ยิ่งเต็ม Claude ยิ่งลืมคำสั่งเดิม แนวปฏิบัติส่วนใหญ่จึงเป็นการรักษา context ให้สะอาด

### ระหว่างทำงาน

- ให้ Claude มีทางตรวจงานตัวเองเสมอ ระบุ test, build หรือภาพที่ต้องตรงกัน และขอหลักฐานแทนคำยืนยัน
- สำรวจ → วางแผน → ลงมือ → commit ใช้ plan mode เมื่อแก้หลายไฟล์หรือยังไม่แน่ใจแนวทาง
- prompt ให้เจาะจง ระบุไฟล์ สถานการณ์ และชี้ไปที่โค้ดต้นแบบ ใช้ `@` อ้างไฟล์แทนการอธิบาย
- แก้ทิศทางเร็ว กด `Esc` เพื่อหยุด `Esc Esc` หรือ `/rewind` เพื่อย้อนกลับ

### จัดการ context

- `/clear` ทุกครั้งที่เปลี่ยนไปทำงานที่ไม่เกี่ยวกัน แก้เรื่องเดิมไม่สำเร็จสองครั้ง ให้ `/clear` แล้วเริ่มใหม่ด้วย prompt ที่ดีกว่า
- งานสำรวจ codebase ส่งให้ sub-agent Explore ทำ เพื่อไม่ให้ context หลักเต็ม
- ตั้งชื่อ session ด้วย `/rename` และกลับมาทำต่อด้วย `claude --continue`

### ทำงานคู่ขนาน

- งานที่แตะไฟล์คนละชุด แยก session ด้วย `claude --worktree <ชื่องาน>` เริ่มที่ 2 ถึง 3 session
- รูปแบบ Writer/Reviewer: session หนึ่งเขียน อีก session ที่ context สะอาดรัน `/ecc:code-review`

| อาการ | วิธีแก้ |
| --- | --- |
| ทำหลายเรื่องปนกันใน session เดียว | `/clear` ระหว่างงาน |
| แก้ซ้ำไปซ้ำมา | หลังสองครั้ง `/clear` แล้วเขียน prompt ใหม่ |
| AGENTS.md ยาวจน Claude ข้ามกฎ | ตัดให้สั้น กฎที่ต้องเป็นจริงเสมอย้ายไป lefthook หรือ permission |
| เชื่อว่าเสร็จโดยไม่มีหลักฐาน | ตรวจไม่ได้ ไม่ส่ง |
| สำรวจไม่มีขอบเขต | จำกัดขอบเขต หรือส่งให้ sub-agent |

## ตัวชี้วัด

วัดจากข้อมูลที่ git และไฟล์ใน repo เก็บอยู่แล้ว ไม่ต้องกรอกเพิ่ม ทุกเดือน Product Owner ทบทวนตัวชี้วัด AGENTS.md และ `rules/int.md` ว่ายังตรงกับสิ่งที่เกิดจริง

| ระยะ | ตัวชี้วัด | อ่านจาก |
| --- | --- | --- |
| Plan | เวลาจากคุยครั้งแรกถึง plan ผ่าน Gate 1 | git log |
| Design | จำนวน commit ที่แก้ Requirements หลังผ่าน Gate 1 | git log |
| Build | จำนวนครั้งที่ Claude ทำผิดซ้ำเรื่องที่ AGENTS.md ควรกันไว้ | ประวัติ AGENTS.md |
| Test | bug ที่พบหลัง release ต่อ plan | `docs/lessons/`, ticket |
| Deploy | เวลาจาก plan ผ่าน Gate 1 ถึง release | git log, tag |
| Maintain | ปัญหาประเภทเดิมที่เกิดซ้ำ | `docs/lessons/`, `/ecc:instinct-status` |

## ยังไม่บังคับในรอบนี้

- CI บน GitHub, branch protection, CODEOWNERS และ Environments ของ production เพิ่มเมื่อทีมโตขึ้น โดยใช้คำสั่งชุดเดียวกับ `lefthook.yml`
- package กลางของ Corporate Identity (`@int/ui`) ที่ทุกระบบติดตั้งแทนการคัดลอก token

## แหล่งอ้างอิง

- [The AI-Native SDLC Playbook](<https://cdn.prod.website-files.com/6889473510b50328dbb70ae6/6aba736348bdf4f08183f463_AI%20Native%20SDLC%20Playbook_designv2%20(1).pdf>) (Anthropic)
- [Explore the .claude directory](https://code.claude.com/docs/en/claude-directory)
- [Best practices for Claude Code](https://code.claude.com/docs/en/best-practices)
- [ECC repository](https://github.com/affaan-m/everything-claude-code)
- [ECC Complete Cheatsheet](https://dev.to/shimo4228/everything-claude-code-ecc-complete-cheatsheet-24ok)
- [ck (Context Keeper) skill](https://github.com/affaan-m/ECC/blob/main/skills/ck/SKILL.md)
- [Lefthook](https://github.com/evilmartians/lefthook)
