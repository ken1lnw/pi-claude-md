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

ถ้าต้องการใช้ skill ในทุกโปรเจค ให้ symlink **ทั้งโฟลเดอร์** เข้า `~/.claude/skills/`:

```bash
ln -s ~/path/to/pi-claude-md/skills/<ชื่อ> ~/.claude/skills/<ชื่อ>
```

> ⚠️ **ต้องเป็นโฟลเดอร์เท่านั้น** — Claude Code มองหา `~/.claude/skills/<ชื่อ>/SKILL.md`
> ถ้า symlink เป็นไฟล์ `.md` เดี่ยว ๆ (`~/.claude/skills/<ชื่อ>.md`) **skill จะไม่ถูกโหลดเลย** และจะไม่มี error บอกด้วย

ตรวจว่าโหลดสำเร็จ: พิมพ์ `/` ใน Claude Code แล้วต้องเห็นชื่อ skill ในลิสต์

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
│   └── external-skills.md      ← skills ของทีมอื่นที่ติดตั้งผ่าน marketplace
├── INDEX.md                    ← master index (ค้นหา skills/plugins)
└── CLAUDE.md                   ← this file
```

---

## Skills จากภายนอก (ไม่เก็บไฟล์ใน repo นี้)

skills ของทีมอื่น (เช่นชุด .NET/Akka/Aspire 35 ตัว) **ห้าม copy ไฟล์เข้ามา** — ติดตั้งผ่าน marketplace แทน:

```
/plugin marketplace add Aaronontheweb/dotnet-skills
```

รายชื่อทั้งหมด + เหตุผล: [`docs/external-skills.md`](./docs/external-skills.md)

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
name: my-skill-name
description: อธิบายว่า skill ทำอะไร + ใช้เมื่อไหร่ ให้ครบในบรรทัดเดียว (ไม่เกิน 1024 ตัวอักษร)
metadata:
  author: your-name
  version: 1.0.0
  created: YYYY-MM-DD
  updated: YYYY-MM-DD
  tags: [tag1, tag2]
  status: stable
---

# My Skill Name

## Description
## When to Use
## Instructions
## Example
```

**2 field ที่ Claude Code บังคับ — ผิดแล้ว skill ไม่ทำงาน:**

| Field | กติกา | ถ้าผิดจะเป็นยังไง |
|---|---|---|
| `name` | **lowercase-kebab-case** และต้องตรงกับชื่อโฟลเดอร์ เช่น `create-guide` | skill ไม่โหลด |
| `description` | **บังคับ** — Claude ใช้บรรทัดนี้ตัดสินใจว่าจะเรียก skill นี้ไหม ต้องมีคีย์เวิร์ดที่ผู้ใช้จะพิมพ์ | skill โหลดแต่ไม่เคยถูก trigger |

field ที่เหลือ (`author`, `version`, `tags`, `status`) เป็น metadata ของทีมเรา — ต้องอยู่ใต้ `metadata:` ไม่ใช่ระดับบนสุด

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
