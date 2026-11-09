@AGENTS.md

# Claude Code only
## Subagents
- ใช้ subagent เพื่อกรอง context: งานที่อ่านเยอะแต่ผลสั้น (ค้น codebase ใช้ Explore)
- ทุกครั้งที่ส่งงาน กำหนดรูปแบบผลลัพธ์: รายการไม่เกิน 10 ข้อ มี file:line ห้ามแปะโค้ด log หรือเนื้อไฟล์
- ห้ามส่งงานต่อกันเป็นทอด และห้ามเรียก reviewer ที่ไม่มีเหตุตาม rules
## Session
- เริ่มด้วย /ecc:ck resume จบด้วย /ecc:ck save
- ตอน compact ให้เก็บรายชื่อไฟล์ที่แก้, path ของ plan และคำสั่ง test
