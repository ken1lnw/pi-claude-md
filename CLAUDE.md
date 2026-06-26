# PI Claude Md — Project Documentation

## 📌 About This Project

**PI Claude Md** คือที่เก็บกลางของทีม (Central Hub) สำหรับ:
- 🧠 **Claude Skills** — Custom tools ที่เพิ่มความสามารถของ Claude
- 🔌 **Plugins** — Integration tools และ extensions
- 📚 **Documentation** — Guide เอกสาร และ resources

ทีมใช้ที่เก็บนี้เป็นศูนย์กลางในการ share, maintain, และ version-control skills/plugins ของเรา

---

## 🚀 Featured Skills

### 🎨 Create Guide (`/create-guide`)
**สร้างคู่มือการใช้งาน HTML แบบ standalone สำหรับโมดูล**

สำหรับโปรเจค web ใดก็ได้ (Next.js, Vue, Angular, generic) — รวบรวมข้อมูลจาก 4 แหล่ง:
1. CLAUDE.md — สี/convention ของโมดูล
2. Source Code — ปุ่ม/form/dialog ที่มีจริง
3. Routes — path และ menu structure
4. Screenshots — รูปหน้าจอ (auto-capture หรือ vision analysis)

```bash
# ครั้งแรก — Playwright auto-capture + plan
/create-guide basket-circulation --capture

# ครั้งต่อไป — ใช้ cache เร็ว
/create-guide basket-circulation

# 2 ภาษา + toggle + capture
/create-guide basket-circulation th+en --capture --no-plan
```

**ลักษณะเด่น:**
- ✅ **Standalone HTML** — embed รูปเป็น base64 แชร์ได้ทันที
- ✅ **Auto-Capture** — Playwright automation PC + mobile viewport
- ✅ **Bilingual** — ไทย + อังกฤษ พร้อม toggle ปุ่ม
- ✅ **Smart Cache** — ข้ามขั้นตอนที่ทำแล้ว
- ✅ **Safe Credentials** — session + password เก็บนอก project

👉 เพิ่มเติม: [`skills/create-guide/SKILL.md`](./skills/create-guide/SKILL.md)

---

## 📂 Directory Structure

```
pi-claude-md/
├── skills/
│   ├── create-guide/          ← Guide generation skill
│   │   └── SKILL.md           ← Documentation
│   ├── example-skill/         ← Template for new skills
│   └── README.md
├── plugins/
│   ├── example-plugin/        ← Template for new plugins
│   └── README.md
├── docs/                      ← General documentation
├── INDEX.md                   ← Master index (ค้นหา skills/plugins)
├── README.md                  ← Quick start guide
├── CLAUDE.md                  ← This file
└── .github/                   ← GitHub workflows (CI/CD)
```

---

## 👨‍💻 Development Workflow

### สำหรับสมาชิกทีม

#### 1️⃣ Clone & Setup
```bash
git clone https://github.com/YOUR_ORG/pi-claude-md.git
cd pi-claude-md
# ไม่ต้อง npm install — เป็นแค่ content repo
```

#### 2️⃣ สร้าง Feature Branch
```bash
git checkout -b feature/ชื่อ-skill-ของคุณ
```

#### 3️⃣ เพิ่ม Skill ใหม่
```bash
mkdir -p skills/ชื่อ-skill
# คัดลอก template จาก skills/example-skill/SKILL.md
cp skills/example-skill/SKILL.md skills/ชื่อ-skill/
# แก้ไข SKILL.md ของคุณ
```

#### 4️⃣ อัพเดต INDEX.md
```bash
# เพิ่มแถวสำหรับ skill ใหม่
# เพิ่มสถิติในตาราง "สถิติ"
# เพิ่ม tags ใหม่ถ้ามี
```

#### 5️⃣ Commit & Push
```bash
git add .
git commit -m "feat: เพิ่ม skill ชื่อ-skill"
git push origin feature/ชื่อ-skill-ของคุณ
```

