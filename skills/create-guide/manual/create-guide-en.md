---
description: Generate standalone HTML user guides — combining 4 sources (CLAUDE.md + Source Code + Routes + Screenshot Vision), multi-project, multi-framework, bilingual (th/en), Playwright auto-capture, secure auth
---

> **Preface — Read before starting every time**
>
> This command generates a standalone HTML user guide for any module in the project
> by gathering data from **4 sources** in parallel: CLAUDE.md (module color/conventions),
> Source Code (actual buttons/forms/dialogs), Route Structure (paths and menu labels),
> and Screenshot Vision (real captured screenshots).
>
> The result is a **single HTML file** — all images embedded as base64, shareable immediately,
> no server required, no extra installation, open directly in any browser.
>
> **`--capture` mode** uses Playwright to log in and screenshot every page automatically across **2 viewports**
> — 🖥️ **PC** (1440×900) for full layout and 📱 **Mobile** (390×844) for detail clarity.
> Each page in the HTML guide has a `🖥️ PC` / `📱 Mobile` toggle button.
> Credentials are never stored in code — session tokens are used instead, stored at `~/docs/<projectName>/` outside the project directory.
> Before every screenshot, the script automatically hides **React Query DevTools / TanStack Query DevTools**
> via JavaScript inject to prevent panels from blocking forms or page content.
> For PC-only capture, add `--no-mobile`.
>
> **Without `--capture`** → uses images already in `~/docs/<projectName>/images/`.
> Suitable when screenshots already exist, or when updating content without recapturing.

---

Generate HTML user guide for module: **$ARGUMENTS**

**Argument format:**
```
/create-guide <moduleName>                          → Thai only (default)
/create-guide <moduleName> en                       → English only
/create-guide <moduleName> th+en                    → Bilingual with toggle button in sidebar
/create-guide <moduleName> --capture                → Thai + planning + Playwright auto-capture (PC + mobile)
/create-guide <moduleName> en --capture             → English + planning + auto-capture (PC + mobile)
/create-guide <moduleName> th+en --capture          → Bilingual + toggle + planning + auto-capture (PC + mobile)
/create-guide <moduleName> --capture --no-mobile    → PC capture only (skip mobile viewport)
/create-guide <moduleName> --capture --replan       → New capture + re-ask plan for all pages
/create-guide <moduleName> --capture --recapture    → Capture all images fresh (ignore cached images)
/create-guide <moduleName> --fresh                  → Clear all cache + recapture everything
/create-guide <moduleName> --refresh-code           → Re-read source code for all slugs
```

**Skip flags — use to reduce token usage by skipping steps:**
```
── skip capture steps ──
--no-plan          → Skip planning prompt, use existing extraCaptures (or overview-only if none)
--overview-only    → Capture overview PC+mobile only, skip all dialog/extra captures

── skip source reading ──
--no-code          → Skip Glob + read source code for all slugs (use cache or routes only)
--no-vision        → Skip vision analysis for all images, use filename pattern for type detection

── skip almost everything ──
--html-only        → Build HTML from existing cache only, skip auth + capture + all sources
                     (if cache is missing → warn and continue with available data)

── shorthand ──
--lite             → Shorthand for: --no-plan --overview-only --no-vision  (fewest tokens, still captures)
```

**Estimated token savings:**
| flag | skips | tokens saved |
|---|---|---|
| `--no-plan` | capture planning prompt | ~500–1,000 |
| `--no-code` | Glob + read source code | ~2,000–5,000 |
| `--no-vision` | vision analysis per image | ~800–2,000 per image |
| `--overview-only` | extra capture steps | ~200–500 |
| `--html-only` | auth + capture + 4 sources | ~80–95% of total |
| `--lite` | plan + extra capture + vision | ~60–80% |

---

## Step 0 — Parse Arguments + Verify Project

### 0-A  Parse $ARGUMENTS
```
parts        = trim("$ARGUMENTS").split(/\s+/)
moduleName   = parts[0]                          # e.g. "basket-circulation"
langMode     = parts.find(p => ["th","en","th+en"].includes(p)) ?? "en"

# ── capture flags ──────────────────────────────────────────────
captureMode  = parts.includes("--capture")       # true | false
mobileMode   = !parts.includes("--no-mobile")   # false = PC only
replanMode   = parts.includes("--replan")        # re-ask plan for all pages
recaptureMode= parts.includes("--recapture")     # capture all images fresh

# ── skip flags (reduce tokens) ────────────────────────────────
noPlanMode   = parts.includes("--no-plan")       # skip planning prompt
overviewOnly = parts.includes("--overview-only") # capture overview only
noCodeMode   = parts.includes("--no-code")       # skip source code reading
noVisionMode = parts.includes("--no-vision")     # skip vision analysis
htmlOnlyMode = parts.includes("--html-only")     # skip everything, build HTML from cache

# ── shorthand ─────────────────────────────────────────────────
if parts.includes("--lite"):
  noPlanMode = overviewOnly = noVisionMode = true  # expand --lite into 3 flags

# ── html-only implies no-capture ──────────────────────────────
if htmlOnlyMode:
  captureMode = false   # no point capturing when only HTML is needed
```

**Decision tree before starting:**
```
htmlOnlyMode = true  → skip Step 0-D, 0-E, 0-F, 1 entirely → go to Step 2 immediately (use cache)
captureMode  = false → skip Step 0-D, 0-E
noPlanMode   = true  → skip Step 0-E.1 (planning dialog)
overviewOnly = true  → in capture loop: skip extraCaptures for all pages
noCodeMode   = true  → skip Source 3 (read from cache if available)
noVisionMode = true  → skip Source 4 vision read (use filename-based type)
```

### 0-B  Find Project Name + Framework

Read `package.json` (parallel with `composer.json` if present):
- `projectName` = field `name`  (if missing → name of the working directory folder)
- Check `dependencies` + `devDependencies` to detect framework:

