# 🔌 Plugins

รวม Plugins และ Tools ของทีม

## วิธีเพิ่ม Plugin ใหม่

1. สร้างโฟลเดอร์: `plugins/ชื่อ-plugin/`
2. สร้างไฟล์ `plugin.json` และ `README.md`
3. อัปเดต `INDEX.md` ที่ root

## โครงสร้าง plugin.json

```json
{
  "name": "ชื่อ plugin",
  "version": "1.0.0",
  "description": "อธิบาย plugin",
  "author": "ชื่อคุณ",
  "tags": ["tag1", "tag2"],
  "status": "stable",
  "created_at": "YYYY-MM-DD"
}
```