#### 6️⃣ Open Pull Request
- ที่ GitHub ให้ PR อธิบายว่า skill นี้ทำอะไร
- รอ code review จากทีม
- merge ลง main

#### 7️⃣ Pull อัพเดต (ที่เครื่องคุณ)
```bash
git checkout main
git pull origin main
```

---

## 🛠️ Creating Your Own Skill

### Minimal SKILL.md Structure
```markdown
---
name: My Skill Name
author: your-name
version: 1.0.0
created: 2025-06-26
updated: 2025-06-26
tags: [your, tags, here]
status: stable
---

# My Skill Name

## Description
อธิบาย skill ทำอะไร (ภาษาไทย)

## When to Use
ใช้เมื่อไหร่

## Instructions
วิธีใช้งาน step-by-step

## Example
Input: ...
Output: ...
```

### Naming Convention
- **Folder**: `skills/my-skill-name/` (kebab-case)
- **File**: `SKILL.md` (always)
- **In INDEX.md**: `[My Skill Name](./skills/my-skill-name/SKILL.md)`

---

## 📖 Tags & Categories

ใช้ tags เพื่อให้คนค้นหา skills ได้ง่าย:

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

## 🎯 Best Practices

### ✅ DO

- ✅ **ชื่อชัดเจน** — `create-guide` ดีกว่า `guide`
- ✅ **Documentation ครบ** — ทุก skill ต้องมี SKILL.md
- ✅ **อัพเดต INDEX.md** — ทุกครั้งที่เพิ่ม skill ใหม่
- ✅ **Version ตามลำดับ** — 1.0.0, 1.0.1, 1.1.0 (semantic versioning)
- ✅ **Git commit ชัดเจน** — `feat: เพิ่ม skill` หรือ `fix: แก้ bug ใน skill`
- ✅ **Tags ครบถ้วน** — เพื่อให้ค้นหาง่าย

### ❌ DON'T

- ❌ **ลืมอัพเดต INDEX.md** — ทำให้คนหา skill ไม่เจอ
- ❌ **Hardcode credentials** — ใช้ environment variables หรือ config files
- ❌ **Large binary files** — โปรเจคนี้ = content repo เท่านั้น
- ❌ **Unpublished work** — push ไปเมื่อ skill พร้อมใช้ได้จริง

---

## 🔍 Finding Skills

### วิธี 1: ดู INDEX.md
```bash
# เปิด INDEX.md → Ctrl+F ค้นหา tag / ชื่อ skill
```

### วิธี 2: ค้นหา by Tag
ใน INDEX.md มีตาราง "🏷️ ค้นหาตาม Tag" — link ตรงๆ ไปยัง skill

### วิธี 3: Browse Folder
```bash
ls skills/
# แล้วเปิด SKILL.md ไฟล์ที่ต้องการ
```

---

## 📞 Support & Questions

- **สอบถามเกี่ยว skill** → เปิด Issue ใน GitHub
- **มีไอเดีย skill ใหม่** → สร้าง Discussion
- **ต้องการ access** → ติดต่อ team admin

---

## 📊 Statistics

| | จำนวน |
|-|-------|
| Skills | 2 |
| Plugins | 1 |
| Contributors | 2+ |
| Last Updated | 2025-06-26 |

---

## 🔗 Quick Links

- [Master Index](./INDEX.md) — ค้นหา skills/plugins ทั้งหมด
- [Skills Guide](./skills/README.md) — วิธีสร้าง skill
- [Plugins Guide](./plugins/README.md) — วิธีสร้าง plugin
- [Create Guide Skill](./skills/create-guide/SKILL.md) — Full documentation

---

## 📝 License & Attribution

- **Repository**: Anthropic + Team
- **Skills**: Each skill has its own author field in SKILL.md frontmatter
- **Contributing**: Pull requests welcome — follow the workflow above

---

**Last Updated**: 2025-06-26  
**Maintained By**: Team  
**Status**: ✅ Active