| dependency found | framework | routePattern |
|---|---|---|
| `"next"` | `nextjs` | `src/app/**/page.tsx`, `src/pages/**/*.tsx`, `src/constants/menu.ts` |
| `"nuxt"` / `"nuxt3"` | `nuxt` | `pages/**/*.vue`, `composables/`, `server/routes/` |
| `"vue"` + `"vue-router"` | `vue-spa` | `src/router/*.{js,ts}`, `src/views/**/*.vue` |
| `"@angular/core"` | `angular` | `src/app/**/*-routing.module.ts`, `src/app/**/*.routes.ts` |
| `"gatsby"` | `gatsby` | `src/pages/**/*.{tsx,jsx}` |
| none match | `generic` | Glob `src/**/{page,route,screen,view,Page}*.{tsx,vue,jsx,py}` |

### 0-C  Docs Root

```
docsRoot       = ~/docs/<projectName>/
imagesDir      = ~/docs/<projectName>/images/
configFile     = ~/docs/<projectName>/guide.config.json
authFile       = ~/docs/<projectName>/auth.json        ← session only, no password
captureEnvFile = ~/docs/<projectName>/capture.env      ← CAPTURE_USER/CAPTURE_PASS outside project dir
```

Create folder if missing:
```powershell
New-Item -ItemType Directory -Force "$env:USERPROFILE\docs\<projectName>\images"
```

---

## Step 0-D — Auth Setup  *(runs only when captureMode = true)*

> If `captureMode = false` → skip this step entirely, go to Step 1.
>
> If `htmlOnlyMode = true` → skip Step 0-D, 0-E, 0-F entirely, go to **Step 1** immediately.
> (sources 1–4 read from cache only; if cache is missing → warn user but continue with available data)
>
> ```
> ⚡ [--html-only] skip all capture/source steps → build HTML from existing cache
>    guide.config.json found: meta✅  routes✅  pages(3/4)✅  images(8 files)✅
> ```

Check in **waterfall** order:

```
1. auth.json exists + age < 8 hours
       → ✅ reuse existing session, skip to Step 0-E

2. ~/docs/<projectName>/capture.env has CAPTURE_USER + CAPTURE_PASS
       → ✅ read credentials from capture.env and log in
          (file is outside project dir — no risk of git commit)

3. neither exists
       → 🔐 prompt user in terminal (see below)
```

### Case 3 — Terminal prompt

Display this message and wait for input:

```
╔══════════════════════════════════════════════════════╗
║  🔐  Login to capture screenshots                     ║
║      URL: <baseUrl from guide.config.json>            ║
╚══════════════════════════════════════════════════════╝

  Username / Email :
  Password         : (characters will be hidden)

  [Enter] continue   [S + Enter] also save to ~/docs/<projectName>/capture.env
```

- Password must use raw mode to hide input (show only `*` or blank)
- If user presses `S` → save `CAPTURE_USER` / `CAPTURE_PASS` to `~/docs/<projectName>/capture.env`
  then confirm **"Saved capture.env at ~/docs/<projectName>/ (outside project — safe)"**
- No `.gitignore` changes needed — file is already outside project directory

### 0-D.1  Login with Playwright

```typescript
// scripts/capture-setup.ts
import { chromium } from 'playwright'
import * as fs from 'fs'
import * as path from 'path'
import * as readline from 'readline'

const AUTH_PATH        = path.join(process.env.HOME!, 'docs', PROJECT_NAME, 'auth.json')
const CAPTURE_ENV_PATH = path.join(process.env.HOME!, 'docs', PROJECT_NAME, 'capture.env')

async function getCredentials(): Promise<{ user: string; pass: string } | null> {
  // case 1 — session still valid
  if (fs.existsSync(AUTH_PATH)) {
    const ageHours = (Date.now() - fs.statSync(AUTH_PATH).mtimeMs) / 3_600_000
    if (ageHours < 8) {
      console.log(`✅ reusing existing session (age ${ageHours.toFixed(1)} hrs)`)
      return null  // no login needed
    }
  }

  // case 2 — ~/docs/<project>/capture.env (outside project dir — safe)
  if (fs.existsSync(CAPTURE_ENV_PATH)) {
    const env = parseEnv(fs.readFileSync(CAPTURE_ENV_PATH, 'utf8'))
    if (env.CAPTURE_USER && env.CAPTURE_PASS) {
      console.log(`✅ reading credentials from ${CAPTURE_ENV_PATH}`)
      return { user: env.CAPTURE_USER, pass: env.CAPTURE_PASS }
    }
  }

  // case 3 — prompt in terminal
  return promptCredentials()
}

async function promptCredentials(): Promise<{ user: string; pass: string }> {
  const rl = readline.createInterface({ input: process.stdin, output: process.stdout })
  const ask = (q: string) => new Promise<string>(res => rl.question(q, res))

  console.log('\n╔══════════════════════════════════════════════════════╗')
  console.log('║  🔐  Login to capture screenshots                     ║')
  console.log(`║      URL: ${BASE_URL.padEnd(42)}║`)
  console.log('╚══════════════════════════════════════════════════════╝\n')

  const user = await ask('  Username / Email : ')
  process.stdout.write('  Password         : ')
  const pass = await hiddenInput()
  const save = await ask(`\n  Save to ${CAPTURE_ENV_PATH}? (y/N) : `)

  rl.close()

  if (save.toLowerCase() === 'y') {
    fs.mkdirSync(path.dirname(CAPTURE_ENV_PATH), { recursive: true })
    fs.writeFileSync(CAPTURE_ENV_PATH, `CAPTURE_USER=${user}\nCAPTURE_PASS=${pass}\n`)
    console.log(`  💾 Saved to ${CAPTURE_ENV_PATH} (outside project — safe)\n`)
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
  if (!creds) return  // session still valid

  const browser = await chromium.launch({ headless: true })
  const context = await browser.newContext()
  const page    = await context.newPage()

  console.log('  🌐 Logging in...')
  await page.goto(`${baseUrl}/login`)

  // adjust selectors to match the target system
  await page.fill('input[type="email"], input[name="username"], input[name="email"]', creds.user)
  await page.fill('input[type="password"]', creds.pass)
  await page.click('button[type="submit"], button:has-text("Login"), button:has-text("Sign in")')

  try {
    await page.waitForURL(url => !url.toString().includes('/login'), { timeout: 15_000 })
    console.log('  ✅ Login successful\n')
  } catch {
    await browser.close()
    throw new Error('❌ Login failed — check username/password and try again')
  }

  // save session (cookies only — no password)
  fs.mkdirSync(path.dirname(AUTH_PATH), { recursive: true })
  await context.storageState({ path: AUTH_PATH })
  console.log(`  💾 Session saved → ${AUTH_PATH}\n`)

  await browser.close()
}
```

