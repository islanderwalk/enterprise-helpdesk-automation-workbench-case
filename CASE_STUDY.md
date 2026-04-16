# Case Study — Enterprise IT Helpdesk Automation Workbench

> Consolidated portfolio case study based on real internal work shipped across the IT Automation Suite. Public-safe reconstruction. Flagship modules: **InQuire 3.0**, **AI Email Behavior Classification Engine**, **EIP2_report**, **outlook_file_download**, and the **Confirmation Email Suite**.

---

## 1. One-line summary

I built and structured a desktop-oriented automation workbench and a set of supporting tools that turned scattered, ticket-driven IT helpdesk handling — VPN provisioning, credential distribution, training-certificate triage, meeting scheduling, and endpoint remediation — into a repeatable, auditable batch workflow that another operator can maintain.

---

## 2. Background

The environment is a Taiwan enterprise IT team supporting several thousand internal users across engineering and operations. The team handles a steady stream of recurring requests:

- SSL-VPN (InQuire / SYS SSLVPN) account creation, renewal, deletion.
- Product Package access for new software users (SVN + H2OIDE + tool bundle).
- Ivanti Local User imports from HR input.
- FTP, SVN, ITS, WPM account provisioning.
- Training-certificate collection for information-security compliance.
- Endpoint first-aid tickets: Teams sign-in loops, Pulse Secure stuck VPN, OneDrive policy, ESET residue.
- Monthly and quarterly reporting to management.

Most requests arrive through Outlook, an internal EIP2 web portal, or an Excel spreadsheet copy-pasted from HR. They look similar on the surface but require different downstream handling.

---

## 3. Original problem

The main issue was not just "too many tickets." It was that **similar-looking requests required different handling paths**, and every path depended on individual memory.

Concrete pain points observed in day-to-day work:

- A single operator was the canonical source of "how do you run the monthly SSL-VPN renewal?"
- Credential emails were composed one at a time; PDF attachments were manually dragged in; BCC lists were hand-grouped.
- Bounce emails from SSL-VPN health checks were triaged by eyeballing — a legitimate firewall block looked identical to an expired account, so accounts got incorrectly deactivated.
- Monthly Ivanti and EIP2 reports were a 30-minute point-and-click grind with no guarantee of consistency.
- Outlook training-completion emails contained certificate images; extracting course name + employee name was done by opening 100 emails by hand.
- Teams / Pulse Secure / OneDrive tickets were re-diagnosed from scratch every time.

**Cost**: hours per week of repeated manual handling, inconsistent record keeping, occasional wrong-account deletions, and no way to hand off the knowledge.

---

## 4. Why the old process was inefficient

- **Ambiguity**: bounce emails, firewall blocks, DNS failures, and expired accounts all produced "delivery failure" — a single surface appearance hid four different decisions.
- **No normalization step**: every request came in a different shape (Outlook email, EIP2 form, Excel row, inbound ticket), and the operator mentally translated each one into "what system do I touch?"
- **No dedup step**: if the same user appeared in two concurrent requests, both paths ran — sometimes creating duplicate accounts.
- **No readback**: after provisioning, there was no automatic verification that the account actually existed on the target system.
- **Knowledge concentration**: the "how" lived in one person's head; no new hire could pick it up from docs alone.
- **No audit trail**: when something broke, reconstructing "what happened" required re-reading hundreds of emails.

---

## 5. My analysis and workflow breakdown

I separated the workflow into five phases and forced each phase to produce an inspectable artifact:

| Phase | Input | Output | Why it matters |
|-------|-------|--------|----------------|
| **Intake** | Outlook folder / Excel / EIP2 export | Raw request rows | Single source of truth |
| **Normalize** | Raw rows | Uniform request shape (user, system, action, source) | Eliminates per-system handling code |
| **Merge** | Normalized rows | Deduplicated rows, with merge reason | No double-provision |
| **Decide** | Merged rows | `CREATE / REVIEW / SKIP` + reason string | Operator only touches exceptions |
| **Execute + verify** | Decided rows | Provisioned account + readback confirmation + draft email | Deterministic, auditable |

