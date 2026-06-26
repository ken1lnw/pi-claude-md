# 📚 Skills

รวม Claude Skill files ที่ทีมสร้างขึ้น

## วิธีเพิ่ม Skill ใหม่

1. สร้างโฟลเดอร์ใหม่: `skills/ชื่อ-skill/`
2. สร้างไฟล์ `SKILL.md` ข้างใน
3. ใส่เนื้อหาตาม template พร้อม metadata
4. อัปเดต `INDEX.md` ที่ root

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
อธิบาย skill นี้ทำอะไร

## When to Use
ใช้เมื่อไหร่

## Instructions
วิธีใช้งาน step-by-step

## Example
Input: ...
Output: ...
```
