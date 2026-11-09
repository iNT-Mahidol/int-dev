---
description: วางไฟล์มาตรฐาน INT ลงในโปรเจกต์ปัจจุบัน ถามก่อนทับไฟล์ที่ต่างจากเดิมทุกไฟล์
disable-model-invocation: true
argument-hint: "[stack=nextjs|none] [design=int|client]"
---

# /int:init

อาร์กิวเมนต์: $ARGUMENTS

## ข้อห้าม

- ห้าม commit, push หรือรัน install ใดๆ
- ห้ามอ่านหรือแก้ `.env` และไฟล์ secret
- ห้ามทับไฟล์เงียบ ไฟล์ที่ต่างจากเดิมต้องแสดง diff และถามทีละไฟล์ ถ้าถามไม่ได้ให้ข้ามและรายงาน
- ค่าใน Bash ไม่ข้ามระหว่าง tool call ให้จำ path ของ `ROOT`, `STAGE`, `ECC` แล้วเขียนเต็มทุกครั้ง

## 1. ตรวจ git repo

รัน `git rev-parse --show-toplevel` ถ้าไม่สำเร็จให้แจ้งว่า "ต้องรันใน git repo" แล้วหยุด ผลลัพธ์คือ `ROOT`

## 2. รับ stack และ design

อ่าน `stack=` และ `design=` จากอาร์กิวเมนต์ ค่าที่รับคือ `stack`: `nextjs` หรือ `none` และ `design`: `int` หรือ `client` ถ้าไม่ระบุหรือค่าไม่ถูกต้องให้ถามผู้ใช้

`design=client` คือโปรเจกต์ลูกค้า จะไม่คัด DESIGN ของ INT แต่สร้าง `.claude/design/DESIGN.md` เปล่าให้กรอกแบรนด์ลูกค้า

## 3. ถามคำสั่ง Check, Test, Build

ถามผู้ใช้ทั้งสามคำสั่ง ถ้า `stack=nextjs` เสนอค่าเริ่มต้น `pnpm check:all`, `pnpm test`, `pnpm build`

แต่ละค่าต้องไม่ว่าง เป็นบรรทัดเดียว ไม่มี `: ` หรือ ` #` และไม่ขึ้นต้นด้วยอักขระพิเศษของ YAML (`{ [ & * ! | > ' " % @` และ backtick) เพราะค่าจะถูกวางลงใน `lefthook.yml` ถ้าไม่ผ่านให้ขอให้ห่อเป็น script ใน `package.json` แล้วถามใหม่

## 4. เตรียมไฟล์ใน staging

ทุกไฟล์เตรียมในโฟลเดอร์ชั่วคราวก่อน เพื่อเทียบกับโปรเจกต์หลังแทนค่าแล้ว

```bash
STAGE="$(mktemp -d)"
cp -R "${CLAUDE_PLUGIN_ROOT}/templates/base/." "$STAGE/"
mkdir -p "$STAGE/docs"
cp -R "${CLAUDE_PLUGIN_ROOT}/templates/docs/." "$STAGE/docs/"
echo "$STAGE"
```

ถ้า `stack=nextjs`:

```bash
cp -R "${CLAUDE_PLUGIN_ROOT}/templates/stacks/nextjs/." "$STAGE/"
```

ถ้า `stack=none` ให้ตัดงาน `lint` ออกจาก lefthook เพราะผูกกับ pnpm และ eslint:

```bash
perl -0pi -e 's/^    lint:\n(      .*\n)+//m' "$STAGE/lefthook.yml"
```

## 5. คัด rules ของ ECC

หา path ที่ติดตั้ง (โปรเจกต์นี้ก่อน แล้ว user scope แล้วรายการแรก):

```bash
jq -r --arg root "ROOT" '(.plugins["ecc@ecc"] // []) as $e | (($e | map(select(.projectPath == $root)) | first) // ($e | map(select(.scope == "user")) | first) // ($e | first)) | .installPath // empty' "${CLAUDE_CONFIG_DIR:-$HOME/.claude}/plugins/installed_plugins.json"
```

ผลลัพธ์คือ `ECC` ถ้าว่าง หรือ `ECC/rules` ไม่มีอยู่ ให้แจ้งวิธีติดตั้งต่อไปนี้ กำหนด `ECC_VERSION=not-installed` แล้วข้ามไปขั้น 6

```text
claude plugin marketplace add affaan-m/everything-claude-code
claude plugin install ecc@ecc
```

ถ้าพบ ECC ให้อ่านเวอร์ชันจาก `ECC/.claude-plugin/plugin.json` เป็น `ECC_VERSION` แล้วคัด `common` เสมอ:

```bash
mkdir -p "$STAGE/.claude/rules/ecc"
cp -R "$ECC/rules/common" "$STAGE/.claude/rules/ecc/"
```

ถ้า `stack=nextjs` คัดเพิ่ม `typescript`, `react` และ `web` โดยไม่เอา `web/design-quality.md` ในทุกกรณี:

```bash
cp -R "$ECC/rules/typescript" "$ECC/rules/react" "$STAGE/.claude/rules/ecc/"
mkdir -p "$STAGE/.claude/rules/ecc/web"
find "$ECC/rules/web" -maxdepth 1 -type f ! -name design-quality.md -exec cp {} "$STAGE/.claude/rules/ecc/web/" \;
```

## 6. Design

