---
description: สร้างคู่มือการใช้งาน HTML — รวม 4 แหล่งข้อมูล (CLAUDE.md + Source Code + Routes + Screenshot Vision) รองรับหลายโปรเจค หลาย framework และ 2 ภาษา (th/en) พร้อม Playwright auto-capture และ auth แบบปลอดภัย
---

> **คำนำ — อ่านก่อนเริ่มทำงานทุกครั้ง**
>
> command นี้สร้างคู่มือการใช้งาน HTML แบบ standalone สำหรับ module ใดก็ได้ในโปรเจค
> โดยรวบรวมข้อมูลจาก **4 แหล่ง** พร้อมกัน ได้แก่ CLAUDE.md (สี/convention ของ module),
> Source Code (ปุ่ม/form/dialog ที่มีจริง), Route Structure (path และ menu label),
> และ Screenshot Vision (รูปหน้าจอจริงที่ capture ไว้)
>
> ผลลัพธ์คือ **ไฟล์ HTML ไฟล์เดียว** — รูปทุกรูป embed เป็น base64 แชร์ได้ทันที
> ไม่ต้องมี server ไม่ต้องติดตั้งอะไรเพิ่ม เปิดด้วย browser ได้เลย
>
> **โหมด `--capture`** จะให้ Playwright เข้าระบบและ screenshot ทุกหน้าให้อัตโนมัติ **2 viewport**
> — 🖥️ **PC** (1440×900) เพื่อเห็น layout เต็มจอ และ 📱 **Mobile** (390×844) เพื่อเห็นรายละเอียดชัดขึ้น
> ในคู่มือ HTML แต่ละหน้าจะมีปุ่ม `🖥️ PC` / `📱 Mobile` สลับดูได้ในคลิกเดียว
> credential จะไม่ถูกเก็บในโค้ด — ใช้ session token แทน เก็บที่ `~/docs/<projectName>/` นอก project directory
> ก่อน screenshot ทุกครั้ง script จะซ่อน **React Query DevTools / TanStack Query DevTools**
> โดยอัตโนมัติด้วย JavaScript inject เพื่อป้องกัน panel บัง form หรือเนื้อหาในหน้า
> ถ้าต้องการ PC เท่านั้น → เพิ่ม `--no-mobile`
>
> **ถ้าไม่ใส่ `--capture`** → ใช้รูปที่วางไว้ใน `~/docs/<projectName>/images/` แทน
> เหมาะกับกรณีที่ capture ไว้แล้ว หรือต้องการ update เนื้อหาโดยไม่ capture ใหม่

---

สร้างคู่มือการใช้งาน HTML สำหรับ module: **$ARGUMENTS**

**รูปแบบ argument:**
```
/create-guide <moduleName>                          → ภาษาไทยอย่างเดียว (default)
/create-guide <moduleName> en                       → ภาษาอังกฤษอย่างเดียว
/create-guide <moduleName> th+en                    → 2 ภาษา มี toggle ปุ่มสลับภาษาใน sidebar
/create-guide <moduleName> --capture                → ไทย + ถามแผน + Playwright auto-capture (PC + mobile)
/create-guide <moduleName> en --capture             → อังกฤษ + ถามแผน + auto-capture (PC + mobile)
/create-guide <moduleName> th+en --capture          → 2 ภาษา + toggle + ถามแผน + auto-capture (PC + mobile)
/create-guide <moduleName> --capture --no-mobile    → capture เฉพาะ PC (ข้าม mobile viewport)
/create-guide <moduleName> --capture --replan       → capture ใหม่ + ถามแผนใหม่ทั้งหมด
/create-guide <moduleName> --capture --recapture    → capture รูปใหม่ทุกรูป (ไม่ใช้ cache รูป)
/create-guide <moduleName> --fresh                  → ล้าง cache ทั้งหมด + capture ใหม่ทุกอย่าง
/create-guide <moduleName> --refresh-code           → อ่าน source code ใหม่ทุก slug
```

**Skip flags — ใช้ลด token เมื่อต้องการข้ามขั้นตอนบางส่วน:**
```
── ข้าม capture steps ──
--no-plan          → ข้ามการถามแผน ใช้ extraCaptures เดิม (หรือ overview only ถ้าไม่มี)
--overview-only    → capture overview PC+mobile เท่านั้น ไม่ capture dialog/extra ใดๆ

── ข้าม source reading ──
--no-code          → ข้ามการ Glob + อ่าน source code ทุก slug (ใช้ cache เดิม หรือ routes only)
--no-vision        → ข้าม vision analysis รูปทั้งหมด ใช้ filename pattern detect type แทน

── ข้ามเกือบทุกอย่าง ──
--html-only        → สร้าง HTML จาก cache ที่มีอยู่เท่านั้น ข้าม auth + capture + sources ทั้งหมด
                     (ถ้า cache ขาด → แจ้งและดำเนินการต่อด้วยข้อมูลที่มี)

── shorthand รวม ──
--lite             → ย่อ: --no-plan --overview-only --no-vision  (token น้อยที่สุด ยังคง capture)
```

**ตาราง token ที่ประหยัดได้ (โดยประมาณ):**
| flag | ข้ามขั้นตอน | token ที่ประหยัด |
|---|---|---|
| `--no-plan` | ถามแผน capture | ~500–1,000 |
| `--no-code` | Glob + read source code | ~2,000–5,000 |
| `--no-vision` | vision analysis รูป (ต่อรูป) | ~800–2,000 ต่อรูป |
| `--overview-only` | extra capture steps | ~200–500 |
| `--html-only` | auth + capture + 4 sources | ~80–95% ของทั้งหมด |
| `--lite` | plan + extra capture + vision | ~60–80% |

---

## ขั้นตอนที่ 0 — Parse Arguments + ตรวจสอบโปรเจค

### 0-A  Parse $ARGUMENTS
```
parts        = trim("$ARGUMENTS").split(/\s+/)
moduleName   = parts[0]                          # เช่น "basket-circulation"
langMode     = parts.find(p => ["th","en","th+en"].includes(p)) ?? "th"

# ── capture flags ──────────────────────────────────────────────
captureMode  = parts.includes("--capture")       # true | false
mobileMode   = !parts.includes("--no-mobile")   # false = PC only
replanMode   = parts.includes("--replan")        # ถามแผนใหม่ทั้งหมด
recaptureMode= parts.includes("--recapture")     # capture รูปใหม่ทุกรูป

# ── skip flags (ลด token) ──────────────────────────────────────
noPlanMode   = parts.includes("--no-plan")       # ข้ามการถามแผน
overviewOnly = parts.includes("--overview-only") # capture overview เท่านั้น
noCodeMode   = parts.includes("--no-code")       # ข้าม source code reading
noVisionMode = parts.includes("--no-vision")     # ข้าม vision analysis
htmlOnlyMode = parts.includes("--html-only")     # ข้ามทุกอย่าง สร้าง HTML จาก cache

# ── shorthand ─────────────────────────────────────────────────
if parts.includes("--lite"):
  noPlanMode = overviewOnly = noVisionMode = true  # ขยาย --lite เป็น 3 flags

# ── html-only implies no-capture ──────────────────────────────
if htmlOnlyMode:
  captureMode = false   # ไม่มีประโยชน์ capture เพราะต้องการแค่ HTML
```

**Decision tree ก่อนเริ่มงาน:**
```
htmlOnlyMode = true  → ข้าม Step 0-D, 0-E, 0-F, 1 ทั้งหมด → ไป Step 2 ทันที (ใช้ cache)
captureMode  = false → ข้าม Step 0-D, 0-E
noPlanMode   = true  → ข้าม Step 0-E.1 (planning dialog)
overviewOnly = true  → ใน capture loop: ข้าม extraCaptures ทุกหน้า
noCodeMode   = true  → ข้าม Source 3 (อ่านได้จาก cache ถ้ามี)
noVisionMode = true  → ข้าม Source 4 vision read (ใช้ filename-based type)
```

### 0-B  หา Project Name + Framework

อ่าน `package.json` (parallel กับ `composer.json` ถ้ามี):
- `projectName` = field `name`  (ถ้าไม่มี → ชื่อ folder ของ working directory)
- ตรวจ `dependencies` + `devDependencies` เพื่อ detect framework:

| dependency พบ | framework | routePattern |
|---|---|---|
| `"next"` | `nextjs` | `src/app/**/page.tsx`, `src/pages/**/*.tsx`, `src/constants/menu.ts` |
| `"nuxt"` / `"nuxt3"` | `nuxt` | `pages/**/*.vue`, `composables/`, `server/routes/` |
| `"vue"` + `"vue-router"` | `vue-spa` | `src/router/*.{js,ts}`, `src/views/**/*.vue` |
| `"@angular/core"` | `angular` | `src/app/**/*-routing.module.ts`, `src/app/**/*.routes.ts` |
| `"gatsby"` | `gatsby` | `src/pages/**/*.{tsx,jsx}` |
| ไม่ตรงใดเลย | `generic` | Glob `src/**/{page,route,screen,view,Page}*.{tsx,vue,jsx,py}` |

### 0-C  Docs Root

```
docsRoot      = ~/docs/<projectName>/
imagesDir     = ~/docs/<projectName>/images/
configFile    = ~/docs/<projectName>/guide.config.json
authFile      = ~/docs/<projectName>/auth.json        ← session เท่านั้น ไม่มี password
captureEnvFile = ~/docs/<projectName>/capture.env     ← CAPTURE_USER/CAPTURE_PASS นอก project dir
```

สร้าง folder ถ้ายังไม่มี:
```powershell
New-Item -ItemType Directory -Force "$env:USERPROFILE\docs\<projectName>\images"
```

---

## ขั้นตอนที่ 0-D — Auth Setup  *(รันเฉพาะเมื่อ captureMode = true)*

> ถ้า `captureMode = false` → ข้ามขั้นตอนนี้ทั้งหมด ไปที่ขั้นตอนที่ 1 เลย
>
> ถ้า `htmlOnlyMode = true` → ข้าม Step 0-D, 0-E, 0-F ทั้งหมด ข้ามไป **Step 1** ทันที
> (source 1-4 จะอ่านจาก cache เท่านั้น ถ้า cache ขาด → แจ้ง user แต่ดำเนินการต่อด้วยข้อมูลที่มี)
>
> ```
> ⚡ [--html-only] ข้ามทุก capture/source step → ใช้ cache ที่มีอยู่สร้าง HTML
>    guide.config.json พบ: meta✅  routes✅  pages(3/4)✅  images(8 ไฟล์)✅
> ```

