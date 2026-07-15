# 📚 Skills

รวม Claude Skill ที่ทีมสร้างขึ้น

> **skills ของทีมอื่น** (ชุด .NET / Akka / Aspire ฯลฯ) ไม่เก็บที่นี่ — ติดตั้งผ่าน marketplace
> ดู [`../docs/external-skills.md`](../docs/external-skills.md)

## วิธีเพิ่ม Skill ใหม่

1. สร้างโฟลเดอร์: `skills/<ชื่อ-kebab-case>/`
2. สร้างไฟล์ `SKILL.md` ข้างใน (ชื่อไฟล์ต้องเป็น `SKILL.md` ตัวใหญ่ทั้งหมด)
3. ใส่ frontmatter ให้ครบตาม template ข้างล่าง
4. อัปเดต `INDEX.md` ที่ root

## โครงสร้าง SKILL.md

```markdown
---
name: my-skill-name
description: อธิบายว่า skill ทำอะไร + ใช้เมื่อไหร่ ให้ครบในบรรทัดเดียว
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
อธิบาย skill นี้ทำอะไร

## When to Use
ใช้เมื่อไหร่

## Instructions
วิธีใช้งาน step-by-step

## Example
Input: ...
Output: ...
```

## ⚠️ 2 เรื่องที่พลาดบ่อยที่สุด

### 1. `name` ต้องเป็น lowercase-kebab-case และตรงกับชื่อโฟลเดอร์

```yaml
name: Create Guide      # ❌ skill ไม่โหลด
name: create-guide      # ✅
```

### 2. `description` เป็น field บังคับ

Claude อ่านบรรทัดนี้เพื่อตัดสินใจว่า **จะเรียก skill นี้หรือไม่** — ถ้าไม่มี skill จะโหลดแต่ไม่เคยถูกใช้

```yaml
# ❌ ไม่มี description — skill เงียบตลอดกาล
name: create-guide
author: ken

# ✅ มีคีย์เวิร์ดที่ผู้ใช้จะพิมพ์จริง
name: create-guide
description: สร้างคู่มือ HTML แบบ standalone สำหรับโมดูล ใช้เมื่อผู้ใช้ขอ "ทำคู่มือ", "create guide", "user manual", หรือเอกสารสอนใช้งานระบบ
```

## ✅ ตรวจว่า skill ทำงานจริง

```bash
# 1. โครงสร้างถูกไหม — ต้องเป็นโฟลเดอร์ที่มี SKILL.md
ls ~/.claude/skills/<ชื่อ>/SKILL.md

# 2. เปิด Claude Code แล้วพิมพ์ /  → ต้องเห็นชื่อ skill ในลิสต์
```

ถ้าไม่เห็นชื่อในลิสต์ = frontmatter ผิด หรือวางไฟล์ผิดที่ (ไฟล์ `.md` เดี่ยว ๆ ใน `~/.claude/skills/` **จะไม่ถูกโหลด**)
