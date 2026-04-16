# How to Publish This Portfolio Package

Short, copy-pasteable steps to take `portfolio_package/` from local filesystem to a public GitHub repo named `enterprise-helpdesk-automation-workbench-case`, then reply to the 本萬 conversation.

Target GitHub profile: **`islanderwalk`** (confirmed from screenshot 2026-04-15).
Sibling repo already public: **`islanderwalk/enterprise-provisioning-workbench-demo`**.

---

## Step 0 — Crop and save the GitHub-profile screenshot

The screenshot you shared shows two Chrome tabs:
- ✅ `GitHub` tab — useful
- ❌ `Insyde - 6.8helpdesk_troubleshoo...` tab — **must be cropped out** before publishing (it identifies the employer)

Crop the image so only the GitHub viewport is visible. Save the cropped image to:

```
C:\Users\wen.liang\Documents\IT Automation Suite\104\portfolio_package\assets\screenshots\00_github_profile_repos.png
```

Quick crop options:
- **Windows Snipping Tool** (`Win + Shift + S`) on the already-captured screenshot, save with that filename.
- **Paint** → open → select → crop → save as PNG.

---

## Step 1 — Prepare the repo locally

```powershell
cd "C:\Users\wen.liang\Documents\IT Automation Suite\104\portfolio_package"

git init -b main
git add .
git status   # sanity check: .gitignore hides docs/104_benwan_* and docs/reply_to_benwan.md and docs/_frame_analysis_*.md

git commit -m "Initial portfolio evidence package"
```

**Before committing, verify the gitignore kept the private files out:**

```powershell
git ls-files | Select-String -Pattern "104_benwan|reply_to_benwan|_frame_analysis"
```

If that returns anything other than empty, stop — edit `.gitignore` or `git rm --cached <file>` until it's empty.

---

## Step 2 — Create the GitHub repo

### Option A — via `gh` CLI (if installed and authed)

```powershell
gh repo create islanderwalk/enterprise-helpdesk-automation-workbench-case `
    --public `
    --description "Public-safe evidence layer for enterprise IT helpdesk / Outlook / AI email automation. Sibling to enterprise-provisioning-workbench-demo." `
    --source . `
    --push
```

### Option B — via web UI

1. Go to <https://github.com/new>.
2. Owner: `islanderwalk`
3. Repository name: `enterprise-helpdesk-automation-workbench-case`
4. Visibility: **Public**
5. Do **not** initialize with README / .gitignore / license (you already have them).
6. Click Create.
7. Then push from local:

```powershell
git remote add origin https://github.com/islanderwalk/enterprise-helpdesk-automation-workbench-case.git
git push -u origin main
```

---

## Step 3 — Confirm the public view

Open the new repo in a browser. Expected landing:

```
enterprise-helpdesk-automation-workbench-case
├── README.md           ← renders with intro + screenshot + repo guide
├── CASE_STUDY.md       ← full narrative (14 sections)
├── JD_ALIGNMENT.md     ← mapped to 本萬 JD + recruiter 5 questions
├── FOR_BENWAN.md       ← direct 5+Q6 answer
├── SYSTEM_OVERVIEW.md
├── TECH_STACK.md
├── SCREENSHOT_INDEX.md
├── SETUP.md
├── SANITIZATION_NOTES.md
├── MISSING_ITEMS.md
├── AGENT_REPORT.md
├── HOW_TO_PUBLISH.md   ← this file
├── .gitignore
├── docs/
│   ├── workflow-diagram.md
│   ├── handoff-notes.md
│   └── process-breakdown.md
├── assets/
│   └── screenshots/
│       └── 00_github_profile_repos.png  ← cropped
└── examples/
    ├── sanitized_input/requests.json
    └── sanitized_output/run-report.json
```

Verify the `00_github_profile_repos.png` renders inside README and `FOR_BENWAN.md`.
Click each top-level `.md` and skim — should be no broken links, no `<your-handle>`, no internal hostnames.

---

## Step 4 — Reply to 本萬 on 104

Open `docs/reply_to_benwan.md` locally (it is **not** in the public repo). Copy the message body inside the code block labelled "最終建議版". The two GitHub URLs in that body are already set to `islanderwalk`.

Paste into 104 訊息中心 → 本萬有限公司 chatroom `8qf2c`.

Send.

---

## Step 5 — Post-publish watch items

- Do **not** push again without re-running the gitignore check (Step 1's `git ls-files | Select-String ...`). Future changes to `FOR_BENWAN.md` are safe; future changes to anything under `docs/` need verification.
- If the recruiter replies, see `docs/reply_to_benwan.md` §「如果他回 X，你回 Y」for the 4 branches already prepared.

---

## Appendix — If you prefer to merge this into the existing demo repo instead

Some reviewers prefer one link over two. If you'd rather drop this into the existing `enterprise-provisioning-workbench-demo` repo as a `/case-study/` folder:

```powershell
cd "<path to local clone of enterprise-provisioning-workbench-demo>"
git pull
mkdir case-study
Copy-Item -Recurse "C:\Users\wen.liang\Documents\IT Automation Suite\104\portfolio_package\*" case-study\
# Then manually merge .gitignore entries
git add case-study
git commit -m "Add enterprise helpdesk automation case study (5-question answer, JD-aligned)"
git push
```

In that case, update the GitHub URLs inside `docs/reply_to_benwan.md` to point to `/tree/main/case-study/FOR_BENWAN.md` instead of a separate repo.

Either approach is fine. Separate-repo is the default in this doc because it gives cleaner independent commit histories.