### 0-D.2  Files that should NOT be committed

| File | Reason |
|---|---|
| `~/docs/*/capture.env` | Contains CAPTURE_USER/CAPTURE_PASS — already outside project dir, no commit risk |
| `~/docs/*/auth.json` | Contains session token — already outside project dir, no commit risk |
| `~/docs/*/images/` | Screenshots (optional) |

---

## Step 0-E — Auto Capture  *(runs only when captureMode = true)*

> If `captureMode = false` → skip this step, go to Step 1

> ⚠️ **Security Warning — notify user before every capture:**
> Captured images may contain real user data (names, order IDs, amounts, etc.)
> If building a guide to share externally → use demo/staging data instead of production
> HTML output embeds all images as base64 — sharing the HTML file = sharing all images

### 0-E.1 — Interactive Planning (always ask before capturing)

> **Skip this step if:**
> - `noPlanMode = true` (`--no-plan`) → use existing `extraCaptures` from `guide.config.json`
>   Pages without `extraCaptures` → capture overview only for that page
>   Show: `⏭️  [--no-plan] skip planning — using existing extraCaptures (bc-domains✅ bc-trade⬜→overview)`
> - `overviewOnly = true` (`--overview-only`) → skip immediately, no prompts, no extraCaptures
>   Show: `⏭️  [--overview-only] skip planning — capture overview PC+📱 only`
> - `replanMode = false` + all pages already have `extraCaptures` → use existing values, no re-prompt

Before capturing, scan source code for each page in the module and **ask the user page by page** which dialogs/buttons to open for screenshots.

Display in this format and wait for response:

```
╔══════════════════════════════════════════════════════════════════╗
║  🗂️  Capture plan for: basket-circulation                         ║
║      Found 4 pages — please answer one page at a time            ║
╚══════════════════════════════════════════════════════════════════╝

─────────────────────────────────────────
📄 Page 1/4: Domains  (/basket-circulation/domains)
─────────────────────────────────────────
Found interactive elements:

  Dialogs / Sheets:
    [A] "Add Domain"     → Sheet to add a new Domain        (trigger: button "Add Domain")
    [B] "Update Quota"   → Dialog to edit Quota             (trigger: button "Edit" in row)
    [C] "Domain Detail"  → Sheet with Domain details        (trigger: button "Details" in row)

  Other buttons found:
    [D] "Delete"         → ⚠️  submit/delete — not recommended to capture
    [E] "Save"           → ⚠️  form submit — not recommended to capture
    [F] filter dropdown  → dropdown filter selector (safe to view)

Select elements to capture (e.g.: A C F)
or [S] skip this page / [ALL-SAFE] select all non-submit items :
```

**Grouping rules** (scanned from source code + labels):

| signal found in code | classified as | recommendation |
|---|---|---|
| `onClick` → open Sheet/Dialog state | `nav-dialog` | ✅ safe to capture |
| `href` / `router.push` | `nav-page` | ✅ safe to capture |
| `mutation.mutate` / `form.handleSubmit` | `submit` | ⚠️ not recommended |
| `onDelete` / `method: DELETE` | `delete` | ⚠️ not recommended |
| dropdown/select without submit | `nav-filter` | ✅ safe to capture |
| tooltip / hover state | `nav-hover` | ✅ safe to capture |

After user responds for each page → save as `extraCaptures` to `guide.config.json` automatically, then ask next page.

When all pages answered, show summary before actual capture:

```
╔══════════════════════════════════════════════════════════════════╗
║  ✅  Capture plan for basket-circulation                          ║
╚══════════════════════════════════════════════════════════════════╝

  bc-domains    overview (PC+📱) + [A] Add Domain sheet + [C] Domain Detail + [F] filter
  bc-basket     overview (PC+📱) only (skipped as selected)
  bc-trade      overview (PC+📱) + [A] Scan dialog + [D] Status tooltip
  bc-void       overview (PC+📱) + [A] Void Reason dialog

  Total: 4 pages / ~11 PC images + ~4 mobile images = ~15 images / no submit actions ✅
  📱 mobile capture for all pages (390×844)  |  disable with --no-mobile

[Enter] start capture  |  [E] edit plan :
```

> **Important:** if `guide.config.json` already has `extraCaptures` for a page →
> skip asking and use existing values directly, no re-prompt.
> Unless user runs `/create-guide <module> --capture --replan` → re-ask all pages.

Read `guide.config.json → modules[moduleName].routes` then capture each page:

