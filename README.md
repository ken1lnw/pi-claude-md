# 🧠 PI Claude Md

ที่เก็บกลางของทีมสำหรับ Claude Skills, Plugins, และเอกสาร

## 📁 โครงสร้าง

| โฟลเดอร์ | ใช้เก็บอะไร |
|----------|------------|
| `skills/` | Claude Skill files (.md) |
| `plugins/` | Plugins และ Tools |
| `docs/` | เอกสารทั่วไป |

## ⚡ เริ่มต้นใช้งาน

```bash
git clone https://github.com/ken1lnw/pi-claude-md.git
cd pi-claude-md
claude .
```

Claude Code จะอ่าน `CLAUDE.md` อัตโนมัติ — ไม่ต้อง install อะไรเพิ่ม

## 📖 ค้นหา Skills & Plugins

เปิด [INDEX.md](./INDEX.md) แล้ว `Ctrl+F` หา tag หรือชื่อที่ต้องการ

## 📤 เพิ่ม Skill ใหม่

```bash
git checkout -b feature/ชื่อ-skill-ของคุณ
mkdir -p skills/ชื่อ-skill
cp skills/example-skill/SKILL.md skills/ชื่อ-skill/
# แก้ไข SKILL.md และอัปเดต INDEX.md
git add .
git commit -m "feat: เพิ่ม skill ชื่อ-skill"
git push origin feature/ชื่อ-skill-ของคุณ
# เปิด Pull Request บน GitHub
```

## 🔗 เพิ่ม Skills นี้เข้า Claude Code (Global)

ถ้าต้องการให้ skills ใช้ได้ทุกโปรเจค ไม่ใช่แค่ใน repo นี้:

**วิธีที่ 1 — Import ใน global CLAUDE.md**

```bash
# เพิ่มบรรทัดนี้ใน ~/.claude/CLAUDE.md
echo "@~/github/pi-claude-md/CLAUDE.md" >> ~/.claude/CLAUDE.md
```

Claude Code จะโหลด skills จาก repo นี้ในทุก session อัตโนมัติ

**วิธีที่ 2 — Symlink skill เดี่ยวๆ**

```bash
# ถ้าต้องการแค่ skill ใดก็ตาม
ln -s ~/github/pi-claude-md/skills/<ชื่อ-skill>/SKILL.md ~/.claude/skills/<ชื่อ-skill>.md
```

## 📥 Pull อัปเดตล่าสุด

```bash
git checkout main
git pull origin main
```