ตรวจสอบตามลำดับ **waterfall** นี้:

```
1. auth.json มีอยู่ + อายุ < 8 ชั่วโมง
       → ✅ ใช้ session เดิม ข้ามไป Step 0-E

2. ~/docs/<projectName>/capture.env มี CAPTURE_USER + CAPTURE_PASS
       → ✅ อ่าน credential จาก capture.env ไป login
          (ไฟล์นี้อยู่นอก project dir — ไม่เสี่ยง commit ขึ้น Git)

3. ไม่มีทั้งคู่
       → 🔐 ถามผู้ใช้ใน terminal (ดูด้านล่าง)
```

### กรณีที่ 3 — Prompt ใน terminal

แสดงข้อความนี้แล้วรอ input:

```
╔══════════════════════════════════════════════════════╗
║  🔐  Login เพื่อ capture หน้าจอ                      ║
║      URL: <baseUrl จาก guide.config.json>            ║
╚══════════════════════════════════════════════════════╝

  Username / Email : 
  Password         : (ตัวอักษรจะถูกซ่อน)

  [Enter] ดำเนินการต่อ   [S + Enter] บันทึกลง ~/docs/<projectName>/capture.env ด้วย
```

- password ต้องใช้ raw mode ซ่อนตัวอักษรขณะพิมพ์ (แสดงแค่ `*` หรือว่างเปล่า)
- ถ้าผู้ใช้กด `S` → บันทึก `CAPTURE_USER` / `CAPTURE_PASS` ลง `~/docs/<projectName>/capture.env`
  แล้วแจ้งว่า **"บันทึก capture.env ไว้ที่ ~/docs/<projectName>/ แล้ว (นอก project — ปลอดภัย)"**
- ไม่ต้องแก้ `.gitignore` เพราะไฟล์อยู่นอก project directory

### 0-D.1  Login ด้วย Playwright

```typescript
// scripts/capture-setup.ts
import { chromium } from 'playwright'
import * as fs from 'fs'
import * as path from 'path'
import * as readline from 'readline'

const AUTH_PATH        = path.join(process.env.HOME!, 'docs', PROJECT_NAME, 'auth.json')
const CAPTURE_ENV_PATH = path.join(process.env.HOME!, 'docs', PROJECT_NAME, 'capture.env')

async function getCredentials(): Promise<{ user: string; pass: string } | null> {
  // กรณี 1 — session ยังใช้ได้
  if (fs.existsSync(AUTH_PATH)) {
    const ageHours = (Date.now() - fs.statSync(AUTH_PATH).mtimeMs) / 3_600_000
    if (ageHours < 8) {
      console.log(`✅ ใช้ session เดิม (อายุ ${ageHours.toFixed(1)} ชม.)`)
      return null  // ไม่ต้อง login
    }
  }

  // กรณี 2 — ~/docs/<project>/capture.env (นอก project dir — ปลอดภัย)
  if (fs.existsSync(CAPTURE_ENV_PATH)) {
    const env = parseEnv(fs.readFileSync(CAPTURE_ENV_PATH, 'utf8'))
    if (env.CAPTURE_USER && env.CAPTURE_PASS) {
      console.log(`✅ อ่าน credential จาก ${CAPTURE_ENV_PATH}`)
      return { user: env.CAPTURE_USER, pass: env.CAPTURE_PASS }
    }
  }

  // กรณี 3 — ถามใน terminal
  return promptCredentials()
}

async function promptCredentials(): Promise<{ user: string; pass: string }> {
  const rl = readline.createInterface({ input: process.stdin, output: process.stdout })
  const ask = (q: string) => new Promise<string>(res => rl.question(q, res))

  console.log('\n╔══════════════════════════════════════════════════════╗')
  console.log('║  🔐  Login เพื่อ capture หน้าจอ                      ║')
  console.log(`║      URL: ${BASE_URL.padEnd(42)}║`)
  console.log('╚══════════════════════════════════════════════════════╝\n')

  const user = await ask('  Username / Email : ')
  process.stdout.write('  Password         : ')
  const pass = await hiddenInput()
  const save = await ask(`\n  บันทึกลง ${CAPTURE_ENV_PATH} ด้วยไหม? (y/N) : `)

  rl.close()

  if (save.toLowerCase() === 'y') {
    fs.mkdirSync(path.dirname(CAPTURE_ENV_PATH), { recursive: true })
    fs.writeFileSync(CAPTURE_ENV_PATH, `CAPTURE_USER=${user}\nCAPTURE_PASS=${pass}\n`)
    console.log(`  💾 บันทึกลง ${CAPTURE_ENV_PATH} แล้ว (นอก project — ปลอดภัย)\n`)
  }

  return { user, pass }
}

function hiddenInput(): Promise<string> {
  return new Promise(resolve => {
    const chars: string[] = []
    process.stdin.setRawMode(true)
    process.stdin.resume()
    const handler = (buf: Buffer) => {
      const c = buf.toString()
      if (c === '\r' || c === '\n') {
        process.stdin.setRawMode(false)
        process.stdin.pause()
        process.stdin.removeListener('data', handler)
        resolve(chars.join(''))
      } else if (c === '\x7f') {
        chars.pop()
      } else if (c !== '\x03') {
        chars.push(c)
      }
    }
    process.stdin.on('data', handler)
  })
}

export async function setupAuth(baseUrl: string) {
  const creds = await getCredentials()
  if (!creds) return  // session ยังดีอยู่

  const browser = await chromium.launch({ headless: true })
  const context = await browser.newContext()
  const page    = await context.newPage()

  console.log('  🌐 กำลัง login...')
  await page.goto(`${baseUrl}/login`)

  // ปรับ selector ให้ตรงกับระบบ
  await page.fill('input[type="email"], input[name="username"], input[name="email"]', creds.user)
  await page.fill('input[type="password"]', creds.pass)
  await page.click('button[type="submit"], button:has-text("Login"), button:has-text("Sign in")')

  try {
    await page.waitForURL(url => !url.toString().includes('/login'), { timeout: 15_000 })
    console.log('  ✅ Login สำเร็จ\n')
  } catch {
    await browser.close()
    throw new Error('❌ Login ไม่สำเร็จ — ตรวจสอบ username/password แล้วลองใหม่')
  }

  // บันทึก session (cookie เท่านั้น — ไม่มี password)
  fs.mkdirSync(path.dirname(AUTH_PATH), { recursive: true })
  await context.storageState({ path: AUTH_PATH })
  console.log(`  💾 บันทึก session → ${AUTH_PATH}\n`)

  await browser.close()
}
```

### 0-D.2  ไฟล์ที่ไม่ควร Commit (ตรวจสอบและเพิ่มใน .gitignore อัตโนมัติ)

| ไฟล์ | เหตุผล |
|---|---|
| `~/docs/*/capture.env` | มี CAPTURE_USER/CAPTURE_PASS — อยู่นอก project dir อยู่แล้ว ไม่เสี่ยง commit |
| `~/docs/*/auth.json` | มี session token — อยู่นอก project dir อยู่แล้ว ไม่เสี่ยง commit |
| `~/docs/*/images/` | รูป screenshot (optional) |

---

## ขั้นตอนที่ 0-E — Auto Capture  *(รันเฉพาะเมื่อ captureMode = true)*

> ถ้า `captureMode = false` → ข้ามขั้นตอนนี้ ไปที่ขั้นตอนที่ 1

> ⚠️ **คำเตือนด้านความปลอดภัย — แจ้งผู้ใช้ก่อน capture ทุกครั้ง:**
> รูปภาพที่ capture อาจมีข้อมูลจริงของผู้ใช้ปรากฏอยู่ (ชื่อ, รหัสออเดอร์, ยอดเงิน ฯลฯ)
> หากต้องการสร้างคู่มือเพื่อแชร์ภายนอก → ใช้ข้อมูล demo/staging แทน production
> HTML output ที่สร้างจะ embed รูปทั้งหมดเป็น base64 — การแชร์ไฟล์ HTML = แชร์รูปทั้งหมดด้วย

### 0-E.1 — Interactive Planning (ถามก่อน capture เสมอ)

> **ข้าม step นี้ถ้า:**
> - `noPlanMode = true` (`--no-plan`) → ใช้ `extraCaptures` ใน `guide.config.json` ที่มีอยู่
>   ถ้าหน้าใดยังไม่มี `extraCaptures` → capture overview only สำหรับหน้านั้น
>   แสดงข้อความ: `⏭️  [--no-plan] ข้ามการวางแผน — ใช้ extraCaptures เดิม (bc-domains✅ bc-trade⬜→overview)`
> - `overviewOnly = true` (`--overview-only`) → ข้ามทันที ไม่ถาม ไม่ใช้ extraCaptures ใด ๆ ทั้งสิ้น
>   แสดง: `⏭️  [--overview-only] ข้ามการวางแผน — capture overview PC+📱 เท่านั้น`
> - `replanMode = false` + ทุกหน้ามี `extraCaptures` อยู่แล้ว → ใช้ค่าเดิม ไม่ถามซ้ำ

ก่อนเริ่ม capture ให้ scan source code ของแต่ละหน้าใน module แล้ว **ถามผู้ใช้ทีละหน้า** ว่า dialog/ปุ่มไหนควรเปิดเพื่อถ่ายรูป

แสดงผลในรูปแบบนี้และรอคำตอบ:

```
╔══════════════════════════════════════════════════════════════════╗
║  🗂️  วางแผน capture สำหรับ: basket-circulation                  ║
║      พบ 4 หน้า — กรุณาตอบทีละหน้า                               ║
╚══════════════════════════════════════════════════════════════════╝

─────────────────────────────────────────
📄 หน้า 1/4: Domains  (/basket-circulation/domains)
─────────────────────────────────────────
พบ interactive elements ดังนี้:

  Dialogs / Sheets ที่เปิดได้:
    [A] "Add Domain"     → Sheet เพิ่ม Domain ใหม่        (trigger: button "Add Domain")
    [B] "Update Quota"   → Dialog แก้ไข Quota             (trigger: button "Edit" ในแถว)
    [C] "Domain Detail"  → Sheet รายละเอียด Domain        (trigger: button "Details" ในแถว)

  ปุ่มอื่น ๆ ที่พบ:
    [D] "Delete"         → ⚠️  submit/delete — ไม่แนะนำ capture
    [E] "Save"           → ⚠️  submit form — ไม่แนะนำ capture
    [F] filter dropdown  → dropdown เลือก filter (ดูได้ปลอดภัย)

เลือก elements ที่ต้องการ capture (เช่น: A C F)
หรือ [S] ข้ามหน้านี้ / [ALL-SAFE] เลือกทุกตัวที่ไม่ใช่ submit :
```

