---
description: Install CLAUDE-SKILL skills into ~/.claude/commands/ automatically — run in a project that has CLAUDE-SKILL/ folder
---

# 📦 Installation Methods

You can install skills from `CLAUDE-SKILL/` in 2 ways:

## Option 1: Manual — Copy Files Yourself
For those who prefer direct file management:
1. Find `CLAUDE-SKILL/` folder in your project
2. Copy any `.md` files you want
3. Paste into `~/.claude/commands/` folder
   - **Windows**: `C:\Users\<YourUsername>\.claude\commands\`
   - **macOS/Linux**: `~/.claude/commands/`
4. Create the folder if it doesn't exist yet
5. That's it — the command is ready to use immediately

## Option 2: Automated — Use `/install-guide`
For hands-off installation with verification:
1. Open your project in Claude Code
2. Run: `/install-guide` (installs all skills) or `/install-guide create-guide` (installs specific skill)
3. Follow the prompts (e.g., choose default language for `create-guide`)
4. Done — all files copied + verified + summary shown

---

# Automated Installation Details

Install skill files from `CLAUDE-SKILL/` in the current project into `~/.claude/commands/` automatically.

**$ARGUMENTS** — name of the skill to install, e.g. `create-guide`
If not specified → install all `.md` files in `CLAUDE-SKILL/` (except `install-guide.md` itself)

---

## Steps — run all steps in sequence without stopping to ask

### Step 1 — Verify CLAUDE-SKILL/

Use Glob tool to find `CLAUDE-SKILL/*.md` in the current working directory.

If not found → display:
```
❌ CLAUDE-SKILL/ not found in this working directory
   Please cd to the folder that contains CLAUDE-SKILL/ first
```
Then stop. Do not continue to next step.

### Step 2 — Identify skill files to install

```
If $ARGUMENTS has a skill name → install only  CLAUDE-SKILL/<skillName>.md
If $ARGUMENTS is empty         → install all   CLAUDE-SKILL/*.md
                                  except install-guide.md
```

### Step 3 — Handle create-guide language versions

**Only when** the skill list from Step 2 includes `create-guide` OR the CLAUDE-SKILL/ folder contains `create-guide-th.md` or `create-guide-en.md`.

**3-A — Check which language files exist:**
```
found = Glob CLAUDE-SKILL/create-guide-th.md  → hasTH = true/false
found = Glob CLAUDE-SKILL/create-guide-en.md  → hasEN = true/false
```

**3-B — Install both language files as separate commands (always):**
- If `hasTH = true` → Read `CLAUDE-SKILL/create-guide-th.md` → Write to `commands/create-guide-th.md`
- If `hasEN = true` → Read `CLAUDE-SKILL/create-guide-en.md` → Write to `commands/create-guide-en.md`

This gives the user two permanent commands:
- `/create-guide-th <module>` — always runs the Thai prompt
- `/create-guide-en <module>` — always runs the English prompt

**3-C — Ask which version becomes the default `/create-guide`:**

Only ask if BOTH `hasTH` and `hasEN` are true. Display:

```
╔══════════════════════════════════════════════════════╗
║  🌐  Default language for /create-guide              ║
╚══════════════════════════════════════════════════════╝

  Both language versions are installed as separate commands:
    /create-guide-th  — Thai prompt
    /create-guide-en  — English prompt

  Which should /create-guide (no suffix) point to?

  [1] 🇹🇭 ภาษาไทย  (create-guide-th)
  [2] 🇬🇧 English   (create-guide-en)

  Choose [1/2] :
```

- Accept only `1` or `2`
- `1` → copy `create-guide-th.md` content → Write to `commands/create-guide.md`
- `2` → copy `create-guide-en.md` content → Write to `commands/create-guide.md`

If only one language file exists → use that one as default automatically without asking.
If neither exists but `create-guide.md` exists → install it as default.

### Step 4 — Find commands folder path

```
Windows : %USERPROFILE%\.claude\commands\
macOS   : ~/.claude/commands/
Linux   : ~/.claude/commands/
```

Check with Bash or PowerShell whether the folder exists. If missing → create automatically:

**Windows:**
```powershell
New-Item -ItemType Directory -Force "$env:USERPROFILE\.claude\commands"
```

**macOS/Linux:**
```bash
mkdir -p ~/.claude/commands
```

### Step 5 — Install all other skill files

For each skill in the list that is NOT `create-guide`, `create-guide-th`, or `create-guide-en`:
1. Read with **Read tool** from `CLAUDE-SKILL/<skillName>.md`
2. Write with **Write tool** to `commands/<skillName>.md`
3. Overwrite if already exists — no confirmation needed

### Step 6 — Show summary

Display in this format:

```
✅ CLAUDE-SKILL installation complete

   commands folder: C:\Users\<username>\.claude\commands\

   create-guide installed:
     ✅ /create-guide-th  → commands\create-guide-th.md  (Thai prompt)
     ✅ /create-guide-en  → commands\create-guide-en.md  (English prompt)
     ✅ /create-guide     → commands\create-guide.md     (default: Thai ← points to create-guide-th)

   other skills:
     ✅ <skill-name>  → commands\<skill-name>.md

   How to use:
     /create-guide-th <moduleName> --capture    ← Thai prompt, Thai output (default)
     /create-guide-en <moduleName> --capture    ← English prompt, English output (default)
     /create-guide    <moduleName> --capture    ← uses the default version chosen above

     Override output language anytime (works with both TH and EN prompts):
     /create-guide-th <moduleName> en           ← Thai prompt → English HTML output
     /create-guide-en <moduleName> th           ← English prompt → Thai HTML output
     /create-guide-th <moduleName> th+en        ← Thai prompt → bilingual HTML output
```

- If any skill fails → show `❌ <skillName> — <reason>` but continue installing remaining skills
