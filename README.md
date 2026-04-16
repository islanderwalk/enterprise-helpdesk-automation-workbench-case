# Enterprise IT Helpdesk Automation Workbench — Portfolio Evidence

> Target role reference: **本萬有限公司 (Benwan Co., Ltd.)** via 104 人力銀行
> Applicant: 梁瀚文 (Wen Liang)
> Package type: **Portfolio evidence repo** (public-safe adaptation of real internal work)

---

## Summary

I turn fragmented, ticket-driven internal IT operations into **batch-first, deterministic workflows** that an IT team can run, audit, and hand off safely. This package is the public-safe proof layer for work I have already shipped inside a Taiwan enterprise environment — covering email-driven account provisioning, helpdesk ticket automation, VPN / identity remediation, bulk Outlook communication, and AI-assisted classification.

The focus is not "I can write scripts." The focus is: I can take a messy day-to-day IT support process, break it into a workflow, attach clear ownership, and leave behind a system the next operator can keep running.

---

## The problem (before)

Inside a corporate IT / helpdesk team, repetitive user-facing work pulls a disproportionate share of engineer time:

- **Account lifecycle**: SSL-VPN, SVN, FTP, H2OIDE, Ivanti, Product Package — each system has its own form, its own approval flow, its own credential email.
- **Ticket follow-through**: each request spawned a manual email with manually attached PDFs, manually composed templates, and manually copied account IDs.
- **Inbox triage**: hundreds of bounce emails, training-completion confirmations, audit requests — all surviving on individual memory.
- **Environment fixes**: Teams sign-in loops, Pulse Secure stuck caches, OneDrive policy toggles, ESET cleanup — each one a manual runbook that lived in someone's head.

Result: inconsistent handling, repeated back-and-forth, single-person knowledge concentration, and no audit trail when things went wrong.

---

## The solution (what I built)

A layered workbench that treats user requests as **batches, not tickets**:

1. **Normalize** inbound requests (Outlook mailboxes, Excel workbooks, EIP2 / Ivanti exports) into a single row shape.
2. **Deduplicate and merge** before any external write, so no account gets double-provisioned.
3. **Decide** `create / skip / review` deterministically — the operator only touches the exception queue.
4. **Execute** via deterministic adapters (Outlook COM, pywin32, Playwright, Ivanti API, SSLVPN provisioning endpoints).
5. **Verify and readback** — every write is confirmed, every failure surfaces a bounded debug bundle.
6. **Generate communication artifacts** (confirmation emails, PDFs, meeting invites) only from verified results.
7. **Observe** — execution summary, telemetry, replayable artifacts kept in private environments only.

---

## What was delivered (highest-impact items)

- **InQuire 3.0** — PySide6 desktop shell orchestrating four real domain workflows (Product Package, SYS SSLVPN, Ivanti Local User, WPM meeting). 587/587 tests green on the sibling 104 codebase; InQuire 3.0 has its own 90+ integration-test bank. *Private source; sanitized derivative available.*
- **AI-Based Email Behavior Classification Engine** — AI + few-shot + firewall-immunity pattern matcher that safely buckets SSL-VPN bounce emails into `DELETE / REVIEW / KEEP` and prevents over-aggressive account deletion.
- **EIP2_report** — Five-stage pipeline (Cerberus FTP export → EIP2 form fetch → merge → Outlook draft generation → batch form sign-off) that collapses a 20-step manual provisioning task into one operator run.
- **outlook_file_download** — Tkinter + Tesseract OCR + Outlook COM tool that extracts training-certificate images from Outlook, fuzzy-matches course names, and auto-fills the enrollment roster Excel.
- **Confirmation email suite (7 systems)** — VBA macros for FTP / InQuire / ITS / Product Package / SVN / VPN_Internal / WPM, covering bulk credential distribution with HTML templates, multi-PDF attachments, and BCC-chunking (200-per-draft) to avoid SMTP limits.
- **Helpdesk first-aid toolkit** — `Teams_Full_Reset.bat`, `fix_pulse_secure.bat`, `OneDrive_off.bat`, `eset_cleanup_deep.bat` — reusable one-click fixes that replace tribal knowledge runbooks.
- **Enterprise Provisioning Workbench Demo** — already-published sanitized public derivative at `../enterprise-provisioning-workbench-demo/`.