**กฎการจัดกลุ่ม** (scan จาก source code + label):

| สัญญาณที่พบใน code | จัดเป็น | แนะนำ |
|---|---|---|
| `onClick` → open Sheet/Dialog state | `nav-dialog` | ✅ capture ได้ |
| `href` / `router.push` | `nav-page` | ✅ capture ได้ |
| `mutation.mutate` / `form.handleSubmit` | `submit` | ⚠️ ไม่แนะนำ |
| `onDelete` / `method: DELETE` | `delete` | ⚠️ ไม่แนะนำ |
| dropdown/select ที่ไม่ submit | `nav-filter` | ✅ capture ได้ |
| tooltip / hover state | `nav-hover` | ✅ capture ได้ |

หลังผู้ใช้ตอบแต่ละหน้า → บันทึกเป็น `extraCaptures` ลง `guide.config.json` อัตโนมัติ แล้วถามหน้าถัดไป

เมื่อตอบครบทุกหน้าแล้ว แสดงสรุปก่อน capture จริง:

```
╔══════════════════════════════════════════════════════════════════╗
║  ✅  แผน capture สำหรับ basket-circulation                       ║
╚══════════════════════════════════════════════════════════════════╝

  bc-domains    overview (PC+📱) + [A] Add Domain sheet + [C] Domain Detail + [F] filter
  bc-basket     overview (PC+📱) เท่านั้น (ข้ามตามที่เลือก)
  bc-trade      overview (PC+📱) + [A] Scan dialog + [D] Status tooltip
  bc-void       overview (PC+📱) + [A] Void Reason dialog

  รวม: 4 หน้า / ~11 รูป PC + ~4 รูป mobile = ~15 รูป / ไม่มี submit action เลย ✅
  📱 mobile capture ทุกหน้า (390×844)  |  ปิดด้วย --no-mobile

[Enter] เริ่ม capture  |  [E] แก้ไขแผน :
```

> **สำคัญ:** ถ้า `guide.config.json` มี `extraCaptures` อยู่แล้วในหน้านั้น →
> ข้ามการถามและใช้ค่าที่มีอยู่ได้เลย ไม่ต้องถามซ้ำ
> เว้นแต่ผู้ใช้รัน `/create-guide <module> --capture --replan` → ถามใหม่ทั้งหมด

อ่าน `guide.config.json → modules[moduleName].routes` แล้ว capture แต่ละหน้า:

```typescript
// scripts/capture.ts
import { chromium } from 'playwright'
import * as path from 'path'
import * as fs from 'fs'

const AUTH_PATH = path.join(process.env.HOME!, 'docs', PROJECT_NAME, 'auth.json')
const IMG_DIR   = path.join(process.env.HOME!, 'docs', PROJECT_NAME, 'images')

// viewports ที่ใช้ capture — PC (1440×900) และ mobile (390×844 ≈ iPhone 14 Pro)
const VIEWPORTS = {
  pc:     { width: 1440, height: 900 },
  mobile: { width: 390,  height: 844 },
} as const

export async function captureModule(
  baseUrl: string,
  moduleName: string,
  routes: Array<{ slug: string; path: string; extraCaptures?: CaptureStep[] }>,
  options: { mobile?: boolean } = {}         // mobile=true by default (ปิดด้วย --no-mobile)
) {
  const withMobile = options.mobile !== false

  const browser = await chromium.launch({ headless: true })

  // PC context — overview + extra captures ทุกอย่าง
  const pcContext = await browser.newContext({
    storageState: AUTH_PATH,
    viewport: VIEWPORTS.pc,
  })
  const page = await pcContext.newPage()

  // Mobile context — overview อย่างเดียว (ถ้าไม่ได้ใส่ --no-mobile)
  const mobileContext = withMobile
    ? await browser.newContext({
        storageState: AUTH_PATH,
        viewport: VIEWPORTS.mobile,
        isMobile: true,
        hasTouch: true,
        deviceScaleFactor: 2,
      })
    : null
  const mobilePage = mobileContext ? await mobileContext.newPage() : null

  fs.mkdirSync(IMG_DIR, { recursive: true })

  // helper — ซ่อน React Query DevTools (และ dev panel อื่น ๆ) ก่อน screenshot ทุกครั้ง
  // ใช้ JS inject แทนการคลิก เพราะ DevTools panel อาจบัง element ที่ต้องการ
  async function hideDevTools(p = page) {
    await p.evaluate(() => {
      const DEVTOOL_SELECTORS = [
        // TanStack Query v5 (tsqd = TanStack Query Devtools)
        '.tsqd-parent-container',
        '.tsqd-main-panel',
        '.tsqd-open-btn-container',
        '#TanStackQueryDevtools',
        // React Query v4
        '.ReactQueryDevtools',
        '[class*="ReactQueryDevtools"]',
        '[data-testid="ReactQueryDevtoolsPanel"]',
        '[data-testid="ReactQueryDevtoolsToggleButton"]',
        '#react-query-devtools-portal',
        // React Query v5 data-testid
        '[data-testid="devtools-open-btn"]',
        '[data-testid="devtools-panel"]',
      ]
      DEVTOOL_SELECTORS.forEach(sel => {
        document.querySelectorAll<HTMLElement>(sel).forEach(el => {
          el.style.setProperty('display', 'none', 'important')
        })
      })
    })
  }

  for (const route of routes) {
    const pcOverviewPath     = path.join(IMG_DIR, `${route.slug}.png`)
    const mobileOverviewPath = path.join(IMG_DIR, `${route.slug}-mobile.png`)

    const pcSkip     = fs.existsSync(pcOverviewPath)     && !RECAPTURE_MODE
    const mobileSkip = fs.existsSync(mobileOverviewPath) && !RECAPTURE_MODE

    // ข้ามหน้าที่รูปครบทั้ง PC และ mobile แล้ว
    if (pcSkip && (!withMobile || mobileSkip) && !(route.extraCaptures?.length)) {
      console.log(`  ⏭️  [skip] ${route.slug} — รูปมีอยู่แล้ว (PC${withMobile ? '+mobile' : ''})`)
      continue
    }

    console.log(`  📸 Capturing ${route.slug}  (${route.path})`)

    // ─── PC overview ───────────────────────────────────────────────
    if (!pcSkip) {
      await page.goto(`${baseUrl}${route.path}`, { waitUntil: 'networkidle' })
      await page.waitForTimeout(800)           // รอ animation เสร็จ
      await hideDevTools()
      await page.screenshot({ path: pcOverviewPath, fullPage: false })
      console.log(`    ✅ overview (PC 1440×900) → ${route.slug}.png`)
    } else {
      console.log(`    ⏭️  skip PC overview — มีอยู่แล้ว`)
    }

    // ─── Mobile overview ──────────────────────────────────────────
    if (withMobile && mobilePage && !mobileSkip) {
      await mobilePage.goto(`${baseUrl}${route.path}`, { waitUntil: 'networkidle' })
      await mobilePage.waitForTimeout(800)
      await hideDevTools(mobilePage)
      await mobilePage.screenshot({ path: mobileOverviewPath, fullPage: false })
      console.log(`    ✅ overview (📱 390×844)  → ${route.slug}-mobile.png`)
    } else if (withMobile && mobileSkip) {
      console.log(`    ⏭️  skip mobile overview — มีอยู่แล้ว`)
    }

    // ─── Extra captures (PC เท่านั้น — dialog/zoom/btn/step) ─────
    // ข้ามทั้งหมดถ้า overviewOnly = true (--overview-only)
    if (OVERVIEW_ONLY) {
      if (route.extraCaptures?.length) {
        console.log(`    ⏭️  [--overview-only] ข้าม ${route.extraCaptures.length} extra step(s)`)
      }
    } else {
      // หน้าที่ PC skip แต่ต้องทำ extra → goto PC ก่อน
      if (route.extraCaptures?.length && pcSkip) {
        await page.goto(`${baseUrl}${route.path}`, { waitUntil: 'networkidle' })
        await page.waitForTimeout(800)
        await hideDevTools()
      }
      for (const step of route.extraCaptures ?? []) {
        await runCaptureStep(page, step, route.slug, IMG_DIR)
      }
    }
  }

  await pcContext.close()
  if (mobileContext) await mobileContext.close()
  await browser.close()
  console.log(`\n✅ Capture เสร็จทั้งหมด → ${IMG_DIR}\n`)
}

type CaptureStep =
  | { action: 'screenshot';         label: string }
  | { action: 'click';              selector: string }
  | { action: 'screenshot_element'; selector: string; label: string; padding?: number }
  | { action: 'wait';               ms: number }
  | { action: 'fill';               selector: string; value: string }
  | { action: 'hide_devtools' }
  // --- capture เฉพาะจุด ---
  | { action: 'highlight';   selector: string; label: string; color?: string; caption?: string; padding?: number }
  //    วาด box ล้อม element + label กำกับ แล้วถ่าย — ใช้อธิบายปุ่มสำคัญในคู่มือ
  | { action: 'hover';       selector: string; label: string }
  //    hover เพื่อให้ tooltip/dropdown โผล่ แล้วถ่ายทั้งหน้า — ใช้อธิบาย badge/status ที่มี tooltip
  | { action: 'wait_for';    selector: string; timeout?: number }
  //    รอ element ปรากฏก่อนทำ step ถัดไป — ใช้หลัง click ที่มี loading หรือ animation

async function runCaptureStep(page: any, step: CaptureStep, slug: string, dir: string) {
  switch (step.action) {

    case 'screenshot': {
      await hideDevTools()
      const p = path.join(dir, `${slug}-${step.label}.png`)
      await page.screenshot({ path: p, fullPage: false })
      console.log(`    ✅ ${step.label} → ${slug}-${step.label}.png`)
      break
    }

    case 'click':
      await page.click(step.selector)
      await page.waitForTimeout(400)
      break

    case 'screenshot_element': {
      await hideDevTools()
      const pad = step.padding ?? 0
      const el  = await page.$(step.selector)
      if (!el) { console.log(`    ⚠️  ไม่พบ element: ${step.selector}`); break }
      const box = await el.boundingBox()
      const p   = path.join(dir, `${slug}-${step.label}.png`)
      await page.screenshot({
        path: p,
        clip: { x: box.x - pad, y: box.y - pad, width: box.width + pad*2, height: box.height + pad*2 }
      })
      console.log(`    ✅ ${step.label} → ${slug}-${step.label}.png`)
      break
    }

    case 'wait':
      await page.waitForTimeout(step.ms)
      break

    case 'fill':
      await page.fill(step.selector, step.value)
      break

    case 'hide_devtools':
      await hideDevTools()
      console.log(`    🙈 ซ่อน DevTools แล้ว`)
      break

    // ─────────────────────────────────────────────────────────────
    // highlight — วาด box + label กำกับ element แล้วถ่าย
    //   ใช้: อธิบายปุ่มสำคัญ เช่น "ปุ่มนี้คือ Scan" ในคู่มือ
    // ─────────────────────────────────────────────────────────────
    case 'highlight': {
      await hideDevTools()
      const pad     = step.padding ?? 12
      const color   = step.color   ?? '#ef4444'   // แดง default
      const caption = step.caption ?? step.label

      const el = await page.$(step.selector)
      if (!el) { console.log(`    ⚠️  ไม่พบ element: ${step.selector}`); break }
      const box = await el.boundingBox()

      // inject overlay: box + label badge ด้านบน
      await page.evaluate(({ b, color, caption, pad }: any) => {
        const wrap = document.createElement('div')
        wrap.id    = '__guide_highlight__'
        wrap.style.cssText = `
          position:fixed; pointer-events:none; z-index:2147483647;
          left:${b.x - pad}px; top:${b.y - pad}px;
          width:${b.width + pad*2}px; height:${b.height + pad*2}px;
          outline: 3px solid ${color};
          border-radius: 6px;
          box-shadow: 0 0 0 2px white, 0 0 0 4px ${color};
        `
        const badge = document.createElement('div')
        badge.style.cssText = `
          position:absolute; top:-26px; left:0;
          background:${color}; color:white;
          font:700 12px/1 -apple-system,sans-serif;
          padding:4px 10px; border-radius:4px; white-space:nowrap;
        `
        badge.textContent = caption
        wrap.appendChild(badge)
        document.body.appendChild(wrap)
      }, { b: box, color, caption, pad })

      // ถ่ายแบบ crop รอบ element + พื้นที่ badge ด้านบน
      const p = path.join(dir, `${slug}-${step.label}.png`)
      await page.screenshot({
        path: p,
        clip: {
          x:      Math.max(0, box.x - pad - 20),
          y:      Math.max(0, box.y - pad - 40),  // เผื่อพื้นที่ badge
          width:  box.width  + pad*2 + 40,
          height: box.height + pad*2 + 60,
        }
      })

      // เอา overlay ออกหลังถ่ายเสร็จ
      await page.evaluate(() => {
        document.getElementById('__guide_highlight__')?.remove()
      })
      console.log(`    ✅ highlight → ${slug}-${step.label}.png  ("${caption}")`)
      break
    }

    // ─────────────────────────────────────────────────────────────
    // hover — hover element แล้วถ่ายทั้งหน้า (tooltip/dropdown โผล่)
    //   ใช้: อธิบาย badge status ที่มี tooltip, icon ที่มีคำอธิบาย
    // ─────────────────────────────────────────────────────────────
    case 'hover': {
      await page.hover(step.selector)
      await page.waitForTimeout(500)   // รอ tooltip animate เสร็จ
      await hideDevTools()
      const p = path.join(dir, `${slug}-${step.label}.png`)
      await page.screenshot({ path: p, fullPage: false })
      console.log(`    ✅ hover → ${slug}-${step.label}.png`)
      break
    }

    // ─────────────────────────────────────────────────────────────
    // wait_for — รอ selector ปรากฏใน DOM ก่อนทำ step ถัดไป
    //   ใช้: หลัง click ที่มี loading spinner, lazy render, animation
    // ─────────────────────────────────────────────────────────────
    case 'wait_for': {
      const timeout = step.timeout ?? 8_000
      await page.waitForSelector(step.selector, { state: 'visible', timeout })
      console.log(`    ⏳ wait_for พบ "${step.selector}"`)
      break
    }

  }
}
```