```typescript
// scripts/capture.ts
import { chromium } from 'playwright'
import * as path from 'path'
import * as fs from 'fs'

const AUTH_PATH = path.join(process.env.HOME!, 'docs', PROJECT_NAME, 'auth.json')
const IMG_DIR   = path.join(process.env.HOME!, 'docs', PROJECT_NAME, 'images')

// viewports used — PC (1440×900) and mobile (390×844 ≈ iPhone 14 Pro)
const VIEWPORTS = {
  pc:     { width: 1440, height: 900 },
  mobile: { width: 390,  height: 844 },
} as const

export async function captureModule(
  baseUrl: string,
  moduleName: string,
  routes: Array<{ slug: string; path: string; extraCaptures?: CaptureStep[] }>,
  options: { mobile?: boolean } = {}         // mobile=true by default (disable with --no-mobile)
) {
  const withMobile = options.mobile !== false

  const browser = await chromium.launch({ headless: true })

  // PC context — overview + all extra captures
  const pcContext = await browser.newContext({
    storageState: AUTH_PATH,
    viewport: VIEWPORTS.pc,
  })
  const page = await pcContext.newPage()

  // Mobile context — overview only (unless --no-mobile)
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

  // helper — hide React Query DevTools (and other dev panels) before every screenshot
  // uses JS inject instead of clicking, because DevTools panel may block target elements
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

    // skip pages where both PC and mobile images already exist
    if (pcSkip && (!withMobile || mobileSkip) && !(route.extraCaptures?.length)) {
      console.log(`  ⏭️  [skip] ${route.slug} — images already exist (PC${withMobile ? '+mobile' : ''})`)
      continue
    }

    console.log(`  📸 Capturing ${route.slug}  (${route.path})`)

    // ─── PC overview ───────────────────────────────────────────────
    if (!pcSkip) {
      await page.goto(`${baseUrl}${route.path}`, { waitUntil: 'networkidle' })
      await page.waitForTimeout(800)           // wait for animations to finish
      await hideDevTools()
      await page.screenshot({ path: pcOverviewPath, fullPage: false })
      console.log(`    ✅ overview (PC 1440×900) → ${route.slug}.png`)
    } else {
      console.log(`    ⏭️  skip PC overview — already exists`)
    }

    // ─── Mobile overview ──────────────────────────────────────────
    if (withMobile && mobilePage && !mobileSkip) {
      await mobilePage.goto(`${baseUrl}${route.path}`, { waitUntil: 'networkidle' })
      await mobilePage.waitForTimeout(800)
      await hideDevTools(mobilePage)
      await mobilePage.screenshot({ path: mobileOverviewPath, fullPage: false })
      console.log(`    ✅ overview (📱 390×844)  → ${route.slug}-mobile.png`)
    } else if (withMobile && mobileSkip) {
      console.log(`    ⏭️  skip mobile overview — already exists`)
    }

    // ─── Extra captures (PC only — dialog/zoom/btn/step) ──────────
    // skip all if overviewOnly = true (--overview-only)
    if (OVERVIEW_ONLY) {
      if (route.extraCaptures?.length) {
        console.log(`    ⏭️  [--overview-only] skip ${route.extraCaptures.length} extra step(s)`)
      }
    } else {
      // page was PC-skipped but has extras → goto PC first
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
  console.log(`\n✅ Capture complete → ${IMG_DIR}\n`)
}

type CaptureStep =
  | { action: 'screenshot';         label: string }
  | { action: 'click';              selector: string }
  | { action: 'screenshot_element'; selector: string; label: string; padding?: number }
  | { action: 'wait';               ms: number }
  | { action: 'fill';               selector: string; value: string }
  | { action: 'hide_devtools' }
  // --- targeted captures ---
  | { action: 'highlight';   selector: string; label: string; color?: string; caption?: string; padding?: number }
  //    draw box around element + label badge then screenshot — use to explain key buttons in guide
  | { action: 'hover';       selector: string; label: string }
  //    hover to reveal tooltip/dropdown then full-page screenshot — use for badge/status tooltips
  | { action: 'wait_for';    selector: string; timeout?: number }
  //    wait for element to appear before next step — use after click with loading or animation

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
      if (!el) { console.log(`    ⚠️  element not found: ${step.selector}`); break }
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
      console.log(`    🙈 DevTools hidden`)
      break

    // ─────────────────────────────────────────────────────────────
    // highlight — draw box + label badge around element then screenshot
    //   use: explain key buttons, e.g. "This button is Scan" in the guide
    // ─────────────────────────────────────────────────────────────
    case 'highlight': {
      await hideDevTools()
      const pad     = step.padding ?? 12
      const color   = step.color   ?? '#ef4444'   // red default
      const caption = step.caption ?? step.label

      const el = await page.$(step.selector)
      if (!el) { console.log(`    ⚠️  element not found: ${step.selector}`); break }
      const box = await el.boundingBox()

      // inject overlay: box + label badge above
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

      // crop around element + space for badge above
      const p = path.join(dir, `${slug}-${step.label}.png`)
      await page.screenshot({
        path: p,
        clip: {
          x:      Math.max(0, box.x - pad - 20),
          y:      Math.max(0, box.y - pad - 40),  // room for badge
          width:  box.width  + pad*2 + 40,
          height: box.height + pad*2 + 60,
        }
      })

      // remove overlay after screenshot
      await page.evaluate(() => {
        document.getElementById('__guide_highlight__')?.remove()
      })
      console.log(`    ✅ highlight → ${slug}-${step.label}.png  ("${caption}")`)
      break
    }

    // ─────────────────────────────────────────────────────────────
    // hover — hover element then full-page screenshot (tooltip/dropdown appears)
    //   use: explain status badges with tooltips, icons with descriptions
    // ─────────────────────────────────────────────────────────────
    case 'hover': {
      await page.hover(step.selector)
      await page.waitForTimeout(500)   // wait for tooltip animation
      await hideDevTools()
      const p = path.join(dir, `${slug}-${step.label}.png`)
      await page.screenshot({ path: p, fullPage: false })
      console.log(`    ✅ hover → ${slug}-${step.label}.png`)
      break
    }

    // ─────────────────────────────────────────────────────────────
    // wait_for — wait for selector to appear in DOM before next step
    //   use: after click with loading spinner, lazy render, animation
    // ─────────────────────────────────────────────────────────────
    case 'wait_for': {
      const timeout = step.timeout ?? 8_000
      await page.waitForSelector(step.selector, { state: 'visible', timeout })
      console.log(`    ⏳ wait_for found "${step.selector}"`)
      break
    }

  }
}
```