Around this spine, I attached:

- a **classification layer** (firewall-immunity + AI few-shot) for ambiguous bounce emails;
- an **OCR + fuzzy-match layer** for training-certificate extraction;
- a **communication layer** (VBA + Outlook COM) for bulk credential delivery;
- a **helpdesk-chat layer** (Claude RAG over internal KB) for the long-tail user questions.

The AI layer is explicitly positioned as **assist**, not **decide** — no LLM call ever triggers a system write directly.

---

## 6. Solution design

I chose a **desktop-first batch workbench** over a web dashboard or a bot framework.

Reasoning:

- Most of the target systems (Outlook, Ivanti admin UI, Cerberus FTP, SSL-VPN admin panel) are Windows-bound COM / pywin32 / Playwright surfaces. A desktop shell is the least-friction host.
- Operators are Windows users with Outlook open; a desktop app fits their workflow.
- Batch-first UX means the default run processes many rows; exceptions bubble up as a review queue. Single-row handling is the fallback, not the default.
- Deterministic adapters own all external writes — UI layers never directly touch a connector.
- The demo companion (`enterprise-provisioning-workbench-demo`) is a runnable Python CLI with the same shape, so the public story and the private implementation match.

---

## 7. What I actually delivered

### 7.1 InQuire 3.0 — the flagship workbench

- **Shell**: PySide6 tabbed desktop app with launcher, status panel, and execution logs.
- **Domains** (`jobs/`): Product Package ETL, SYS SSLVPN batch import, Ivanti Local User import, WPM meeting scheduler.
- **Services**: helpdesk AI chat (Claude + `chromadb` RAG over internal KB), OpenAI/Claude routing via `infrastructure/llm_gateway.py`, credential store.
- **Testing**: 90+ integration tests with safe fixtures; brainstorm / plan / implementation docs per milestone.
- **Private source of truth** at `InQuire 3.0/`; sanitized derivative at `enterprise-provisioning-workbench-demo/`.

### 7.2 AI-Based Email Behavior Classification Engine

- PowerShell export (`Export_SSLVPN.ps1`) pulls SSL-VPN bounce emails from Outlook via MAPI.
- Python analyzer (`analyze_emails.py`) runs a two-layer filter: (1) regex **firewall-immunity** patterns catch spam-filter / IP-block / DNS failures and protect those accounts from deletion; (2) OpenAI few-shot matching against golden sample emails classifies the rest.
- Excel report (`SSLVPN_Account_Review_Final.xlsx`) buckets accounts into `DELETE / REVIEW / KEEP` with a justification per row.
- Strict Mode adds a second safety pass before recommending `DELETE`.

### 7.3 EIP2_report — the 5-stage provisioning pipeline

- `1.fetch_cerberus_list.py` exports the current FTP account list from Cerberus.
- `2.fetch_EIP2_FSR_list.py` pulls pending FTP service requisition forms from EIP2.
- `3.process_ftp_requests.py` merges the two feeds, generates password candidates, and identifies `create / reset / noop` rows.
- `4.create_mail_drafts.py` composes Outlook drafts (HTML template + credential rows).
- `5.sign_off_forms.py` batch-signs the EIP2 approval forms via Selenium.

Configuration is externalized (`config_local.py` pattern + env vars); no credentials in source.

### 7.4 outlook_file_download — OCR + fuzzy-match enrollment tool

- Tkinter GUI with calendar date picker, Outlook folder selector, and one-click "download + OCR + fill Excel" action.
- Tesseract OCR on embedded certificate images; multi-threshold fuzzy match (0.85 / 0.95 / 0.78) against the course catalog; primary-domain alias handling for employee emails.
- Auto-populates the `上課名單` (enrollment roster) Excel with the normalized course name + employee ID.

### 7.5 Confirmation email suite (VBA)