### กำหนด extraCaptures ใน `guide.config.json`

```json
{
  "baseUrl": "http://localhost:3000",
  "framework": "nextjs",
  "modules": {
    "basket-circulation": {
      "routes": [
        {
          "slug": "bc-domains",
          "path": "/basket-circulation/domains",
          "extraCaptures": [
            { "action": "screenshot_element",
              "selector": ".data-table-toolbar",
              "label": "zoom-toolbar",
              "padding": 8 },

            { "action": "highlight",
              "selector": "button:has-text('Add Domain')",
              "label": "btn-add-domain",
              "caption": "คลิกเพื่อเพิ่ม Domain ใหม่",
              "color": "#ef4444",
              "padding": 10 },

            { "action": "click",
              "selector": "button:has-text('Add Domain')" },
            { "action": "hide_devtools" },
            { "action": "screenshot", "label": "dialog-create" }
          ]
        },
        {
          "slug": "bc-trade",
          "path": "/basket-circulation/trade",
          "extraCaptures": [
            { "action": "highlight",
              "selector": "button:has-text('Scan')",
              "label": "btn-scan",
              "caption": "Scan QR เพื่อเริ่ม Trade",
              "color": "#2563eb",
              "padding": 8 },

            { "action": "hover",
              "selector": "[data-status='pending'] .status-badge",
              "label": "tooltip-status-pending" },

            { "action": "click",
              "selector": "button:has-text('Scan')" },
            { "action": "wait_for",
              "selector": ".scan-input",
              "timeout": 5000 },
            { "action": "hide_devtools" },
            { "action": "screenshot", "label": "scan-ready" }
          ]
        }
      ]
    }
  }
}
```

**naming convention ของรูปที่ได้:**

| action / รูปแบบ | ชื่อไฟล์ที่ได้ | ใช้ใน guide อย่างไร |
|---|---|---|
| overview PC | `bc-domains.png` | รูปหลัก PC (1440×900) ใน screenshot block |
| overview mobile | `bc-domains-mobile.png` | แท็บ 📱 mobile ใน screenshot block |
| `highlight` | `bc-trade-btn-scan.png` | `type:"btn"` → `<img class="img-inline">` ข้างคำอธิบายใน step |
| `hover` | `bc-trade-tooltip-status-pending.png` | `type:"zoom"` → `.img-zoom` อธิบาย tooltip ใน step |
| `wait_for` | *(ไม่สร้างรูป — แค่รอ)* | ใช้คู่กับ `screenshot` ถัดไป |

### แสดง progress ระหว่าง capture

```
🎬 เริ่ม capture module: basket-circulation
   URL: http://localhost:3000
   Session: ~/docs/wms/auth.json (อายุ 0.2 ชม.)
   Viewports: 🖥️ PC (1440×900) + 📱 mobile (390×844)

  ⏭️  [skip] bc-domains — รูปมีอยู่แล้ว (PC+mobile, 4 ไฟล์)
  ⏭️  [skip] bc-basket  — รูปมีอยู่แล้ว (PC+mobile, 2 ไฟล์)

  📸 [3/4] bc-trade     /basket-circulation/trade
    ✅ overview (PC 1440×900) → bc-trade.png
    ✅ overview (📱 390×844)  → bc-trade-mobile.png
    ✅ highlight              → bc-trade-btn-scan.png  ("Scan QR เพื่อเริ่ม Trade")
    ✅ hover                  → bc-trade-tooltip-status-pending.png
    ⏳ wait_for พบ ".scan-input"
    🙈 ซ่อน DevTools แล้ว
    ✅ scan-ready             → bc-trade-scan-ready.png

  📸 [4/4] bc-void      /basket-circulation/void
    ✅ overview (PC 1440×900) → bc-void.png
    ✅ overview (📱 390×844)  → bc-void-mobile.png

✅ Capture เสร็จ — ถ่ายใหม่ 2 หน้า (PC+mobile) / ข้าม 2 หน้า (มีอยู่แล้ว)
```

---

## ขั้นตอนที่ 0-F — ตรวจสอบ Cache ก่อนทำงาน

> **เป้าหมาย:** ข้ามงานที่ทำแล้ว ทำเฉพาะที่ขาด — ยกเว้น HTML ที่ต้องสร้างใหม่เสมอ

ตรวจสอบทั้งหมดนี้ก่อนเริ่ม Step 1:

```
cacheState = {
  claudeMd:     มีข้อมูลใน guide.config.json → modules[moduleName].meta แล้วหรือไม่
  routes:       มี routes ใน guide.config.json → modules[moduleName].routes แล้วหรือไม่
  sourceCode:   มีข้อมูลใน guide.config.json → modules[moduleName].pages[slug] แล้วหรือไม่  (ต่อ slug)
  images:       ไฟล์รูปมีอยู่ใน ~/docs/<projectName>/images/ แล้วหรือไม่               (ต่อ slug)
  html:         ❌ ไม่ cache — สร้างใหม่เสมอ
}
```

แสดงสรุป cache status ก่อนเริ่ม (รวม skip flags ที่ active):

```
⚡ Cache check — basket-circulation
   Active flags: --no-vision --overview-only

  Source 1  CLAUDE.md meta     ✅ cached → ข้าม
  Source 2  Routes             ✅ cached → ข้าม
  Source 3  Source code
              bc-domains       ✅ cached → ข้าม
              bc-basket        ✅ cached → ข้าม
              bc-trade         ❌ ไม่มี  → อ่านใหม่
              bc-void          ❌ ไม่มี  → อ่านใหม่
  Source 4  Screenshots (vision)
              bc-domains.png               ✅ มีแล้ว → ข้าม (PC)
              bc-domains-mobile.png        ✅ มีแล้ว → ข้าม (📱)
              bc-domains-dialog-create.png ✅ มีแล้ว → ข้าม
              bc-trade.png                 ❌ ไม่มี  → capture ใหม่ (PC)
              bc-trade-mobile.png          ❌ ไม่มี  → capture ใหม่ (📱)
              bc-void.png                  ❌ ไม่มี  → capture ใหม่ (PC)
              bc-void-mobile.png           ❌ ไม่มี  → capture ใหม่ (📱)
              vision analysis              ⏭️  [--no-vision] ข้าม — ใช้ filename pattern
  Capture   extra steps                   ⏭️  [--overview-only] ข้าม — overview only
  HTML      <moduleName>-guide.html       🔄 สร้างใหม่เสมอ

ประหยัดได้: ~75% token (cache 2 slug + CLAUDE.md + routes + ข้าม vision + ข้าม extra)
```