### Define extraCaptures in `guide.config.json`

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
              "caption": "Click to add a new Domain",
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
              "caption": "Scan QR to start Trade",
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

**Naming convention for captured images:**

| action / pattern | filename produced | how used in guide |
|---|---|---|
| overview PC | `bc-domains.png` | main PC image (1440×900) in screenshot block |
| overview mobile | `bc-domains-mobile.png` | 📱 mobile tab in screenshot block |
| `highlight` | `bc-trade-btn-scan.png` | `type:"btn"` → `<img class="img-inline">` beside step description |
| `hover` | `bc-trade-tooltip-status-pending.png` | `type:"zoom"` → `.img-zoom` explaining tooltip in step |
| `wait_for` | *(no image — waits only)* | used with following `screenshot` step |

### Progress display during capture

```
🎬 Starting capture for module: basket-circulation
   URL: http://localhost:3000
   Session: ~/docs/wms/auth.json (age 0.2 hrs)
   Viewports: 🖥️ PC (1440×900) + 📱 mobile (390×844)

  ⏭️  [skip] bc-domains — images already exist (PC+mobile, 4 files)
  ⏭️  [skip] bc-basket  — images already exist (PC+mobile, 2 files)

  📸 [3/4] bc-trade     /basket-circulation/trade
    ✅ overview (PC 1440×900) → bc-trade.png
    ✅ overview (📱 390×844)  → bc-trade-mobile.png
    ✅ highlight              → bc-trade-btn-scan.png  ("Scan QR to start Trade")
    ✅ hover                  → bc-trade-tooltip-status-pending.png
    ⏳ wait_for found ".scan-input"
    🙈 DevTools hidden
    ✅ scan-ready             → bc-trade-scan-ready.png

  📸 [4/4] bc-void      /basket-circulation/void
    ✅ overview (PC 1440×900) → bc-void.png
    ✅ overview (📱 390×844)  → bc-void-mobile.png

✅ Capture done — 2 pages recaptured (PC+mobile) / 2 pages skipped (already existed)
```

---

## Step 0-F — Cache Check Before Work

> **Goal:** skip work already done, only process what is missing — except HTML which is always rebuilt.

Check all of these before starting Step 1:

```
cacheState = {
  claudeMd:     is there data in guide.config.json → modules[moduleName].meta?
  routes:       are there routes in guide.config.json → modules[moduleName].routes?
  sourceCode:   is there data in guide.config.json → modules[moduleName].pages[slug]?  (per slug)
  images:       do image files exist in ~/docs/<projectName>/images/?                  (per slug)
  html:         ❌ not cached — always rebuilt
}
```

Show cache status summary before starting (including active skip flags):

```
⚡ Cache check — basket-circulation
   Active flags: --no-vision --overview-only

  Source 1  CLAUDE.md meta     ✅ cached → skip
  Source 2  Routes             ✅ cached → skip
  Source 3  Source code
              bc-domains       ✅ cached → skip
              bc-basket        ✅ cached → skip
              bc-trade         ❌ missing → read fresh
              bc-void          ❌ missing → read fresh
  Source 4  Screenshots (vision)
              bc-domains.png               ✅ exists → skip (PC)
              bc-domains-mobile.png        ✅ exists → skip (📱)
              bc-domains-dialog-create.png ✅ exists → skip
              bc-trade.png                 ❌ missing → capture fresh (PC)
              bc-trade-mobile.png          ❌ missing → capture fresh (📱)
              bc-void.png                  ❌ missing → capture fresh (PC)
              bc-void-mobile.png           ❌ missing → capture fresh (📱)
              vision analysis              ⏭️  [--no-vision] skip — use filename pattern
  Capture   extra steps                   ⏭️  [--overview-only] skip — overview only
  HTML      <moduleName>-guide.html       🔄 always rebuilt

Savings: ~75% tokens (cached 2 slugs + CLAUDE.md + routes + skip vision + skip extras)
```

> When running `--html-only` → show only:
> ```
> ⚡ [--html-only] skip all source steps — use full cache
>    meta✅  routes✅  pages(4/4)✅  images(12 files)✅  → building HTML now
> ```

**Rules per source:**

| Source | cached where | invalidated when |
|---|---|---|
| CLAUDE.md meta | `guide.config.json → modules[mod].meta` | no `meta` key / run `--refresh-meta` |
| Routes | `guide.config.json → modules[mod].routes` | no `routes` key / run `--refresh-routes` |
| Source code (per slug) | `guide.config.json → modules[mod].pages[slug]` | no `pages[slug]` key / run `--refresh-code` |
| Images (per file) | file exists in `images/` | file missing / run `--recapture` |
| Vision cache (per file) | `imageCache[filename]` in guide.config.json | no key / run `--refresh-images` |
| HTML | — | **always rebuilt** |

**Additional `--refresh-*` flags:**
```
--refresh-meta    → clear CLAUDE.md cache and re-read
--refresh-routes  → clear routes cache and re-read
--refresh-code    → clear source code cache for all slugs and re-read
--refresh-images  → clear all imageCache entries and re-run vision on all images
                    use when: replacing images with new ones (same filename, new content)
                    or: added new images manually and want HTML to reflect actual image content
--recapture       → capture all images fresh (regardless of existing) — must use with --capture
--fresh           → clear all cache (equivalent to running all 5 flags together)
```

**Cache structure in `guide.config.json`:**
```json
{
  "baseUrl": "http://localhost:3000",
  "framework": "nextjs",
  "modules": {
    "basket-circulation": {
      "meta": {
        "color": "green",
        "description": "Basket circulation system",
        "conventions": { "badge": "green=active", "button": "outline style" },
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
          "description": "Manage Domains that own baskets",
          "cachedAt": "2025-06-10T10:00:00Z"
        },
        "bc-trade": null
      },
      "extraCaptures": { }
    }
  }
}
```