---

## Why it matters operationally

- Repeated handling dropped from "one engineer, one ticket, one copy-paste" to **batch runs with an exception queue**.
- Account provisioning is **deterministic and auditable** — every decision is `create / skip / review` with a reason string.
- New operators can be onboarded from docs, not from shadowing — handoff is real.
- AI layer assists extraction and classification, but **never performs the final system write**. Safety boundary is explicit.

---

## Tech stack (what I actually used)

- **Languages**: Python 3.11 (main), VBA (Outlook/Word/Excel), PowerShell, Windows batch, TypeScript (Next.js for 104 side).
- **Desktop**: PySide6 / PyQt6, Tkinter, Win32 COM via `pywin32`.
- **Browser/UI automation**: Playwright (persistent context, Chromium), Selenium + webdriver-manager (legacy), Chrome DevTools Protocol over WebSocket.
- **AI / LLM**: OpenAI `gpt-4o-mini`, Anthropic Claude (RAG), `chromadb`, `sentence-transformers`.
- **HTTP**: `requests` + custom `RetrySession` (exponential back-off for Cloudflare 403).
- **Data**: SQLite (`applied_history.db`), `openpyxl`, `pandas`, CSV pipelines.
- **OCR**: Tesseract (`pytesseract`), OpenCV, Pillow.
- **Testing**: `pytest` (587/587 green on 104 side; 90+ integration tests in InQuire 3.0).
- **Observability**: structured JSON logs, telemetry snapshots, replay-friendly debug bundles.

---

## Repository guide

| File | What it is |
|------|-----------|
| [`CASE_STUDY.md`](CASE_STUDY.md) | Full narrative: background → problem → analysis → solution → challenges → result → handoff. |
| [`JD_ALIGNMENT.md`](JD_ALIGNMENT.md) | Direct mapping from the role's expectations to evidence in this repo. |
| [`TECH_STACK.md`](TECH_STACK.md) | Complete technology inventory with role, reason, and public visibility flag. |
| [`SYSTEM_OVERVIEW.md`](SYSTEM_OVERVIEW.md) | Before / after workflow view + component diagram. |
| [`SCREENSHOT_INDEX.md`](SCREENSHOT_INDEX.md) | What each screenshot shows, which claim it backs, redaction status. |
| [`SETUP.md`](SETUP.md) | How to run the sanitized demo path locally. |
| [`SANITIZATION_NOTES.md`](SANITIZATION_NOTES.md) | What was removed, renamed, mocked, or reconstructed — and why. |
| [`docs/`](docs/) | Workflow diagrams, handoff notes, process breakdown. |
| [`examples/`](examples/) | Sanitized sample input and output. |

---

## Notes on public-safe adaptation

This package is a **portfolio evidence repo**, not a full source dump. The underlying workflows are real and in production; internal hostnames, tenant IDs, credentials, customer identifiers, and raw HTML traces are deliberately excluded. See [`SANITIZATION_NOTES.md`](SANITIZATION_NOTES.md) for the full boundary.

A companion sanitized code repo exists at `../enterprise-provisioning-workbench-demo/` with runnable Python CLI + tests and a reproducible `create / skip / review` workflow.

## Links

- **This repo (case study + 5-question answer)**: <https://github.com/islanderwalk/enterprise-helpdesk-automation-workbench-case>
- **Sibling runnable demo**: <https://github.com/islanderwalk/enterprise-provisioning-workbench-demo>
- **Direct answer to 本萬 5 題**: [`FOR_BENWAN.md`](FOR_BENWAN.md)

## GitHub profile visibility

Visible repos (from public GitHub profile):

![GitHub profile — islanderwalk](assets/screenshots/00_github_profile_repos.png)

Shown: `enterprise-provisioning-workbench-demo` (public sanitized derivative), `InQuire-3.0` (private flagship source), `Outlook-Email-Behavior-Classification-Engine` (AI email triage), `BMC_BIOS_1213`, `EnglishAI-Toolkit`, `ai-short-video-mvp`, and profile repo `islanderwalk/islanderwalk`. Each maps to a section in [`TECH_STACK.md`](TECH_STACK.md) or [`CASE_STUDY.md`](CASE_STUDY.md).