> ถ้ารัน `--html-only` → แสดงแค่:
> ```
> ⚡ [--html-only] ข้ามทุก source step — ใช้ cache ทั้งหมด
>    meta✅  routes✅  pages(4/4)✅  images(12 ไฟล์)✅  → สร้าง HTML ทันที
> ```

**กฎแต่ละ source:**

| Source | cache ที่ไหน | invalidate เมื่อไหร่ |
|---|---|---|
| CLAUDE.md meta | `guide.config.json → modules[mod].meta` | ไม่มี `meta` key / รัน `--refresh-meta` |
| Routes | `guide.config.json → modules[mod].routes` | ไม่มี `routes` key / รัน `--refresh-routes` |
| Source code (ต่อ slug) | `guide.config.json → modules[mod].pages[slug]` | ไม่มี `pages[slug]` key / รัน `--refresh-code` |
| รูปภาพ (ต่อไฟล์) | ไฟล์มีอยู่ใน `images/` | ไฟล์ไม่มี / รัน `--recapture` |
| vision cache (ต่อไฟล์) | `imageCache[filename]` ใน guide.config.json | ไม่มี key / รัน `--refresh-images` |
| HTML | — | **สร้างใหม่เสมอ** |

**`--refresh-*` flags เพิ่มเติม:**
```
--refresh-meta    → ล้าง cache CLAUDE.md แล้วอ่านใหม่
--refresh-routes  → ล้าง cache routes แล้วอ่านใหม่
--refresh-code    → ล้าง cache source code ทุก slug แล้วอ่านใหม่
--refresh-images  → ล้าง imageCache ทุก entry แล้ว vision-อ่านรูปใหม่ทั้งหมด
                    ใช้เมื่อ: วางรูปทับด้วยรูปใหม่ (ชื่อเดิม เนื้อหาใหม่)
                    หรือ: เพิ่มรูปเองแล้วต้องการให้ HTML สะท้อนเนื้อหาในรูป
--recapture       → capture รูปใหม่ทุกรูป (ไม่สนว่ามีอยู่แล้ว) — ต้องใช้กับ --capture
--fresh           → ล้าง cache ทั้งหมด (เทียบเท่ารัน flags ทั้ง 5 พร้อมกัน)
```

**โครงสร้าง cache ใน `guide.config.json`:**
```json
{
  "baseUrl": "http://localhost:3000",
  "framework": "nextjs",
  "modules": {
    "basket-circulation": {
      "meta": {
        "color": "green",
        "description": "ระบบหมุนเวียนตะกร้า",
        "conventions": { "badge": "สีเขียว=active", "button": "outline style" },
        "routePrefix": "/api/bc",
        "cachedAt": "2025-06-10T10:00:00Z"
      },
      "routes": [
        { "slug": "bc-domains", "path": "/basket-circulation/domains", "title": "Domains" }
      ],
      "pages": {
        "bc-domains": {
          "buttons":    [{ "label": "Add Domain", "type": "dialog" }],
          "formFields": ["Domain Name", "Basket Size", "Quota Limit"],
          "dialogs":    ["Create Domain", "Update Quota"],
          "description": "จัดการ Domain ที่เป็นเจ้าของตะกร้า",
          "cachedAt": "2025-06-10T10:00:00Z"
        },
        "bc-trade": null
      },
      "extraCaptures": { }
    }
  }
}
```

> `pages[slug] = null` หมายถึง slug นี้ยังไม่ได้ scan source code
> ถ้า key ไม่มีเลย → ถือว่า null เช่นกัน

---

## ขั้นตอนที่ 1 — รวบรวม 4 Sources พร้อมกัน (parallel)

อ่านเฉพาะ source ที่ยังไม่มี cache (ดู Step 0-F):

### Source 1 — CLAUDE.md
**ข้าม** ถ้า `guide.config.json → modules[moduleName].meta` มีข้อมูลอยู่แล้ว

อ่าน `CLAUDE.md` จาก working directory (ถ้ามี) แล้ว extract:
- `moduleColor` — สีประจำ module
- `moduleDescription` — คำอธิบาย module
- `conventions` — UI conventions
- `routePrefix` — API/route prefix

บันทึกลง `guide.config.json → modules[moduleName].meta` พร้อม `cachedAt`

### Source 2 — Route Structure
**ข้าม** ถ้า `guide.config.json → modules[moduleName].routes` มีข้อมูลอยู่แล้ว

อ่านพร้อมกันตาม `framework` ที่ detect ได้จาก Step 0-B:

**Fallback chain (ลองตามลำดับ หยุดเมื่อพบ route):**
```
1. ~/docs/<projectName>/guide.config.json   → modules[moduleName].routes
2. Framework-specific route file            → ตาม routePattern ของ framework
3. Glob src/**/<moduleName>*/**/*.{tsx,vue} → infer จาก folder + filename
4. ถ้ายังไม่พบ → แจ้ง user ระบุ path เอง
```

**Next.js เพิ่มเติม:** อ่าน `src/constants/menu.ts` (ถ้ามี) → หา array ชื่อ `<moduleName>SubMenu`
**Vue SPA เพิ่มเติม:** อ่าน `src/router/*.{js,ts}` → หา children routes ของ module

สร้าง `routeList` แล้วบันทึกลง `guide.config.json → modules[moduleName].routes`:
```
[{ slug, path, title, menuLabel }, ...]
```

### Source 3 — Source Code

> **ข้ามทั้ง Source 3 ถ้า `noCodeMode = true` (`--no-code`)**
> → ใช้ `pages[slug]` จาก cache ที่มีอยู่ใน `guide.config.json`
> → slug ที่ cache ขาด (`null` / key ไม่มี) → ใส่ `pages[slug] = {}` (empty, ไม่มี action-map/steps จาก code)
> → แสดง: `⏭️  [--no-code] ข้าม source code reading — cache: bc-domains✅ bc-trade❌(empty)`
> → ในคู่มือ HTML: หน้าที่ไม่มี code data → ไม่แสดง action-map แต่ยังแสดง screenshot + steps จาก vision

**ข้าม slug ที่** `guide.config.json → modules[moduleName].pages[slug]` มีข้อมูลอยู่แล้ว
**อ่านเฉพาะ slug ที่** `pages[slug]` เป็น `null` หรือ key ไม่มี

Glob และอ่านพร้อมกันเฉพาะ slug ที่ขาด:
- `src/features/$ARGUMENTS/<slug>/**/*.tsx`
- `src/app/**/<slug>*/**/page.tsx`

Extract สำหรับแต่ละ slug:
- **buttons** — label + type (`nav | submit | dialog | delete`) + target slug
- **formFields** — ชื่อ field ที่ user กรอก
- **dialogs** — Sheet/Dialog ที่เปิดในหน้านี้
- **description** — หน้านี้ทำอะไร

บันทึกลง `guide.config.json → modules[moduleName].pages[slug]` พร้อม `cachedAt`

### Source 4 — Screenshot Vision

> **ข้ามทั้ง Source 4 ถ้า `noVisionMode = true` (`--no-vision`)**
> → ไม่ส่งรูปใด ๆ ให้ Claude วิเคราะห์ ประหยัด token ได้มากที่สุด
> → ใช้ **filename pattern** ตัดสิน `imageType` แทน:
> ```
> filename pattern          → imageType
> ───────────────────────────────────────
> slug.png                  → "overview-pc"
> slug-mobile.png           → "overview-mobile"
> slug-dialog-*.png         → "dialog"
> slug-zoom-*.png           → "zoom"
> slug-btn-*.png            → "btn"
> slug-step-N.png           → "step"
> ```
> → `visibleText = ""` / `uiElements = []` / `pageState = "unknown"`
> → ในคู่มือ HTML: ยังฝัง screenshot ครบ แต่ caption + step text มาจาก source code เท่านั้น
> → แสดง: `⏭️  [--no-vision] ข้าม vision analysis — ใช้ filename-based type (8 รูป)`

**ข้าม รูปที่** มี `imageCache[filename]` ใน `guide.config.json` อยู่แล้ว
**อ่าน vision เฉพาะ รูปที่ไม่มี cache** (ไฟล์ใหม่ หรือถูก invalidate)

Glob รูปใน `~/docs/<projectName>/images/<moduleName>-*.{png,jpg,jpeg}` แล้วแบ่ง:
```
ไฟล์ใหม่ (ไม่มี imageCache key)   → อ่าน vision → extract imageType, visibleText, uiElements, pageState
ไฟล์มี imageCache อยู่แล้ว        → ใช้ข้อมูล cache ← ข้ามอ่าน vision
--refresh-images                   → ล้าง imageCache ทุก entry แล้วอ่าน vision ใหม่ทั้งหมด
```

> **กรณีที่ต้อง `--refresh-images`:**
> - วางรูปทับด้วยรูปใหม่ (ชื่อไฟล์เดิม เนื้อหาใหม่) เช่น capture มาใหม่แทนรูปเดิมที่ฟีเจอร์ยังไม่ครบ
> - เพิ่มรูปเองหลายไฟล์แล้วต้องการให้ caption/steps ใน HTML สะท้อนเนื้อหาในรูปจริง
>
> **ถ้าแค่เพิ่มไฟล์ใหม่ (ชื่อใหม่):** รัน `/create-guide <moduleName>` ปกติ ไม่ต้อง `--refresh-images`

บันทึก vision result ลง `guide.config.json → modules[moduleName].imageCache[filename]`

**Naming convention:**

| Pattern | ตัวอย่าง | ความหมาย |
|---|---|---|
| `slug.png` | `bc-domains.png` | Overview ฝั่ง PC (1440×900) — หลัก |
| `slug-mobile.png` | `bc-domains-mobile.png` | Overview ฝั่ง mobile (390×844) — แท็บ 📱 |
| `slug-2.png` | `bc-domains-2.png` | Secondary overview PC |
| `slug-dialog-xxx.png` | `bc-domains-dialog-create.png` | Dialog ที่เปิดในหน้า |
| `slug-zoom-xxx.png` | `bc-domains-zoom-toolbar.png` | Zoom เฉพาะส่วน |
| `slug-btn-xxx.png` | `bc-trade-btn-scan.png` | ปุ่มเฉพาะชิ้น |
| `slug-step-N.png` | `bc-trade-step-2.png` | ภาพขั้นตอนที่ N |

สำหรับแต่ละรูป → อ่านด้วย vision แล้ว extract imageType, visibleText, uiElements, pageState, slug, context

---

## ขั้นตอนที่ 2 — Page Match