One macro per system — FTP, InQuire, ITS, Product Package, SVN, VPN_Internal, WPM — plus the BCC-batching `outlook 批次寄信 (SSLVPN.自動批次每200封)` that chunks large recipient lists into 200-per-draft to avoid SMTP cutoff. HTML-encoded bodies, signature preservation, multi-PDF attachment loops, and (for WPM) DST-aware meeting scheduling.

### 7.6 Helpdesk first-aid toolkit (Windows)

- `Teams_Full_Reset.bat` — kills Teams/Office sign-in stack, clears AAD/WAM/OneAuth caches, removes Credential Manager entries, runs `dsreg` cleanup.
- `fix_pulse_secure.bat` — five-step Pulse Secure / Ivanti Secure Access cache recovery with auto-elevation and per-step logging.
- `OneDrive_off.bat` / `OneDrive_on.bat` — policy-driven OneDrive toggle with HKLM registry writes.
- `eset_cleanup_deep.bat` — deep ESET removal with scheduled retry and explicit exit codes.

### 7.7 Enterprise Provisioning Workbench Demo

Already published at `enterprise-provisioning-workbench-demo/` — runnable Python CLI, sample JSON input, Mermaid architecture diagram, unit tests, interview story, sanitization checklist.

---

## 8. Technologies used

| Technology | Used for | Why it mattered |
|---|---|---|
| Python 3.11 | Core automation, orchestration, AI layer | Windows-compatible, rich COM bindings, mature AI SDKs |
| PySide6 / PyQt6 | InQuire 3.0 desktop shell | Tabbed multi-domain operator workspace |
| Tkinter | outlook_file_download lightweight GUI | Zero dependency, fast to ship |
| `pywin32` + Outlook COM | Mailbox access, draft generation, folder walk | The only reliable path to Outlook state |
| Playwright (persistent context) | EIP2, Google Form, SSL-VPN admin UI automation | Modern, resilient, session-aware |
| Selenium + webdriver-manager | Legacy EIP2 form paths | Existing coverage where migration cost > benefit |
| OpenAI `gpt-4o-mini` | Email classification, cover-letter assist | Cheap, fast, good-enough for extraction |
| Claude + `chromadb` + `sentence-transformers` | Internal helpdesk RAG chat | Safer on long-context, better at refusal |
| Tesseract + OpenCV + Pillow | Certificate OCR in training emails | Free, offline, predictable |
| `openpyxl` + `pandas` | Excel I/O, formula translation, deduplication | Every target system touches Excel somewhere |
| SQLite (`applied_history.db`) | Idempotency, audit trail | File-based, zero-ops |
| VBA / Outlook / Word COM | Confirmation email templates, bulk delivery | Matches existing operator habit |
| PowerShell | Scheduled-task registration, MAPI export, elevation | Native Windows scheduler integration |
| `pytest` (587/587 green on 104 side; 90+ on InQuire 3.0) | Regression safety net | Gives confidence to keep shipping |
| FastAPI + Next.js 16 | 104 side: API + operator web UI | Showcase full-stack capability |

---

## 9. Challenges encountered

### Challenge 1 — Ambiguous bounce emails

**Problem.** SSL-VPN health-check bounces looked similar whether the root cause was an expired account (safe to delete), a firewall block (must preserve), spam-filter rejection (must preserve), DNS failure (must preserve), or mailbox full (must preserve). A naive "delete all bouncers" rule would nuke live users.

**Action taken.** Built a two-layer classifier: first, a set of regex **firewall-immunity** patterns that force-classify known-benign bounce shapes into `KEEP`; second, an OpenAI few-shot match against a `samples/` golden set. Added Strict Mode — any `DELETE` must pass both layers.

**Result / tradeoff.** False-positive deletions eliminated for the known patterns. Tradeoff: the golden-sample set needs periodic refresh when mail-server wording changes. Documented refresh runbook; quarterly review scheduled.

### Challenge 2 — Credential hygiene