> `pages[slug] = null` means this slug has not had source code scanned yet
> If the key is missing entirely → treated as null

---

## Step 1 — Gather 4 Sources in Parallel

Read only sources that don't have cache yet (see Step 0-F):

### Source 1 — CLAUDE.md
**Skip** if `guide.config.json → modules[moduleName].meta` already has data.

Read `CLAUDE.md` from working directory (if present) and extract:
- `moduleColor` — module's primary color
- `moduleDescription` — module description
- `conventions` — UI conventions
- `routePrefix` — API/route prefix

Save to `guide.config.json → modules[moduleName].meta` with `cachedAt`.

### Source 2 — Route Structure
**Skip** if `guide.config.json → modules[moduleName].routes` already has data.

Read in parallel according to `framework` detected in Step 0-B:

**Fallback chain (try in order, stop when routes found):**
```
1. ~/docs/<projectName>/guide.config.json   → modules[moduleName].routes
2. Framework-specific route file            → per framework routePattern
3. Glob src/**/<moduleName>*/**/*.{tsx,vue} → infer from folder + filename
4. If still not found → ask user to specify path manually
```

**Next.js extra:** read `src/constants/menu.ts` (if present) → find array named `<moduleName>SubMenu`
**Vue SPA extra:** read `src/router/*.{js,ts}` → find children routes for the module

Build `routeList` and save to `guide.config.json → modules[moduleName].routes`:
```
[{ slug, path, title, menuLabel }, ...]
```

### Source 3 — Source Code

> **Skip all of Source 3 if `noCodeMode = true` (`--no-code`)**
> → use `pages[slug]` from existing cache in `guide.config.json`
> → slugs missing from cache (`null` / key missing) → set `pages[slug] = {}` (empty, no action-map/steps from code)
> → show: `⏭️  [--no-code] skip source code reading — cache: bc-domains✅ bc-trade❌(empty)`
> → in HTML guide: pages without code data → no action-map shown, but screenshots + steps from vision still appear

**Skip slugs where** `guide.config.json → modules[moduleName].pages[slug]` already has data.
**Read only slugs where** `pages[slug]` is `null` or key is missing.

Glob and read in parallel only missing slugs:
- `src/features/$ARGUMENTS/<slug>/**/*.tsx`
- `src/app/**/<slug>*/**/page.tsx`

Extract for each slug:
- **buttons** — label + type (`nav | submit | dialog | delete`) + target slug
- **formFields** — field names that user fills in
- **dialogs** — Sheets/Dialogs opened on this page
- **description** — what this page does

Save to `guide.config.json → modules[moduleName].pages[slug]` with `cachedAt`.

### Source 4 — Screenshot Vision

> **Skip all of Source 4 if `noVisionMode = true` (`--no-vision`)**
> → do not send any images to Claude for analysis, saves the most tokens
> → use **filename pattern** to determine `imageType` instead:
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
> → in HTML guide: screenshots still embedded, but captions + step text come from source code only
> → show: `⏭️  [--no-vision] skip vision analysis — using filename-based type (8 images)`

**Skip images that** have `imageCache[filename]` in `guide.config.json` already.
**Run vision only on images** without cache (new files, or invalidated).

Glob images in `~/docs/<projectName>/images/<moduleName>-*.{png,jpg,jpeg}` then split:
```
new files (no imageCache key)   → run vision → extract imageType, visibleText, uiElements, pageState
files with imageCache           → use cached data ← skip vision
--refresh-images                → clear all imageCache entries then re-run vision on all images
```

> **When to use `--refresh-images`:**
> - Replacing an image with a new one (same filename, new content), e.g. recaptured after feature update
> - Added multiple new images manually and want captions/steps in HTML to reflect actual image content
>
> **If just adding new files (new filenames):** run `/create-guide <moduleName>` normally, no `--refresh-images` needed

Save vision results to `guide.config.json → modules[moduleName].imageCache[filename]`

**Naming convention:**

| Pattern | Example | Meaning |
|---|---|---|
| `slug.png` | `bc-domains.png` | PC overview (1440×900) — primary |
| `slug-mobile.png` | `bc-domains-mobile.png` | Mobile overview (390×844) — 📱 tab |
| `slug-2.png` | `bc-domains-2.png` | Secondary PC overview |
| `slug-dialog-xxx.png` | `bc-domains-dialog-create.png` | Dialog opened on this page |
| `slug-zoom-xxx.png` | `bc-domains-zoom-toolbar.png` | Zoomed section |
| `slug-btn-xxx.png` | `bc-trade-btn-scan.png` | Individual button |
| `slug-step-N.png` | `bc-trade-step-2.png` | Step N image |

For each image → run vision and extract imageType, visibleText, uiElements, pageState, slug, context.

---

## Step 2 — Page Match

Build `pageMap` by merging data from all sources then show summary:

```
📋 Page Match for module: $ARGUMENTS

  slug             path                              images
  ──────────────── ─────────────────────────────── ─────────────────────────────────────
  ✅ bc-domains    /basket-circulation/domains      overview✅  dialog✅  zoom✅
  ✅ bc-basket     /basket-circulation/basket       overview✅
  ⚠️  bc-trade     /basket-circulation/trade        overview✅  btn❌
  ✅ bc-void       /basket-circulation/void         overview✅  dialog✅

CLAUDE.md : ✅ found  (color: green, conventions: ✅)
Capture   : ✅ auto-captured with Playwright  (or "📁 using images from images/")
```

**Synthesis rules:**
- If Source 4 (vision) conflicts with Source 3 (code) → trust Source 4
- If no overview image → use first available image instead
- If no images at all for a page → list what needs to be prepared then stop

**langMode rules:**
| langMode | build TH | build EN | HTML output |
|---|---|---|---|
| `th` | ✅ | ❌ | Thai text, no toggle |
| `en` | ❌ | ✅ | English text, no toggle |
| `th+en` | ✅ | ✅ | toggle, defaults to Thai |