ถ้า `design=int`:

```bash
mkdir -p "$STAGE/.claude/design"
cp -R "${CLAUDE_PLUGIN_ROOT}/design-profiles/int/." "$STAGE/.claude/design/"
```

ถ้า `design=client` ให้สร้าง `$STAGE/.claude/design/DESIGN.md` ด้วยเนื้อหานี้:

```markdown
# Design Profile: <ชื่อแบรนด์ลูกค้า>

Component library:
Fonts:
Themes:
Source:

## หลักการ
## สี
## ธีมสว่างและมืด
## ตัวอักษร
## ระยะและ layout
## โครงสร้างหน้าจอ
## ข้อความ (UX writing)
## ไอคอน
## สถานะและการตอบสนอง
## ห้าม
```

## 7. แทนค่า เทียบ และวางไฟล์

### แทน placeholder

ใส่ค่าที่ได้ผ่านตัวแปรแวดล้อม เพื่อให้อักขระพิเศษในคำสั่งไม่ถูกตีความ `VERSION` อ่านจาก `${CLAUDE_PLUGIN_ROOT}/.claude-plugin/plugin.json`

```bash
VERSION='...' STACK='...' DESIGN='...' ECC_VERSION='...' CHECK_CMD='...' TEST_CMD='...' BUILD_CMD='...' \
  perl -pi -e 's/\{\{(?<k>VERSION|STACK|DESIGN|ECC_VERSION|CHECK_CMD|TEST_CMD|BUILD_CMD)\}\}/$ENV{$+{k}}/g' "$STAGE/AGENTS.md" "$STAGE/lefthook.yml"
grep -rn '{{' "$STAGE" --include='AGENTS.md' --include='lefthook.yml'
```

ถ้า `grep` ยังเจอ `{{` ให้หยุดและรายงาน

### เทียบกับโปรเจกต์

```bash
cd "$STAGE" && find . -type f ! -path ./gitignore.append | sort | while IFS= read -r f; do
  t="ROOT/${f#./}"
  if [ ! -e "$t" ]; then echo "NEW $f"; elif cmp -s "$f" "$t"; then echo "SAME $f"; else echo "DIFF $f"; fi
done
```

| ผล | ทำอย่างไร | ลงตารางเป็น |
| --- | --- | --- |
| `NEW` | `mkdir -p` โฟลเดอร์ปลายทาง แล้ว `cp` จาก staging | สร้าง |
| `SAME` | ไม่แตะ | ข้าม (เหมือนเดิม) |
| `DIFF` | แสดงผล `diff -u <ไฟล์เดิม> <ไฟล์ใหม่>` (ยาวเกิน 80 บรรทัดให้ตัดและบอกจำนวนบรรทัดทั้งหมด) แล้วถาม "ทับ" หรือ "ข้าม" ทีละไฟล์ ห้ามถามรวม | ทับ หรือ ข้าม (ผู้ใช้เลือก) |

วางไฟล์ด้วย `cp` เท่านั้น ห้ามพิมพ์เนื้อหาใหม่ เพื่อให้ไฟล์ตรงกับ template ทุกไบต์

### ต่อท้าย .gitignore

```bash
cd "ROOT" && touch .gitignore
[ -n "$(tail -c1 .gitignore)" ] && echo >> .gitignore
while IFS= read -r line; do
  grep -qxF -- "$line" .gitignore || { printf '%s\n' "$line" >> .gitignore; echo "ADDED $line"; }
done < "${CLAUDE_PLUGIN_ROOT}/templates/base/gitignore.append"
```

ถ้ามีบรรทัด `ADDED` ลงตารางเป็นแก้ไข ถ้าไม่มีลงตารางเป็นข้าม

### เพิ่ม prepare ใน package.json

ทำเมื่อมี `ROOT/package.json` เท่านั้น ตรวจก่อนด้วย `jq -e 'type == "object"' package.json` ถ้าไม่ผ่านให้ข้ามและรายงาน

| `.scripts.prepare` ปัจจุบัน | ทำอย่างไร |
| --- | --- |
| ไม่มี | เพิ่ม (คำสั่งด้านล่าง) |
| `lefthook install` | ข้าม |
| ค่าอื่น | แสดงค่าเดิม แล้วถาม "ทับ" หรือ "ข้าม" |

```bash
cd "ROOT" && jq '.scripts.prepare = "lefthook install"' package.json > "$STAGE/package.json.new" && cp "$STAGE/package.json.new" package.json
```

## 8. สรุป

แสดงตาราง `ไฟล์ | ผล` โดยผลเป็น สร้าง, ทับ, แก้ไข หรือ ข้าม พร้อมจำนวนรวมแต่ละแบบ แล้วบอกขั้นตอนที่ผู้ใช้ต้องทำเองตามลำดับนี้:

1. ติดตั้ง `gitleaks` ให้เรียกได้จาก PATH (https://github.com/gitleaks/gitleaks)
2. `pnpm add -D lefthook` ต้องทำก่อน `pnpm install` เพราะ script `prepare` เรียก `lefthook`
3. `pnpm install`
4. `/ecc:ck init`
5. `claude plugin list` ตรวจว่า `ecc@ecc` เปิดอยู่
6. ปรับ `pnpm deploy:prod` ใน `.claude/settings.json` ให้ตรงคำสั่ง deploy จริงของระบบ
7. `/int:doctor`
