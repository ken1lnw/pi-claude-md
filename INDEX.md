# 📖 PI Claude Md — Index

ค้นหา Skills และ Plugins ทั้งหมดได้ที่นี่

---

## 🧠 Skills

| ชื่อ | คำอธิบาย | Tags | Author | Status | อัปเดตล่าสุด |
|------|----------|------|--------|--------|-------------|
| [Create Guide](./skills/create-guide/SKILL.md) | สร้างคู่มือ HTML แบบ standalone สำหรับโมดูล | `documentation` `guide` `html` `capture` `bilingual` | Anthropic | ✅ stable | 2025-06-26 |
| [Example Skill](./skills/example-skill/SKILL.md) | ตัวอย่าง skill template | `example` `template` | your-name | ✅ stable | 2025-01-01 |

---

## 🌍 External Skills (ติดตั้งผ่าน Marketplace — ไม่เก็บไฟล์ใน repo)

| ชุด | มีอะไร | ที่มา | License | ติดตั้ง |
|-----|--------|-------|---------|---------|
| [.NET / Akka / Aspire](./docs/external-skills.md) | 35 skills — C#, Akka.NET, Aspire, EF Core, testing, OpenTelemetry | [Aaronontheweb/dotnet-skills](https://github.com/Aaronontheweb/dotnet-skills) | MIT | `/plugin marketplace add Aaronontheweb/dotnet-skills` |

> **ห้าม copy ไฟล์ skill ของทีมอื่นเข้า repo นี้** — repo เป็น public และไม่มี LICENSE
> ใช้ marketplace แทน แล้วบันทึกไว้ใน [`docs/external-skills.md`](./docs/external-skills.md)

---

## 🔌 Plugins

| ชื่อ | คำอธิบาย | Author | Status | อัปเดตล่าสุด |
|------|----------|--------|--------|-------------|
| [Example Plugin](./plugins/example-plugin/README.md) | ตัวอย่าง plugin structure | your-name | ✅ stable | 2025-01-01 |

---

## 📄 Docs

| ชื่อ | คำอธิบาย |
|------|----------|
| [Skills Guide](./skills/README.md) | วิธีสร้าง skill (+ 2 เรื่องที่พลาดบ่อยจน skill ไม่โหลด) |
| [Plugins Guide](./plugins/README.md) | วิธีสร้าง plugin |
| [External Skills](./docs/external-skills.md) | skills ของทีมอื่น — ติดตั้งผ่าน marketplace แทนการ copy ไฟล์ |

---

## 🏷️ ค้นหาตาม Tag

| Tag | Skills / Plugins |
|-----|-----------------|
| `documentation` | [Create Guide](./skills/create-guide/SKILL.md) |
| `guide` | [Create Guide](./skills/create-guide/SKILL.md) |
| `html` | [Create Guide](./skills/create-guide/SKILL.md) |
| `capture` | [Create Guide](./skills/create-guide/SKILL.md) |
| `bilingual` | [Create Guide](./skills/create-guide/SKILL.md) |
| `example` | [Example Skill](./skills/example-skill/SKILL.md), [Example Plugin](./plugins/example-plugin/README.md) |
| `template` | [Example Skill](./skills/example-skill/SKILL.md), [Example Plugin](./plugins/example-plugin/README.md) |

---

## 📊 สถิติ

| | จำนวน |
|-|-------|
| Skills ใน repo นี้ | 2 |
| External skills (marketplace) | 35 |
| Plugins ทั้งหมด | 1 |
| Contributors | 2 |

---

## 💡 วิธีใช้ Index ในชีวิตประจำวัน

**เมื่อสร้าง skill ใหม่** — เพิ่ม 1 แถวใน INDEX.md แค่นั้น:

```markdown
| [ชื่อ Skill](./skills/ชื่อโฟลเดอร์/SKILL.md) | คำอธิบาย | `tag1` `tag2` | ชื่อคุณ | ✅ stable | วันที่ |
```

**เมื่ออยากค้นหา** — เปิด INDEX.md แล้ว `Ctrl+F` หา tag หรือชื่อที่ต้องการได้เลย ไม่ต้องเปิดทีละโฟลเดอร์

---

> ⚠️ **สำหรับสมาชิกทีม:** ทุกครั้งที่เพิ่ม Skill หรือ Plugin ใหม่ **ต้องอัปเดต INDEX.md ด้วยทุกครั้ง**
