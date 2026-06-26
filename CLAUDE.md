# PI Claude Md

ที่เก็บกลางของทีมสำหรับ Claude Skills, Plugins, และเอกสาร

---

## การติดตั้ง (สำหรับสมาชิกทีม)

```bash
# 1. Clone repo
git clone https://github.com/ken1lnw/pi-claude-md.git
cd pi-claude-md

# 2. เปิดใน Claude Code
claude .
```

Claude Code จะอ่าน `CLAUDE.md` นี้อัตโนมัติเมื่อเปิดโฟลเดอร์ — ไม่ต้อง install อะไรเพิ่ม

ถ้าต้องการใช้ skill ในโปรเจคอื่น ให้ copy `skills/<ชื่อ>/SKILL.md` ไปไว้ใน `.claude/skills/` ของโปรเจคนั้น หรือ symlink:

```bash
ln -s ~/path/to/pi-claude-md/skills/<ชื่อ>/SKILL.md ~/.claude/skills/<ชื่อ>.md
```

---

## โครงสร้างโปรเจค

```
pi-claude-md/
├── skills/
│   ├── <skill-name>/SKILL.md   ← skill แต่ละตัว
│   └── example-skill/          ← template สำหรับ skill ใหม่
├── plugins/
│   └── example-plugin/         ← template สำหรับ plugin ใหม่
├── docs/
├── INDEX.md                    ← master index (ค้นหา skills/plugins)
└── CLAUDE.md                   ← this file
```

---

## การเพิ่ม Skill ใหม่

```bash
mkdir -p skills/<ชื่อ-kebab-case>
cp skills/example-skill/SKILL.md skills/<ชื่อ>/SKILL.md
# แก้ไข SKILL.md แล้วอัปเดต INDEX.md
```

**Naming**: โฟลเดอร์ใช้ `kebab-case` เสมอ เช่น `create-guide`, `my-skill-name`

---

## โครงสร้าง SKILL.md

```markdown
---
name: Skill Name
author: your-name
version: 1.0.0
created: YYYY-MM-DD
updated: YYYY-MM-DD
tags: [tag1, tag2]
status: stable
---

# Skill Name

## Description
## When to Use
## Instructions
## Example
```

---

## อัปเดต INDEX.md (บังคับทุกครั้ง)

เมื่อเพิ่ม skill/plugin ใหม่ ต้องอัปเดต INDEX.md ใน 3 จุด:
1. เพิ่มแถวในตาราง Skills หรือ Plugins
2. เพิ่ม tags ในตาราง "ค้นหาตาม Tag"
3. อัปเดตสถิติ (จำนวน skills, plugins, contributors)

---

## Tags ที่ใช้อยู่

| Tag | ความหมาย |
|-----|---------|
| `documentation` | สร้างหรือจัดการเอกสาร |
| `guide` | คู่มือการใช้งาน |
| `html` | สร้าง HTML output |
| `capture` | Screenshot/Playwright automation |
| `bilingual` | รองรับหลายภาษา |
| `automation` | Automation scripts/flows |
| `example` | ตัวอย่าง/template |

---

## Best Practices

- ชื่อชัดเจน — `create-guide` ดีกว่า `guide`
- ทุก skill ต้องมี SKILL.md พร้อม metadata ครบ
- อัปเดต INDEX.md ทุกครั้งที่เพิ่ม skill ใหม่
- ใช้ semantic versioning — 1.0.0, 1.0.1, 1.1.0
- ไม่ hardcode credentials ในไฟล์ใดๆ
- ไม่ push binary files ขนาดใหญ่