**Problem.** Early scripts hardcoded OpenAI keys and internal passwords. Discovered during the portfolio audit (`EIP2_ITS` held plaintext admin passwords; `AI Email Classification` held a live `sk-proj-…` key in `key.txt`).

**Action taken.** Established the `config_local.py` + `.env` pattern already used in `EIP2_report`; propagated it backwards. Rotated the exposed OpenAI key. Added credential leak scanning to the sanitization checklist.

**Result / tradeoff.** All current scripts read credentials from environment or an ignored local config. Tradeoff: the older commit history still contains the leaked key shape; rotating the key is sufficient remediation but the historical commit can never be fully disavowed.

### Challenge 3 — DST + Outlook appointment overlap

**Problem.** WPM weekly meetings broke across daylight-saving transitions; Outlook sometimes double-booked rooms when two macros wrote appointments concurrently.

**Action taken.** Added timezone-aware datetime construction (`python-dateutil`), scheduled-appointment pre-check via COM enumeration, and a "single-writer lock" (file-based) around the macro run.

**Result / tradeoff.** No DST regressions for two cycles. Tradeoff: the file lock is cooperative — a user who force-runs the macro outside the launcher can still collide. Documented; acceptable for the ops size.

### Challenge 4 — Public-safe reconstruction without looking hollow

**Problem.** The most impressive work (InQuire 3.0) is deeply internal. A straight source dump would leak hostnames, tenant IDs, real user emails, and internal workflow details. But an over-sanitized public repo would look like a skeleton.

**Action taken.** Created a companion `enterprise-provisioning-workbench-demo/` — a runnable Python CLI with the same architectural shape (normalize → merge → decide → execute → notify), with fake adapters and sample JSON. Kept the private `InQuire 3.0/` repo as the source of truth; documented the sanitization boundary explicitly.

**Result / tradeoff.** The public repo is reviewable end-to-end (`python -m demo_app --show-stages` works). Tradeoff: the public demo shows the *pattern*, not the *scale*. I address that gap in the interview narrative and in this case study.

### Challenge 5 — Chinese-English mixed course names in OCR

**Problem.** Course certificates had titles like "資訊安全意識 / Information Security Awareness" — OCR mis-read mixed scripts, and straight equality matching failed on every entry.

**Action taken.** Added per-course fuzzy-match thresholds (0.78–0.95 depending on course name variance), an "early stopping" heuristic for multi-strategy crop variants, and a primary-domain alias table so `user@insyde.com` and `user@insyde.onmicrosoft.com` resolve to the same employee.

**Result / tradeoff.** Extraction hit rate moved from ~60% (string-match) to >95% on a sample of 300 certificates. Tradeoff: confidence thresholds are empirical — a new course template may need one-off calibration.

---

## 10. Result / improvement

Even without precise formal metrics, the operational delta is visible:

- **Account provisioning**: previously ~20 manual touchpoints per request, now 1 operator run with a `create / review / skip` summary and auto-generated drafts.
- **Bounce-email triage**: previously unscoped; now a weekly report lands in the inbox with a risk-classified account list.
- **Training-certificate enrollment**: from "open 100 emails, copy-paste into Excel" to "pick a date range, click Run."
- **Helpdesk endpoint tickets (Teams / Pulse Secure / OneDrive / ESET)**: one-click `.bat` replaces a runbook that lived in tribal memory.
- **Confirmation email delivery**: BCC-chunked at 200, no SMTP cutoff, consistent template, correct signature.
- **AI helpdesk chat**: long-tail "how do I ..." questions handled by RAG over the internal KB before a ticket is created.
- **Handoff**: every module has a `README.md`, a `brainstorm.md`, a `plan.md`, and a test suite. A new operator can pick it up from docs alone.

---

## 11. Maintenance and handoff

I treated these tools as **a workflow that outlives me**, not a personal shortcut:

- Each module has an `AGENTS.md` / `CLAUDE.md` / `README.md` per-project, including command examples, known states, and rollback paths.
- Tests are the regression safety net: 587/587 on the sibling 104 codebase, 90+ on InQuire 3.0.
- All credentials live in environment variables or ignored local configs, never in source.
- Logs and telemetry write to dated JSONL files so a replay of last Thursday's run is mechanical, not archaeological.
- The private → public split (`InQuire 3.0/` → `enterprise-provisioning-workbench-demo/`) means the architecture can be discussed externally without leaking internals.
- Brainstorm + plan docs (`*.brainstorm.md`, `*.plan.md`) sit next to the code, so the *why* of each change is preserved for whoever touches it next.

---

## 12. Relevance to the role

See [`JD_ALIGNMENT.md`](JD_ALIGNMENT.md) for the full JD mapping table.

Headline fit:

- **Real implementation experience** — not a tutorial stack; this is what I built while running real IT operations.
- **Workflow / process understanding** — five-phase normalize → merge → decide → execute → verify spine, applied consistently.
- **Integration / optimization** — Outlook + EIP2 + Ivanti + Cerberus + Playwright + AI glued into one batch run.
- **Ongoing maintenance** — test coverage, env-var credentials, brainstorm/plan docs, public-safe derivative.
- **Data organization / automation** — normalized inputs, deduped merges, audit-trailed outputs.

---

## 13. Public evidence available

- This portfolio package (you are reading it).
- `../enterprise-provisioning-workbench-demo/` — runnable Python CLI, tests, Mermaid diagram, interview story.
- Screenshots: see [`SCREENSHOT_INDEX.md`](SCREENSHOT_INDEX.md).
- Sanitized sample input/output: [`examples/`](examples/).
- Before / after workflow diagram: [`docs/workflow-diagram.md`](docs/workflow-diagram.md).

---

## 14. What I would improve next

- **Metrics capture**: land a telemetry pipeline that records `rows_processed / rows_created / rows_skipped / rows_reviewed` per run, so the operational impact is numeric, not narrative.
- **Scale-out path**: the desktop shell works for one operator; a thin FastAPI surface would let it host multi-operator or scheduled runs.
- **Expand AI-assist coverage** into the confirmation email templates — today they are static VBA templates; a LLM can tailor wording per user cohort while staying inside the "assist, not decide" boundary.
- **Automate sanitization**: today, preparing a public-safe snapshot is a manual checklist (`SANITIZATION_NOTES.md`). A CI step that scans for internal hostnames / tenant IDs / plaintext credentials before push would make the boundary enforceable.
- **Playwright → MCP**: replace some of the Playwright flows with MCP-driven agent calls for more resilient re-planning when target UIs shift.

---

# Short Version for HR Message

1. This project addressed **repetitive, one-by-one internal IT helpdesk work** — account provisioning, credential email, certificate triage, endpoint remediation.
2. The main issue was that **similar-looking requests required different handling paths, and the knowledge lived in one person's head**.
3. I broke it down by forcing every workflow through a **normalize → merge → decide → execute → verify** spine with explicit `create / skip / review` outcomes.
4. I delivered a **desktop workbench (InQuire 3.0)** plus an **AI email classifier**, a **5-stage provisioning pipeline**, an **OCR-based training-enrollment tool**, and a **first-aid helpdesk toolkit** — all with tests and a public-safe derivative.
5. The result was **consistent, auditable batch handling that another operator can run from docs alone**, and a maintenance model (env-var credentials, brainstorm/plan docs, sanitized public showcase) that is explicitly designed for handoff.

---

# GitHub README Summary Version

I turn fragmented, ticket-driven internal IT work into **batch-first, deterministic workflows** an operator can run, audit, and hand off. This repo is the public-safe evidence layer for a production automation workbench I built inside a Taiwan enterprise — covering Outlook-driven account provisioning across seven systems, AI-assisted bounce-email classification, OCR-based training enrollment, VBA bulk communication, and a Windows first-aid toolkit. The companion runnable demo at `../enterprise-provisioning-workbench-demo/` shows the same architecture end-to-end with sample data.