**Multi-image usage rules:**
- `overview` → primary image at top of section
- `dialog` → shown in step related to opening dialog using `.img-zoom`
- `zoom` → shown in step or action-map using `.img-zoom`
- `btn` → shown inline in step using `<img class="img-inline">`
- `step-N` → shown in matching step N

---

## Step 3 — Encode Images as Base64

```powershell
[Convert]::ToBase64String([IO.File]::ReadAllBytes("$env:USERPROFILE\docs\<projectName>\images\<filename>"))
```

Store as `imageData = { "<slug>": "data:image/png;base64,<encoded>" }`

---

## Step 4 — Generate HTML Guide

**Output filenames:**
```
langMode = "th"    → ~/docs/<projectName>/<moduleName>-guide.html
langMode = "en"    → ~/docs/<projectName>/<moduleName>-guide-en.html
langMode = "th+en" → ~/docs/<projectName>/<moduleName>-guide-bilingual.html
```

HTML is standalone — all styles in a single `<style>` tag, all images embedded as base64.

### HTML Structure

Main layout is **Sidebar + Main content**:

```html
<!DOCTYPE html>
<html lang="en"><!-- for th+en: use lang="th" initially, JS switchLang will handle it -->
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title><!-- module name --> — User Guide</title>
<!-- Thai font — loads with internet, falls back to system Thai font when offline -->
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
      <div class="sidebar-logo-title"><!-- module name --></div>
      <div class="sidebar-logo-sub">
        <!-- th    → คู่มือการใช้งาน -->
        <!-- en    → User Guide -->
        <!-- th+en → <span class="i18n" data-th="คู่มือการใช้งาน" data-en="User Guide"> -->
      </div>
    </div>

    <!-- Lang toggle — shown only for langMode="th+en" -->
    <div class="lang-bar">
      <span class="lang-label i18n" data-th="ภาษา" data-en="Language">Language</span>
      <div class="lang-switch">
        <button class="lang-btn active" data-lang="th" onclick="switchLang('th')">ไทย</button>
        <button class="lang-btn"        data-lang="en" onclick="switchLang('en')">EN</button>
      </div>
    </div>

    <div class="sidebar-nav">
      <div class="nav-group-label">
        <span class="i18n" data-th="หน้าหลัก" data-en="Overview">Overview</span>
      </div>
      <a href="#top" class="nav-item" data-section="top">
        <span class="nav-item-icon">🗺️</span>
        <div>
          <div class="nav-item-label">
            <span class="i18n" data-th="ภาพรวมระบบ" data-en="System Overview">System Overview</span>
          </div>
          <div class="nav-item-sub">
            <span class="i18n" data-th="สารบัญทั้งหมด" data-en="Table of Contents">Table of Contents</span>
          </div>
        </div>
      </a>
      <div class="nav-group-label">
        <span class="i18n" data-th="ฟังก์ชั่นหลัก" data-en="Features">Features</span>
      </div>
      <!-- iterate pageMap → one nav-item per slug -->
      <a href="#<sectionId>" class="nav-item" data-section="<sectionId>">
        <span class="nav-item-icon"><!-- emoji --></span>
        <div>
          <div class="nav-item-label"><!-- original title, no translation --></div>
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
          <span class="i18n" data-th="สารบัญ" data-en="Table of Contents">Table of Contents</span>
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

  /* Lang switch — for langMode=th+en only
   *
   * Supported HTML format (use <template> instead of data-attributes):
   *
   *   <span class="i18n">
   *     <template data-lang="th"><strong>Thai</strong> text here</template>
   *     <template data-lang="en"><strong>English</strong> text here</template>
   *   </span>
   *
   * Advantage: HTML tags (<strong>, <a>, <code>, <br>) render correctly in all cases
   * because <template> stores real DOM, not strings — no entity escaping
   */
  window.switchLang = function(lang) {
    document.body.dataset.lang = lang;
    document.querySelectorAll('.i18n').forEach(function(el) {
      var tpl = el.querySelector('template[data-lang="' + lang + '"]');
      if (!tpl) return;
      // preserve template nodes, clear innerHTML, re-insert clones + original templates
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

  /* Device tab — toggle PC / Mobile screenshot within the same section */
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

### Section anatomy — 1 section per slug

> **Rule for writing `i18n` spans (langMode=th+en)**
>
> Always use `<template data-lang="...">` inside `.i18n` — never use `data-th`/`data-en` attributes
> because HTML attributes will escape `<strong>` to `&lt;strong&gt;` so tags won't render.
>
> ```html
> <!-- ✅ correct — use <template> -->
> <span class="i18n">
>   <template data-lang="th">พื้นที่การผลิตที่ <strong>เป็นเจ้าของ</strong>ตะกร้า</template>
>   <template data-lang="en">Production area that <strong>owns</strong> baskets</template>
> </span>
>
> <!-- ❌ wrong — data-attribute escapes HTML -->
> <span class="i18n" data-th="พื้นที่ <strong>เป็นเจ้าของ</strong>" data-en="...">...</span>
> ```
>
> Default content (before switchLang is called) should be placed directly outside `<template>` in Thai.
> JS swaps content from the template matching the selected language when switchLang is called.

```html
<section id="<sectionId>" class="guide-section">
  <div class="breadcrumb">
    <a href="#top">Overview</a> › Module Name › Page Name
  </div>
  <div class="section-header">
    <div class="section-icon c-<color>">🏭</div>
    <div>
      <div class="section-title">Page Name</div>
      <div class="section-subtitle">Short description</div>
      <div class="section-url">/path · API: /api-prefix/...</div>
    </div>
  </div>

  <!-- section-intro: use <template> so <strong>/<a>/<code> render correctly -->
  <div class="section-intro">
    <span class="i18n">
      <template data-lang="th"><strong>Domain</strong> คือพื้นที่การผลิตที่เป็น &ldquo;เจ้าของ&rdquo; ตะกร้า ต้องตั้งค่าก่อนใช้งาน Trade</template>
      <template data-lang="en"><strong>Domain</strong> is a production area that owns baskets. Must be set up before using Trade.</template>
      <strong>Domain</strong> is a production area that owns baskets. Must be set up before using Trade.
    </span>
  </div>

  <!--
    screenshot block — if both PC + mobile exist, use device-tabs format
    if only PC (--no-mobile or older capture), use plain has-img (no device-tabs)

    hidden rules:
      - mobile div must use "hidden" attribute in HTML only
      - never use CSS .device-mobile{display:none} — it will override JS toggle
      - showDevice() uses v.hidden = true/false which works correctly with hidden attribute
  -->
  <div class="screenshot has-img">
    <!-- device tabs — shown only when mobile image (-mobile.png) exists -->
    <div class="device-tabs">
      <button class="device-tab active" onclick="showDevice(this,'pc')">🖥️ PC</button>
      <button class="device-tab"        onclick="showDevice(this,'mobile')">📱 Mobile</button>
    </div>
    <div class="device-view device-pc">
      <img src="data:image/png;base64,...PC_B64..." alt="Page Name — PC">
    </div>
    <div class="device-view device-mobile" hidden><!-- ← hidden attribute only, no CSS -->
      <img src="data:image/png;base64,...MOBILE_B64..." alt="Page Name — Mobile"
           style="max-width:390px;margin:0 auto;display:block;">
    </div>
    <div class="screenshot-caption">
      <strong>Screen Name</strong> —
      <span class="i18n">
        <template data-lang="th">รายการ Domain ทั้งหมด พร้อมปุ่ม Add Domain</template>
        <template data-lang="en">Full Domain list with Add Domain button</template>
        Full Domain list with Add Domain button
      </span>
    </div>
  </div>

  <div class="action-map">
    <div class="action-map-title">Buttons and actions on this page</div>
    <div class="action-grid">
      <!-- action-card by type: nav / submit / dialog / delete -->
    </div>
  </div>

  <div class="steps">
    <div class="step">
      <div class="step-num">1</div>
      <div class="step-body">
        <!-- step with plain text only -->
        <span class="i18n">
          <template data-lang="th">คลิกปุ่ม <strong>Add Domain</strong> ที่มุมขวาบน</template>
          <template data-lang="en">Click the <strong>Add Domain</strong> button at the top right</template>
          Click the <strong>Add Domain</strong> button at the top right
        </span>
        <!-- if there is a dialog/zoom/step image for this step -->
        <img class="img-zoom" src="data:image/png;base64,..." alt="dialog screenshot">
      </div>
    </div>
  </div>

  <div class="related">
    <span class="related-label">See also:</span>
    <a href="#<relatedSectionId>" class="related-chip">Page Name</a>
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
 * ⚠️  NEVER use .device-mobile { display:none } in CSS
 *     Use hidden attribute in HTML instead: <div class="device-view device-mobile" hidden>
 *     CSS class rules will override v.style.display='' making the toggle non-functional */
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