สร้าง `pageMap` โดย merge ข้อมูลจากทุก source แล้วแสดงสรุป:

```
📋 Page Match สำหรับ module: $ARGUMENTS

  slug             path                              รูป
  ──────────────── ─────────────────────────────── ─────────────────────────────────────
  ✅ bc-domains    /basket-circulation/domains      overview✅  dialog✅  zoom✅
  ✅ bc-basket     /basket-circulation/basket       overview✅
  ⚠️  bc-trade     /basket-circulation/trade        overview✅  btn❌
  ✅ bc-void       /basket-circulation/void         overview✅  dialog✅

CLAUDE.md : ✅ พบ  (color: green, conventions: ✅)
Capture   : ✅ auto-captured ด้วย Playwright  (หรือ "📁 ใช้รูปจาก images/ ที่มีอยู่")
```

**กฎ synthesis:**
- ถ้า Source 4 (vision) ขัดแย้งกับ Source 3 (code) → เชื่อ Source 4
- ถ้าไม่มีรูป overview → ใช้รูปแรกที่มีแทน
- ถ้าไม่มีรูปเลยสักหน้า → แสดงรายการที่ต้องเตรียมแล้วหยุด

**กฎ langMode:**
| langMode | สร้าง TH | สร้าง EN | HTML output |
|---|---|---|---|
| `th` | ✅ | ❌ | ข้อความไทย ไม่มี toggle |
| `en` | ❌ | ✅ | ข้อความอังกฤษ ไม่มี toggle |
| `th+en` | ✅ | ✅ | มี toggle ค่าเริ่มต้นไทย |

**กฎการใช้รูป multi-image:**
- `overview` → รูปหลักบนสุดของ section
- `dialog` → แสดงใน step ที่เกี่ยวกับการเปิด dialog ด้วย `.img-zoom`
- `zoom` → แสดงใน step หรือ action-map ด้วย `.img-zoom`
- `btn` → แสดง inline ใน step ด้วย `<img class="img-inline">`
- `step-N` → แสดงใน step ที่ N ตรงกัน

---

## ขั้นตอนที่ 3 — Encode รูปเป็น Base64

```powershell
[Convert]::ToBase64String([IO.File]::ReadAllBytes("$env:USERPROFILE\docs\<projectName>\images\<filename>"))
```

เก็บเป็น `imageData = { "<slug>": "data:image/png;base64,<encoded>" }`

---

## ขั้นตอนที่ 4 — สร้าง HTML Guide

**ชื่อไฟล์ output:**
```
langMode = "th"    → ~/docs/<projectName>/<moduleName>-guide.html
langMode = "en"    → ~/docs/<projectName>/<moduleName>-guide-en.html
langMode = "th+en" → ~/docs/<projectName>/<moduleName>-guide-bilingual.html
```

HTML เป็น standalone — ทุก style อยู่ใน `<style>` เดียว รูปทั้งหมด embed base64

### โครงสร้าง HTML

Layout หลักเป็น **Sidebar + Main content**:

```html
<!DOCTYPE html>
<html lang="th"><!-- en หรือ th+en ให้ใส่ lang="th" ไว้ก่อน JS switchLang จะ handle -->
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title><!-- ชื่อ module --> — คู่มือการใช้งาน</title>
<!-- Thai font — โหลดเมื่อมี internet, fallback เป็น system Thai font เมื่อ offline -->
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Sarabun:wght@400;500;600;700;800&display=swap" rel="stylesheet">
</head>
<body>
<div class="app-layout">

  <!-- LEFT SIDEBAR (sticky, 260px) -->
  <nav class="sidebar">
    <div class="sidebar-logo">
      <div class="sidebar-logo-badge"><!-- projectName --></div>
      <div class="sidebar-logo-title"><!-- ชื่อ module --></div>
      <div class="sidebar-logo-sub">
        <!-- th    → คู่มือการใช้งาน -->
        <!-- en    → User Guide -->
        <!-- th+en → <span class="i18n" data-th="คู่มือการใช้งาน" data-en="User Guide"> -->
      </div>
    </div>

    <!-- Lang toggle — แสดงเฉพาะ langMode="th+en" -->
    <div class="lang-bar">
      <span class="lang-label i18n" data-th="ภาษา" data-en="Language">ภาษา</span>
      <div class="lang-switch">
        <button class="lang-btn active" data-lang="th" onclick="switchLang('th')">ไทย</button>
        <button class="lang-btn"        data-lang="en" onclick="switchLang('en')">EN</button>
      </div>
    </div>

    <div class="sidebar-nav">
      <div class="nav-group-label">
        <span class="i18n" data-th="หน้าหลัก" data-en="Overview">หน้าหลัก</span>
      </div>
      <a href="#top" class="nav-item" data-section="top">
        <span class="nav-item-icon">🗺️</span>
        <div>
          <div class="nav-item-label">
            <span class="i18n" data-th="ภาพรวมระบบ" data-en="System Overview">ภาพรวมระบบ</span>
          </div>
          <div class="nav-item-sub">
            <span class="i18n" data-th="สารบัญทั้งหมด" data-en="Table of Contents">สารบัญทั้งหมด</span>
          </div>
        </div>
      </a>
      <div class="nav-group-label">
        <span class="i18n" data-th="ฟังก์ชั่นหลัก" data-en="Features">ฟังก์ชั่นหลัก</span>
      </div>
      <!-- วน pageMap → nav-item 1 รายการต่อ slug -->
      <a href="#<sectionId>" class="nav-item" data-section="<sectionId>">
        <span class="nav-item-icon"><!-- emoji --></span>
        <div>
          <div class="nav-item-label"><!-- title ต้นฉบับ ไม่แปล --></div>
          <div class="nav-item-sub">
            <span class="i18n">
              <template data-lang="th"><!-- subtitleTH --></template>
              <template data-lang="en"><!-- subtitleEN --></template>
              <!-- subtitleTH -->
            </span>
          </div>
        </div>
      </a>
    </div>
  </nav>

  <!-- MAIN CONTENT -->
  <main class="main-content">
    <div class="wrapper">

      <!-- TOC -->
      <div class="toc" id="top">
        <div class="toc-title">
          <span class="i18n" data-th="สารบัญ" data-en="Table of Contents">สารบัญ</span>
        </div>
        <div class="toc-grid">
          <a href="#<sectionId>" class="toc-item">
            <div class="toc-num" style="background:<!-- color hex -->;">N</div>
            <div>
              <div class="toc-label"><!-- title --></div>
              <div class="toc-path"><!-- /route/path --></div>
              <div class="toc-desc">
              <span class="i18n">
                <template data-lang="th"><!-- tocDescTH --></template>
                <template data-lang="en"><!-- tocDescEN --></template>
                <!-- tocDescTH -->
              </span>
            </div>
            </div>
          </a>
        </div>
      </div>

      <!-- SECTIONS -->
      <div id="<sectionId>" class="guide-section">...</div>

    </div>
  </main>

</div>

<script>
(function(){
  /* Scroll spy */
  var sections = document.querySelectorAll('.guide-section[id], #top');
  var items    = document.querySelectorAll('.nav-item[data-section]');
  function setActive(id){
    items.forEach(function(el){ el.classList.toggle('active', el.dataset.section===id); });
    var a = document.querySelector('.nav-item[data-section="'+id+'"]');
    if(a) a.scrollIntoView({block:'nearest'});
  }
  var io = new IntersectionObserver(function(entries){
    entries.forEach(function(e){ if(e.isIntersecting) setActive(e.target.id); });
  },{threshold:0.15,rootMargin:'-5% 0px -65% 0px'});
  sections.forEach(function(s){ io.observe(s); });
  if(sections.length) setActive(sections[0].id);
  document.querySelectorAll('.nav-item[href]').forEach(function(a){
    a.addEventListener('click',function(e){
      e.preventDefault();
      var t=document.querySelector(a.getAttribute('href'));
      if(t) document.querySelector('.main-content').scrollTo({top:t.offsetTop-20,behavior:'smooth'});
    });
  });

  /* Lang switch — เฉพาะ langMode=th+en
   *
   * รูปแบบ HTML ที่รองรับ (ใช้ <template> แทน data-attribute):
   *
   *   <span class="i18n">
   *     <template data-lang="th"><strong>ข้อความ</strong>ไทย</template>
   *     <template data-lang="en"><strong>English</strong> text</template>
   *   </span>
   *
   * ข้อดี: HTML tags (<strong>, <a>, <code>, <br>) render ได้ถูกต้องทุกกรณี
   * เพราะ <template> เก็บ DOM จริง ไม่ใช่ string — ไม่มี entity escaping
   */
  window.switchLang = function(lang) {
    document.body.dataset.lang = lang;
    document.querySelectorAll('.i18n').forEach(function(el) {
      var tpl = el.querySelector('template[data-lang="' + lang + '"]');
      if (!tpl) return;
      // เก็บ template nodes ไว้ก่อน แล้วล้าง innerHTML แล้วใส่ clone กลับ + template เดิม
      var templates = Array.from(el.querySelectorAll('template'));
      el.innerHTML = '';
      templates.forEach(function(t) { el.appendChild(t); });
      el.insertBefore(tpl.content.cloneNode(true), el.firstChild);
    });
    document.querySelectorAll('.lang-btn').forEach(function(btn) {
      btn.classList.toggle('active', btn.dataset.lang === lang);
    });
    try { localStorage.setItem('guide-lang', lang); } catch(e) {}
  };
  var saved = (function(){ try{ return localStorage.getItem('guide-lang'); }catch(e){return null;} })();
  if(saved && saved !== 'th') window.switchLang(saved);

  /* Device tab — สลับ PC / Mobile screenshot ใน section เดียวกัน */
  window.showDevice = function(btn, device) {
    var block = btn.closest('.screenshot');
    block.querySelectorAll('.device-tab').forEach(function(b){ b.classList.toggle('active', b===btn); });
    block.querySelectorAll('.device-view').forEach(function(v){
      v.hidden = !v.classList.contains('device-'+device);
    });
  };
})();
</script>
</body>
```

### Section anatomy — 1 section ต่อ 1 slug

