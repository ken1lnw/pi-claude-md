---
name: example-skill
description: ตัวอย่าง skill template สำหรับทีม PI ใช้เป็นจุดเริ่มต้นเมื่อสร้าง skill ใหม่ — copy โฟลเดอร์นี้แล้วแก้ frontmatter กับเนื้อหา
metadata:
  author: your-name
  version: 1.1.0
  created: 2025-01-01
  updated: 2026-07-14
  tags: [example, template]
  status: stable
---

# Example Skill

## Description
อธิบาย skill นี้ทำอะไร

## When to Use
ใช้เมื่อไหร่

## Instructions
วิธีใช้งาน step-by-step

## Example
Input: ...
Output: ...

---

## 📌 ก่อน commit — เช็ค 3 ข้อ

1. `name` เป็น **lowercase-kebab-case** และตรงกับชื่อโฟลเดอร์
2. มี `description` และมีคีย์เวิร์ดที่ผู้ใช้จะพิมพ์จริง (Claude ใช้บรรทัดนี้ตัดสินใจเรียก skill)
3. อัปเดต `INDEX.md` ที่ root แล้ว