| Created | Path |
|---|---|
| Config (first run) | `~/docs/<projectName>/guide.config.json` |
| Capture credentials (if saved) | `~/docs/<projectName>/capture.env` ← outside project, safe |
| Session (if captured) | `~/docs/<projectName>/auth.json` ← outside project, safe |
| Screenshots (if captured) | `~/docs/<projectName>/images/*.png` |
| HTML (th) | `~/docs/<projectName>/<moduleName>-guide.html` |
| HTML (en) | `~/docs/<projectName>/<moduleName>-guide-en.html` |
| HTML (th+en) | `~/docs/<projectName>/<moduleName>-guide-bilingual.html` |

HTML is standalone — all images embedded as base64, shareable immediately, no server needed.

---

## Command Examples

**Standard (use cache + rebuild HTML):**
```
/create-guide basket-circulation                          → English (cache all sources, new HTML)
/create-guide basket-circulation th                       → Thai
/create-guide basket-circulation th+en                    → Bilingual + toggle
```

**Capture (Playwright auto-screenshot):**
```
/create-guide basket-circulation --capture                → plan + capture PC+mobile where missing
/create-guide basket-circulation --capture --no-mobile    → capture PC only
/create-guide basket-circulation --capture --replan       → capture + re-ask plan for all pages
/create-guide basket-circulation --capture --recapture    → capture all images fresh
/create-guide basket-circulation th+en --capture          → bilingual + toggle + capture
```

**Skip flags (reduce tokens):**
```
/create-guide basket-circulation --capture --no-plan      → capture without planning (use existing extraCaptures)
/create-guide basket-circulation --capture --overview-only → capture overview PC+mobile only
/create-guide basket-circulation --capture --no-vision    → capture but skip vision analysis
/create-guide basket-circulation --no-code                → HTML from routes + cache (no source read)
/create-guide basket-circulation --html-only              → HTML from full cache (fastest)
```

**Shorthand / Combo:**
```
/create-guide basket-circulation --capture --lite         → capture + skip plan + overview + skip vision
/create-guide basket-circulation --capture --no-plan --no-vision  → quiet capture, no prompts, no vision
/create-guide basket-circulation th+en --html-only        → bilingual, build HTML from cache immediately
```

**Add images manually then rebuild HTML:**
```
/create-guide basket-circulation                          → new HTML + auto-pick new images (new filenames)
/create-guide basket-circulation --refresh-images         → re-run vision on all images + build HTML
                                                            (use when replacing images with same filename)
/create-guide basket-circulation --no-vision              → rebuild HTML fastest (no vision)
```

**Cache management:**
```
/create-guide basket-circulation --refresh-code           → re-read source code for all slugs
/create-guide basket-circulation --refresh-images         → re-run vision on all images (clear imageCache)
/create-guide basket-circulation --fresh                  → clear all cache + recapture everything
/create-guide receiving --capture                         → different module
```