> **กฎการเขียน `i18n` span (langMode=th+en)**
>
> ใช้ `<template data-lang="...">` ภายใน `.i18n` เสมอ — ห้ามใช้ `data-th`/`data-en` attribute
> เพราะ HTML attribute จะ escape `<strong>` เป็น `&lt;strong&gt;` ทำให้ tag ไม่ render
>
> ```html
> <!-- ✅ ถูก — ใช้ <template> -->
> <span class="i18n">
>   <template data-lang="th">พื้นที่การผลิตที่ <strong>เป็นเจ้าของ</strong>ตะกร้า</template>
>   <template data-lang="en">Production area that <strong>owns</strong> baskets</template>
> </span>
>
> <!-- ❌ ผิด — data-attribute escape HTML -->
> <span class="i18n" data-th="พื้นที่ <strong>เป็นเจ้าของ</strong>" data-en="...">...</span>
> ```
>
> ค่าเริ่มต้น (ก่อน switchLang ถูกเรียก) ให้ใส่เนื้อหาภาษาไทยไว้นอก `<template>` โดยตรง
> JS จะ swap เนื้อหาจาก template ที่ตรงกับภาษาที่เลือกเมื่อ switchLang ถูกเรียก

```html
<section id="<sectionId>" class="guide-section">
  <div class="breadcrumb">
    <a href="#top">หน้าหลัก</a> › ชื่อ Module › ชื่อหน้า
  </div>
  <div class="section-header">
    <div class="section-icon c-<color>">🏭</div>
    <div>
      <div class="section-title">ชื่อหน้า</div>
      <div class="section-subtitle">คำอธิบายสั้น</div>
      <div class="section-url">/path · API: /api-prefix/...</div>
    </div>
  </div>

  <!-- section-intro: ใช้ <template> เพื่อให้ <strong>/<a>/<code> render ได้ -->
  <div class="section-intro">
    <span class="i18n">
      <template data-lang="th"><strong>Domain</strong> คือพื้นที่การผลิตที่เป็น &ldquo;เจ้าของ&rdquo; ตะกร้า ต้องตั้งค่าก่อนใช้งาน Trade</template>
      <template data-lang="en"><strong>Domain</strong> is a production area that owns baskets. Must be set up before using Trade.</template>
      <strong>Domain</strong> คือพื้นที่การผลิตที่เป็น "เจ้าของ" ตะกร้า ต้องตั้งค่าก่อนใช้งาน Trade
    </span>
  </div>

  <!--
    screenshot block — ถ้ามีทั้ง PC + mobile ให้ใช้รูปแบบ device-tabs
    ถ้ามีแค่ PC (--no-mobile หรือ capture เก่า) ให้ใช้ has-img ธรรมดา (ไม่มี device-tabs)

    กฎ hidden:
      - mobile div ต้องใช้ "hidden" attribute ใน HTML เท่านั้น
      - ห้ามใช้ CSS .device-mobile{display:none} เพราะจะทำให้ JS toggle ใช้ไม่ได้
      - showDevice() ใช้ v.hidden = true/false ซึ่งทำงานถูกต้องกับ hidden attribute
  -->
  <div class="screenshot has-img">
    <!-- device tabs — แสดงเฉพาะเมื่อมีรูป mobile (-mobile.png) -->
    <div class="device-tabs">
      <button class="device-tab active" onclick="showDevice(this,'pc')">🖥️ PC</button>
      <button class="device-tab"        onclick="showDevice(this,'mobile')">📱 Mobile</button>
    </div>
    <div class="device-view device-pc">
      <img src="data:image/png;base64,...PC_B64..." alt="ชื่อหน้า — PC">
    </div>
    <div class="device-view device-mobile" hidden><!-- ← hidden attribute เท่านั้น ห้ามใช้ CSS -->
      <img src="data:image/png;base64,...MOBILE_B64..." alt="ชื่อหน้า — Mobile"
           style="max-width:390px;margin:0 auto;display:block;">
    </div>
    <div class="screenshot-caption">
      <strong>ชื่อหน้าจอ</strong> —
      <span class="i18n">
        <template data-lang="th">รายการ Domain ทั้งหมด พร้อมปุ่ม Add Domain</template>
        <template data-lang="en">Full Domain list with Add Domain button</template>
        รายการ Domain ทั้งหมด พร้อมปุ่ม Add Domain
      </span>
    </div>
  </div>

  <div class="action-map">
    <div class="action-map-title">ปุ่มและการทำงานในหน้านี้</div>
    <div class="action-grid">
      <!-- action-card ตาม type: nav / submit / dialog / delete -->
    </div>
  </div>

  <div class="steps">
    <div class="step">
      <div class="step-num">1</div>
      <div class="step-body">
        <!-- step ที่มีแค่ text ธรรมดา -->
        <span class="i18n">
          <template data-lang="th">คลิกปุ่ม <strong>Add Domain</strong> ที่มุมขวาบน</template>
          <template data-lang="en">Click the <strong>Add Domain</strong> button at the top right</template>
          คลิกปุ่ม <strong>Add Domain</strong> ที่มุมขวาบน
        </span>
        <!-- ถ้ามีรูป dialog/zoom/step สำหรับ step นี้ -->
        <img class="img-zoom" src="data:image/png;base64,..." alt="dialog screenshot">
      </div>
    </div>
  </div>

  <div class="related">
    <span class="related-label">ดูเพิ่มเติม:</span>
    <a href="#<relatedSectionId>" class="related-chip">ชื่อหน้า</a>
  </div>
</section>
```

### CSS

