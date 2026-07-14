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

ถ้าต้องการให้ skills ใช้ได้ทุกโปรเจค ไม่ใช่แค่ใน repo นี้ — **symlink ทั้งโฟลเดอร์**:

```bash
# skill เดียว
ln -s ~/github/pi-claude-md/skills/<ชื่อ-skill> ~/.claude/skills/<ชื่อ-skill>

# หรือทุกตัวรวดเดียว
for s in ~/github/pi-claude-md/skills/*/; do
  ln -sfn "$s" ~/.claude/skills/"$(basename "$s")"
done
```

> ⚠️ **ต้อง symlink เป็นโฟลเดอร์** — Claude Code มองหา `~/.claude/skills/<ชื่อ>/SKILL.md`
> ถ้า symlink เป็นไฟล์เดี่ยว (`~/.claude/skills/<ชื่อ>.md`) **skill จะเงียบไปเลย ไม่โหลดและไม่ error**

**ตรวจว่าติดตั้งสำเร็จ:** เปิด Claude Code แล้วพิมพ์ `/` — ต้องเห็นชื่อ skill ในลิสต์

## 🌍 Skills จากภายนอก (ชุด .NET / Akka / Aspire)

**ห้าม copy ไฟล์ของทีมอื่นเข้า repo นี้** — ติดตั้งผ่าน marketplace แทน (ได้ของใหม่อัตโนมัติ ไม่มีปัญหา license):

```
/plugin marketplace add Aaronontheweb/dotnet-skills
```

รายชื่อ 35 skills + วิธีใช้: [`docs/external-skills.md`](./docs/external-skills.md)

## 📥 Pull อัปเดตล่าสุด

```bash
git checkout main
git pull origin main
```
