# Handoff Notes

What the next operator needs to keep these workflows running. Written as if a new hire walked in today and owned the tools tomorrow.

---

## First-week reading list

1. This package's `README.md` → `SYSTEM_OVERVIEW.md` → `CASE_STUDY.md`.
2. The companion public demo at `../enterprise-provisioning-workbench-demo/README.md` and `docs/interview-story.md`.
3. For each module being inherited: its own `README.md` + most recent `*.brainstorm.md` and `*.plan.md`.
4. The top-level `CLAUDE.md` and `AGENTS.md` inside each project repo — these are the authoritative instruction files for a Claude-assisted dev loop.

---

## Running things locally

| Module | Entry point | Virtual env? |
|---|---|---|
| Public demo | `python -m demo_app --show-stages` | Optional; stdlib only |
| 104 side | `cd 104 && .venv\Scripts\activate && python batch_weekend_apply.py --mode night_remote` | Yes — per-project `.venv` |
| InQuire 3.0 | `python main.py` from `InQuire 3.0/` | Yes |
| EIP2_report 5-stage | `1.fetch_cerberus_list.py` → `2.fetch_EIP2_FSR_list.py` → `3.process_ftp_requests.py` → `4.create_mail_drafts.py` → `5.sign_off_forms.py` | Yes |
| AI Email Classification | `Start_All.bat` (orchestrator) | No — `.bat` handles venv |
| outlook_file_download | `main_app.py` via `bootstrap_inquire3.ps1` | Yes |
| Confirmation-email macros | Open the VBA file in Outlook's VBA IDE, run the macro | N/A (Outlook VBA) |
| Helpdesk first-aid | Double-click the `.bat` (auto-elevates) | N/A |

---

## What breaks most often (known risk list)

### 1. Playwright session expiry

Symptom: EIP2 / Google Form flows hang on login.
Fix: Re-run `login_once.py` in the 104 project (or the equivalent per-project session-seed script).

### 2. OpenAI rate limit

Symptom: AI Email Classification stalls on large batches.
Fix: The analyzer has exponential back-off; for large catch-up runs, split the batch or run overnight.

### 3. Outlook COM null pointer

Symptom: `pywin32` call returns `None` mid-run.
Fix: Outlook restart; the COM model doesn't survive certain Outlook UI interactions. Long-running scripts should poll for health and reconnect.

### 4. Tesseract language pack missing

Symptom: `outlook_file_download` OCR returns empty strings.
Fix: Ensure the bundled Tesseract install (`outlook_file_download/Tesseract-OCR/`) has `eng`, `chi_tra`, and `chi_sim` language packs.

### 5. VBA macro signature mismatch on new Outlook version

Symptom: Confirmation-email macros refuse to run after Outlook update.
Fix: Re-sign the project in VBA IDE; update the trust-center self-cert list.

### 6. SMB batch hits SMTP cutoff

Symptom: `outlook 批次寄信` macro BCC chunk gets bounced by the mail gateway.
Fix: The macro caps at 200-per-draft. If the gateway policy tightens, reduce to 100 in `Module1.bas` constant.

### 7. DST transition breaks WPM scheduling

Symptom: Weekly meeting offset by 1 hour after spring-forward / fall-back.
Fix: WPM macro now uses `python-dateutil` timezone arithmetic (see `CASE_STUDY.md §9 Challenge 3`). If a new recurrence pattern is added, test it across a DST boundary.

### 8. SSR (Strip-Send-Restore) duplicate emails

Symptom: `outlook_gateway.py` sends the same draft twice.
Fix: **Known issue documented in memory (`feedback_ssr_broken.md`)**. SSR pattern is broken — do not rely on it. Use the direct-send path and skip strip/restore.

---

## Things to preserve when refactoring

- **Normalize → merge → decide → execute → verify spine.** Every new workflow should plug into this shape. Don't fork the architecture.
- **Reason strings.** Every decision row carries a `reason`. Don't drop them when adding a new adapter.
- **Env-var credentials.** Never hardcode, even "temporarily". The historical leak audit (see `SANITIZATION_NOTES.md`) showed how those temporary values survive in git.
- **pytest as the release gate.** Don't ship a change that leaves the suite red. 587/587 on 104 side, 90+ on InQuire 3.0, plus `unittest discover` on the public demo.
- **Brainstorm → plan → implementation loop.** Before any non-trivial change, write a `*.brainstorm.md` next to the code, then a `*.plan.md`, then implement. Keeps the *why* searchable.

---

## What you can safely change

- UI cosmetics (button labels, column widths, table ordering).
- New adapters — add them as new files in `jobs/` or `adapters/`, don't touch the planner.
- New AI-assist tasks — add them as new files in `services/`, route their output into `normalize`, not into `execute`.
- Logging verbosity (documented env vars control this).

---

## Who to ping if stuck

- For the 104 side: memo is in `104/CLAUDE.md` and the per-repo `AGENTS.md`. Repo-bridge agents exist at `.claude/agents/`:
  - Architecture → `104-architect`
  - Code quality → `104-code-reviewer`
  - Sprint → `104-scrum-master`
- For InQuire 3.0: same pattern — `.claude/agents/` inside `InQuire 3.0/`.
- For the public demo: low-risk; it's stdlib-only and the tests are fast.

---

## Pre-publish checklist (copy-paste)

Before pushing any change to a public repo:

- [ ] `pytest tests --tb=short -q` is green.
- [ ] `python -m unittest discover -s tests` is green on the public demo.
- [ ] No hostnames matching internal patterns in any new file.
- [ ] No real email addresses in any new file or screenshot.
- [ ] No plaintext credentials (`sk-proj-`, `password=`, `token=`, `cookie=`) in any new file.
- [ ] `SANITIZATION_NOTES.md` updated if new surfaces were introduced.
- [ ] Commit message describes the *why*, not only the *what*.