```css
:root {
  --green:#16a34a; --green-light:#dcfce7;
  --blue:#2563eb;  --blue-light:#dbeafe;
  --red:#dc2626;   --red-light:#fee2e2;
  --amber:#d97706; --amber-light:#fef3c7;
  --gray:#64748b;  --gray-light:#f1f5f9;
  --gray-border:#e2e8f0;
  --primary:var(--blue); --primary-light:var(--blue-light);
}
* { box-sizing:border-box; margin:0; padding:0; }
html,body { height:100%; overflow:hidden; }
body { font-family:'Sarabun','Noto Sans Thai','Leelawadee UI','Leelawadee','Tahoma',
                   -apple-system,BlinkMacSystemFont,'Segoe UI',sans-serif;
       background:#f8fafc; color:#1e293b; line-height:1.6; display:flex; flex-direction:column; }
.app-layout { display:flex; flex:1; overflow:hidden; }
.sidebar {
  width:260px; min-width:260px; background:#1e293b; color:white;
  display:flex; flex-direction:column; height:100%; overflow-y:auto; flex-shrink:0;
}
.sidebar-logo { padding:20px 20px 16px; border-bottom:1px solid rgba(255,255,255,.1); }
.sidebar-logo-badge {
  display:inline-block; background:rgba(255,255,255,.12); border:1px solid rgba(255,255,255,.2);
  border-radius:6px; padding:3px 8px; font-size:10px; color:rgba(255,255,255,.7); margin-bottom:8px;
}
.sidebar-logo-title { font-size:14px; font-weight:700; color:white; line-height:1.3; }
.sidebar-logo-sub   { font-size:11px; color:rgba(255,255,255,.45); margin-top:3px; }
.sidebar-nav { padding:10px 0; flex:1; }
.nav-group-label {
  font-size:10px; font-weight:700; text-transform:uppercase; letter-spacing:.08em;
  color:rgba(255,255,255,.3); padding:10px 20px 4px;
}
.nav-item {
  display:flex; align-items:center; gap:10px; padding:9px 18px;
  font-size:13px; font-weight:500; color:rgba(255,255,255,.65);
  text-decoration:none; transition:background .15s,color .15s; border-left:3px solid transparent;
}
.nav-item:hover  { background:rgba(255,255,255,.07); color:white; }
.nav-item.active { background:rgba(255,255,255,.1); color:white; border-left-color:#4ade80; }
.nav-item-icon   { font-size:14px; width:20px; text-align:center; flex-shrink:0; }
.nav-item-label  { flex:1; }
.nav-item-sub    { font-size:10px; color:rgba(255,255,255,.35); margin-top:1px; }
.main-content { flex:1; overflow-y:auto; background:#f3f4f6; }
.wrapper { max-width:880px; }
.section-intro {
  background:white; border:1px solid var(--gray-border); border-radius:8px;
  padding:14px 18px; margin-bottom:20px; font-size:14px; color:#475569; line-height:1.75;
  border-left:4px solid #94a3b8;
}
.section-intro strong { color:#0f172a; }
.toc { background:white; padding:40px 48px; border-bottom:1px solid #e2e8f0; }
.toc-title {
  font-size:11px; font-weight:700; text-transform:uppercase;
  letter-spacing:.08em; color:#94a3b8; margin-bottom:20px;
}
.toc-grid { display:grid; grid-template-columns:1fr 1fr; gap:10px; }
.toc-item {
  display:flex; align-items:flex-start; gap:12px; padding:12px 14px;
  border-radius:8px; text-decoration:none; color:#334155;
  border:1px solid #e2e8f0; transition:background .15s;
}
.toc-item:hover  { background:var(--primary-light); border-color:var(--primary); }
.toc-num {
  width:28px; height:28px; border-radius:50%; background:var(--primary);
  color:white; display:flex; align-items:center; justify-content:center;
  font-weight:700; font-size:13px; flex-shrink:0; margin-top:2px;
}
.toc-label { font-size:14px; font-weight:600; }
.toc-path  { font-size:11px; color:#94a3b8; font-family:monospace; margin-top:2px; }
.toc-desc  { font-size:12px; color:#64748b; margin-top:4px; line-height:1.5; }
.guide-section {
  background:white; border-radius:12px; border:1px solid #e2e8f0;
  padding:32px; margin-bottom:32px;
}
.breadcrumb { font-size:12px; color:#94a3b8; margin-bottom:16px; }
.breadcrumb a { color:#64748b; text-decoration:none; }
.section-header { display:flex; align-items:flex-start; gap:16px; margin-bottom:24px; }
.section-icon {
  width:44px; height:44px; border-radius:10px; background:var(--primary-light);
  display:flex; align-items:center; justify-content:center; font-size:22px; flex-shrink:0;
}
.c-green { background:#dcfce7; color:#16a34a; }
.c-blue  { background:#dbeafe; color:#2563eb; }
.c-red   { background:#fee2e2; color:#dc2626; }
.c-amber { background:#fef3c7; color:#d97706; }
.section-title    { font-size:20px; font-weight:700; color:#0f172a; }
.section-subtitle { font-size:13px; color:#64748b; margin-top:2px; }
.screenshot.has-img   { border:1px solid #e2e8f0; border-radius:8px; overflow:hidden; margin:16px 0; }
.screenshot.has-img img { width:100%; display:block; }
.screenshot-caption   { border-top:1px solid #e2e8f0; background:#f8fafc; padding:12px 16px; font-size:13px; }
.screenshot.no-img    { border:2px dashed #e2e8f0; border-radius:8px; padding:48px; text-align:center; color:#94a3b8; font-size:14px; margin:16px 0; }
.action-map       { background:#f8fafc; border:1px solid #e2e8f0; border-radius:10px; padding:16px; margin:20px 0; }
.action-map-title { font-size:11px; font-weight:700; text-transform:uppercase; letter-spacing:.05em; color:#94a3b8; margin-bottom:12px; }
.action-grid      { display:grid; grid-template-columns:repeat(auto-fill,minmax(200px,1fr)); gap:10px; }
.action-card      { display:flex; flex-direction:column; gap:6px; padding:12px 14px; border-radius:8px; border:1px solid #e2e8f0; background:white; text-decoration:none; color:inherit; }
.action-card.action-nav    { border-color:var(--blue); transition:box-shadow .15s,background .15s; }
.action-card.action-nav:hover { background:var(--blue-light); box-shadow:0 2px 8px rgba(37,99,235,.15); }
.action-card.action-submit { border-color:var(--green); }
.action-card.action-dialog { border-color:var(--amber); }
.action-card.action-delete { border-color:var(--red); }
.action-top   { display:flex; align-items:center; justify-content:space-between; gap:8px; }
.action-label { font-weight:600; font-size:14px; color:#1e293b; }
.action-badge { font-size:10px; font-weight:700; padding:2px 7px; border-radius:10px; white-space:nowrap; flex-shrink:0; }
.action-badge.nav    { background:var(--blue-light);  color:var(--blue);  }
.action-badge.submit { background:var(--green-light); color:var(--green); }
.action-badge.dialog { background:var(--amber-light); color:var(--amber); }
.action-badge.delete { background:var(--red-light);   color:var(--red);   }
.action-dest  { font-size:12px; color:var(--blue); font-weight:500; }
.action-desc  { font-size:12px; color:#64748b; }
.steps  { display:flex; flex-direction:column; margin:20px 0; }
.step   { display:flex; gap:16px; position:relative; padding-bottom:20px; }
.step:last-child { padding-bottom:0; }
.step::before { content:''; position:absolute; left:15px; top:32px; bottom:0; width:2px; background:#e2e8f0; }
.step:last-child::before { display:none; }
.step-num  { width:32px; height:32px; border-radius:50%; background:var(--primary); color:white; display:flex; align-items:center; justify-content:center; font-weight:700; font-size:14px; flex-shrink:0; }
.step-body { padding-top:6px; font-size:14px; line-height:1.6; }
.img-zoom       { border:1px solid #e2e8f0; border-radius:8px; display:block; max-width:560px; width:100%; margin:10px 0; box-shadow:0 2px 8px rgba(0,0,0,.07); }
.img-inline     { display:inline-block; vertical-align:middle; max-height:28px; border:1px solid #e2e8f0; border-radius:4px; margin:0 4px; }
.img-zoom-block { background:#f8fafc; border:1px solid #e2e8f0; border-radius:8px; padding:12px 14px; margin:12px 0; }
.img-zoom-label { font-size:11px; font-weight:700; color:#94a3b8; text-transform:uppercase; letter-spacing:.05em; margin-bottom:8px; }
.box-warn { border-left:4px solid var(--red);   background:var(--red-light);   padding:12px 16px; border-radius:0 6px 6px 0; margin:16px 0; font-size:14px; }
.box-info { border-left:4px solid var(--blue);  background:var(--blue-light);  padding:12px 16px; border-radius:0 6px 6px 0; margin:16px 0; font-size:14px; }
.box-tip  { border-left:4px solid var(--green); background:var(--green-light); padding:12px 16px; border-radius:0 6px 6px 0; margin:16px 0; font-size:14px; }
.badge        { display:inline-block; padding:2px 8px; border-radius:10px; font-size:11px; font-weight:600; }
.badge-green  { background:var(--green-light); color:var(--green); }
.badge-red    { background:var(--red-light);   color:var(--red);   }
.badge-blue   { background:var(--blue-light);  color:var(--blue);  }
.badge-amber  { background:var(--amber-light); color:var(--amber); }
.badge-gray   { background:var(--gray-light);  color:var(--gray);  }
.related      { display:flex; align-items:center; gap:8px; flex-wrap:wrap; margin-top:24px; padding-top:16px; border-top:1px solid #e2e8f0; }
.related-label { font-size:12px; color:#94a3b8; font-weight:600; white-space:nowrap; }
.related-chip  { font-size:12px; padding:4px 12px; background:var(--gray-light); border-radius:12px; color:#475569; text-decoration:none; border:1px solid #e2e8f0; transition:background .15s; }
.related-chip:hover { background:var(--primary-light); color:var(--primary); border-color:var(--primary); }
.lang-bar     { display:flex; align-items:center; justify-content:space-between; padding:10px 20px; border-bottom:1px solid rgba(255,255,255,.08); font-size:11px; color:rgba(255,255,255,.45); }
.lang-switch  { display:flex; gap:4px; }
.lang-btn     { padding:3px 10px; border-radius:4px; font-size:11px; font-weight:700; background:transparent; border:1px solid rgba(255,255,255,.2); color:rgba(255,255,255,.5); cursor:pointer; transition:all .15s; }
.lang-btn:hover  { background:rgba(255,255,255,.1); color:white; }
.lang-btn.active { background:rgba(255,255,255,.15); color:white; border-color:rgba(255,255,255,.4); }
/* device tabs — PC / Mobile screenshot switcher
 * ⚠️  ห้ามใช้ .device-mobile { display:none } ใน CSS เด็ดขาด
 *     ให้ใช้ hidden attribute ใน HTML แทน: <div class="device-view device-mobile" hidden>
 *     เพราะ CSS class rule จะชนะ v.style.display='' ทำให้ toggle ไม่ทำงาน */
.device-tabs   { display:flex; gap:6px; padding:10px 14px 0; background:#f8fafc; border-bottom:1px solid #e2e8f0; }
.device-tab    { padding:5px 14px; border-radius:6px 6px 0 0; border:1px solid #e2e8f0; border-bottom:none;
                 font-size:12px; font-weight:600; background:white; color:#64748b; cursor:pointer; transition:all .15s; }
.device-tab:hover  { color:#1e293b; }
.device-tab.active { background:white; color:#0f172a; border-color:#cbd5e1; border-bottom-color:white; margin-bottom:-1px; z-index:1; position:relative; }
.device-view   { padding:0; }
.device-mobile img { border-radius:12px; box-shadow:0 4px 16px rgba(0,0,0,.12); }
table { width:100%; border-collapse:collapse; margin:16px 0; }
th    { text-transform:uppercase; font-size:11px; letter-spacing:.05em; color:#64748b; padding:8px 12px; border-bottom:2px solid #e2e8f0; text-align:left; }
td    { padding:10px 12px; border-bottom:1px solid #f1f5f9; font-size:14px; }
```

---

## Output

| สิ่งที่สร้าง | Path |
|---|---|
| Config (ครั้งแรก) | `~/docs/<projectName>/guide.config.json` |
| Capture credential (ถ้าบันทึก) | `~/docs/<projectName>/capture.env` ← นอก project ปลอดภัย |
| Session (ถ้า capture) | `~/docs/<projectName>/auth.json` ← นอก project ปลอดภัย |
| Screenshots (ถ้า capture) | `~/docs/<projectName>/images/*.png` |
| HTML (th) | `~/docs/<projectName>/<moduleName>-guide.html` |
| HTML (en) | `~/docs/<projectName>/<moduleName>-guide-en.html` |
| HTML (th+en) | `~/docs/<projectName>/<moduleName>-guide-bilingual.html` |

HTML standalone — รูปทั้งหมด embed base64 แชร์ได้ทันที ไม่ต้องมี server

---

## ตัวอย่าง command

**Standard (ใช้ cache + สร้าง HTML ใหม่):**
```
/create-guide basket-circulation                          → ไทย (cache ทุก source, HTML ใหม่)
/create-guide basket-circulation en                       → อังกฤษ
/create-guide basket-circulation th+en                    → 2 ภาษา + toggle
```

**Capture (Playwright auto-screenshot):**
```
/create-guide basket-circulation --capture                → ถามแผน + capture PC+mobile เฉพาะที่ขาด
/create-guide basket-circulation --capture --no-mobile    → capture PC เท่านั้น
/create-guide basket-circulation --capture --replan       → capture + ถามแผนใหม่ทั้งหมด
/create-guide basket-circulation --capture --recapture    → capture รูปใหม่ทุกรูป
/create-guide basket-circulation th+en --capture          → 2 ภาษา + toggle + capture
```

**Skip flags (ลด token):**
```
/create-guide basket-circulation --capture --no-plan      → capture ไม่ถามแผน (ใช้ extraCaptures เดิม)
/create-guide basket-circulation --capture --overview-only → capture overview PC+mobile เท่านั้น
/create-guide basket-circulation --capture --no-vision    → capture แต่ข้าม vision analysis
/create-guide basket-circulation --no-code                → HTML จาก routes + cache (ไม่อ่าน source)
/create-guide basket-circulation --html-only              → HTML จาก cache ทั้งหมด (เร็วที่สุด)
```

**Shorthand / Combo:**
```
/create-guide basket-circulation --capture --lite         → capture + ข้ามแผน + overview + ข้าม vision
/create-guide basket-circulation --capture --no-plan --no-vision  → capture เงียบ ไม่ถาม ไม่ vision
/create-guide basket-circulation th+en --html-only        → 2 ภาษา สร้าง HTML จาก cache ทันที
```

**เพิ่มรูปเอง แล้วสร้าง HTML ใหม่:**
```
/create-guide basket-circulation                          → สร้าง HTML ใหม่ + auto-pick รูปใหม่ (ชื่อใหม่)
/create-guide basket-circulation --refresh-images         → vision-อ่านรูปใหม่ทั้งหมด + สร้าง HTML
                                                            (ใช้เมื่อวางรูปทับชื่อเดิม หรือแก้ไขรูปเดิม)
/create-guide basket-circulation --no-vision              → สร้าง HTML ใหม่เร็วที่สุด (ไม่ vision)
```

**Cache management:**
```
/create-guide basket-circulation --refresh-code           → อ่าน source code ใหม่ทุก slug
/create-guide basket-circulation --refresh-images         → vision-อ่านรูปใหม่ทุกรูป (ล้าง imageCache)
/create-guide basket-circulation --fresh                  → ล้าง cache ทั้งหมด + capture ใหม่ทุกอย่าง
/create-guide receiving --capture                         → module อื่น
```
