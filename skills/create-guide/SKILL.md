---
name: Create Guide
author: Anthropic
version: 1.0.0
created: 2025-06-26
updated: 2025-06-26
tags: [documentation, guide, html, capture, bilingual]
status: stable
---

# Create Guide Skill

## Description
สร้างคู่มือการใช้งาน HTML แบบ standalone สำหรับโมดูลใดก็ได้ในโปรเจค โดยรวบรวมข้อมูลจาก 4 แหล่งพร้อมกัน:
1. **CLAUDE.md** — สี/convention ของโมดูล
2. **Source Code** — ปุ่ม/form/dialog ที่มีจริง
3. **Route Structure** — path และ menu label
4. **Screenshots** — รูปหน้าจอที่ capture ไว้ (หรือ vision analysis)

ผลลัพธ์คือ **ไฟล์ HTML ไฟล์เดียว** — รูปทั้งหมด embed เป็น base64 แชร์ได้ทันที ไม่ต้องมี server

## When to Use
- สร้างคู่มือการใช้งาน HTML สำหรับโมดูล/หน้าใหม่
- Update คู่มือหลังมีการเปลี่ยนแปลง UI/ฟีเจอร์
- Capture screenshot อัตโนมัติด้วย Playwright (2 viewport: PC + mobile)
- สร้างคู่มือ 2 ภาษา (ไทย + อังกฤษ) พร้อม toggle ปุ่ม

## Key Features

### 1. Auto-Capture (--capture flag)
- **Playwright automation** — login + screenshot ทั้งไทย (PC 1440×900) และ mobile (390×844)
- **Interactive planning** — ถามผู้ใช้ว่า dialog/button ไหนควรเปิดสำหรับ capture
- **DevTools auto-hide** — ซ่อน React Query/TanStack Query DevTools ก่อน screenshot
- **Safe credential** — เก็บ session ไว้นอก project directory (`~/docs/<projectName>/`)

### 2. Smart Caching
- **Cache ที่ทำแล้ว** — ข้ามขั้นตอนที่รูป/data/code มีอยู่แล้ว
- **--refresh-* flags** — บังคับอ่านใหม่เฉพาะส่วนที่ต้องการ
- **--fresh flag** — ล้าง cache ทั้งหมด + capture ใหม่

### 3. Flexible Output
- **ภาษา**: ไทยเท่านั้น / อังกฤษเท่านั้น / 2 ภาษา (พร้อม toggle)
- **Device tabs** — PC / Mobile screenshot สลับในคลิกเดียว
- **Standalone HTML** — เปิดด้วย browser ได้เลย ไม่ต้องมี server

### 4. Skip Flags (ลด token)
```
--no-plan        → ข้ามการถามแผน (ใช้ extraCaptures เดิม)
--overview-only  → capture overview เท่านั้น
--no-code        → ข้ามการอ่าน source code (ใช้ cache)
--no-vision      → ข้าม vision analysis (ใช้ filename pattern)
--html-only      → ข้ามทั้งหมด สร้าง HTML จาก cache
--lite           → shorthand ของ --no-plan --overview-only --no-vision
```

## Instructions

### Basic Usage
```bash
/create-guide <moduleName>                  # ไทย (ใช้ cache)
/create-guide <moduleName> en               # อังกฤษ
/create-guide <moduleName> th+en            # 2 ภาษา + toggle
```

### With Auto-Capture
```bash
/create-guide <moduleName> --capture        # ถามแผน + capture PC+mobile (เฉพาะที่ขาด)
/create-guide <moduleName> --capture --no-mobile    # capture PC เท่านั้น
/create-guide <moduleName> --capture --replan      # capture + ถามแผนใหม่
/create-guide <moduleName> --capture --recapture   # capture รูปใหม่ทุกรูป
```

### Token Optimization
```bash
/create-guide <moduleName> --capture --lite        # capture + ข้ามแผน/overview/vision (~60-80% token)
/create-guide <moduleName> --html-only             # HTML จาก cache ทันที (~5% token)
/create-guide <moduleName> --refresh-images        # vision-อ่านรูปใหม่ (วางรูปทับ)
```

### Combo Examples
```bash
/create-guide basket-circulation th+en --capture                # 2 ภาษา + capture ใหม่
/create-guide basket-circulation --capture --no-plan --no-vision # capture เงียบ
/create-guide receiving --html-only                             # module อื่น ใช้ cache
```

## Output Locations
- **Config**: `~/docs/<projectName>/guide.config.json` — cache ทั้งหมด
- **Images**: `~/docs/<projectName>/images/` — screenshot ที่ capture
- **HTML**: `~/docs/<projectName>/<moduleName>-guide.html` (หรือ `-en` / `-bilingual`)
- **Credentials** (นอก project): `~/docs/<projectName>/capture.env`, `auth.json`

## Example Workflow

### ครั้งแรก (ต้อง capture):
```bash
/create-guide basket-circulation --capture
# → ถามแผน → login → capture screenshot → สร้าง HTML
```

### ครั้งต่อไป (แค่ update HTML):
```bash
/create-guide basket-circulation
# → ใช้ cache ที่มีอยู่ → สร้าง HTML ใหม่ (เร็ว)
```

### เพิ่มรูปเอง แล้วอ่านใหม่:
```bash
# 1. วางรูปใหม่ใน ~/docs/<projectName>/images/
# 2. รัน:
/create-guide basket-circulation --refresh-images
# → vision-อ่านรูปใหม่ → สร้าง HTML
```

## Config File Structure

```json
{
  "baseUrl": "http://localhost:3000",
  "framework": "nextjs",
  "modules": {
    "basket-circulation": {
      "meta": {
        "color": "green",
        "description": "ระบบหมุนเวียนตะกร้า",
        "cachedAt": "2025-06-10T10:00:00Z"
      },
      "routes": [
        {"slug": "bc-domains", "path": "/basket-circulation/domains", "title": "Domains"}
      ],
      "pages": {
        "bc-domains": {
          "buttons": [{"label": "Add Domain", "type": "dialog"}],
          "formFields": ["Domain Name", "Basket Size"],
          "cachedAt": "2025-06-10T10:00:00Z"
        }
      }
    }
  }
}
```

## Tips & Tricks
- **ต้องเข้าระบบทีละครั้ง** — session เก็บไว้ 8 ชั่วโมง
- **การ capture ไม่ submit ข้อมูล** — ข้าม delete/submit action อัตโนมัติ
- **ข้อมูลจริง vs Demo** — ใช้ staging/demo data หากต้องแชร์คู่มือภายนอก
- **ขาด dialog/button?** — รัน `--refresh-code` เพื่ออ่าน source ใหม่
- **เปลี่ยนรูป?** — วางรูป แล้วรัน `--refresh-images` (ไม่ต้องล้าง cache ทั้งหมด)

## Troubleshooting
- **Login ล้มเหลว** → ตรวจสอบ selector ในไฟล์ script (อาจเปลี่ยนตาม framework)
- **รูปไม่ crop ถูก** → ปรับ `--overview-only` หรือ `extraCaptures` ใน config
- **HTML ขาดรูป** → ตรวจสอบชื่อไฟล์ + extension ต้องตรงกับ slug
