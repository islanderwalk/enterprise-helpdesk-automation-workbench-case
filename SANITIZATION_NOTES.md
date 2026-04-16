# Sanitization Notes

Honest documentation of what has been removed, renamed, mocked, or reconstructed in this portfolio package — and why. This file exists to make the public-safe boundary **enforceable and auditable**, not to hide work.

---

## Sanitization policy

The public evidence layer must not contain:

- internal company hostnames or URLs (`eip2.*`, `vpn.*`, `sys.*`, `ftp.*`, or any specific subdomain of a private infrastructure)
- customer / client names
- real account names, employee numbers, internal email addresses
- API keys, OAuth tokens, session cookies, SMB / VPN credentials
- tenant IDs, Azure / AAD GUIDs, customer org identifiers
- raw HTML captures that could be used to reverse-engineer an internal admin UI
- proprietary schemas, internal form GUIDs, or private workflow names that are specific enough to identify the employer
- production debug bundles or telemetry payloads

When a piece of the story requires one of the above, it is replaced with:

- a generic placeholder (`<internal hostname>`, `alice.chen`, `existing-bob.wu`)
- a redacted screenshot (solid black rectangle over the sensitive region)
- a mock workbook with synthetic rows
- a sanitized / pseudocode excerpt
- a redrawn diagram that shows the shape without the identifiers

---

## What was removed from this public package

| Item | Where it lived | Why removed | Replacement |
|------|----------------|-------------|-------------|
| Internal EIP2 / VPN / SYS / FTP hostnames | `EIP2_ITS/*.py`, `EIP2_report/*.py`, launcher configs | Reveals employer infrastructure | `<eip2_host>`, `<vpn_host>`, `<sys_host>` placeholders when referenced in the case study |
| Hardcoded plaintext passwords | `EIP2_ITS/0.python auto_its_full.py`, `EIP2_ITS/2.iventi.py` | Real credentials in source — leak risk | Rotated; pattern migrated to `config_local.py` + env vars; plaintext values not reproduced anywhere |
| OpenAI API key | `AI-Based Email Behavior Classification Engine/.../key.txt` | Live API key was committed | Key was rotated immediately after audit; no key appears in this public package |
| Real SSL-VPN bounce-email samples | `AI Email Classification Engine/samples/` | Contains real user email addresses | Not reproduced; the pattern (firewall-immunity + few-shot) is described instead |
| Ivanti / Cerberus / Product Package response payloads | Various `debug_*.html`, `debug_*.json`, `_tmp_*` files | Could be used to reverse-engineer the internal admin UI | Not reproduced; the adapter interface is described instead |
| Real employee / course lists from training-certificate emails | `outlook_file_download/` configs | PII | Substituted with synthetic names in the examples folder |
| Real BCC recipient lists | `outlook 批次寄信 (SSLVPN.自動批次每200封)/` | 10k+ real email addresses | Not reproduced; the 200-per-draft chunking pattern is described only |
| Customer identifiers referenced in InQuire 3.0 docs | `InQuire 3.0/docs/` | Customer privacy | Not reproduced; private repo only |
| Raw HTML from reverse-engineering EIP2 / Ivanti UIs | `_tmp_*`, `debug_*.html`, `*_innerText.txt` | Reveals internal UI structure | Not reproduced |
| Real cookies / session tokens | `decoded_cookies.txt`, `cookies.pkl` | Auth material | Not reproduced |
| Production telemetry payloads | `data/*.jsonl`, `data/*.json` (many files) | Operational data | Not reproduced; the telemetry *shape* is described in `SYSTEM_OVERVIEW.md` |

---

## What was renamed

| Original (private) | Public name in this package | Why |
|---|---|---|
| Specific employer name | "a Taiwan enterprise IT team" | Keeps the story credible without identifying |
| Real domain names in examples | `example.com`, `example.tw` | Standard placeholder domains |
| Real employee names in sample rows | `alice.chen`, `bob.wu`, `carol.lin`, `existing-bob.wu` | Phonetic placeholders consistent across demo |
| Real customer / project codenames | "Product Package", "SYS SSLVPN", "WPM" | These are *generic enough* to describe the workflow without identifying; kept as-is, verified safe |
| Real Ivanti tenant ID | `<tenant>` | Prevents pivot to the tenant |

---

## What was mocked

- **Sample input data** (`examples/sanitized_input/requests.json`) — 3–4 synthetic requests covering the `create / review / skip` decision branches.
- **Sample output** (`examples/sanitized_output/run-report.json`) — what the planner produces from the mocked input.
- **Fake provisioning adapters** in `../enterprise-provisioning-workbench-demo/demo_app/` — return canned responses so the demo runs end-to-end without touching any real system.
- **Sample training certificate** (if captured) — synthetic PNG with a made-up course name and made-up employee name, built specifically for the OCR demo screenshot.

---

## What was reconstructed

- **Case study narrative** (`CASE_STUDY.md`) — real work, public-safe retelling. The problems, the analysis, the decisions, and the tradeoffs are all real; the hostnames, tenant names, and real user data are stripped.
- **Before / after workflow diagram** (`SYSTEM_OVERVIEW.md` + `docs/workflow-diagram.md`) — matches the actual architecture of the private implementation, drawn without internal system names.
- **Tech stack table** (`TECH_STACK.md`) — derived from three parallel deep-scans of the IT Automation Suite sub-projects + `data/tech_stack_inventory.json`. Every entry is backed by evidence.

---

## What was **not** sanitized because it's already safe

- Generic Python / VBA / PowerShell patterns (regex, retry-with-backoff, OCR fuzzy match, BCC chunking, registry writes, DST-aware datetime construction).
- Open-source dependency names (Playwright, Tesseract, `openpyxl`, `pywin32`, FastAPI, etc.).
- Architecture-level descriptions that do not reveal the employer.
- Generic Windows remediation logic (`Teams_Full_Reset.bat`, `fix_pulse_secure.bat`, etc.) — these target consumer-identifiable surfaces (Teams, Pulse Secure, OneDrive) that are generic to any Windows IT shop.

---

## How the sanitization gate is applied

Before any file in this package (or a supporting screenshot) is published:

1. Grep the file for the internal hostname patterns (documented in the private repo's sanitization checklist, not reproduced here).
2. Grep for real email addresses (`@<private domain>`).
3. Grep for plaintext credential shapes (`sk-proj-`, `password=`, `token=`, `cookie=`, etc.).
4. Visually inspect any screenshot for tab titles, usernames, system URLs.
5. If anything is in doubt, redact with a solid black rectangle or omit.

---

## Historical leak remediation

During the portfolio audit (three parallel deep-scans across the IT Automation Suite), the following credential leaks were discovered and addressed:

| Finding | Location | Action taken |
|---------|----------|--------------|
| Live OpenAI API key in `key.txt` | `AI-Based Email Behavior Classification Engine/` | **Rotated immediately**; `key.txt` moved to `.gitignore`; env-var pattern applied |
| Hardcoded admin password (EIP2) | `EIP2_ITS/0.python auto_its_full.py` | **Rotated**; migrated to `config_local.py`; value not reproduced anywhere in this public package |
| Hardcoded Ivanti password | `EIP2_ITS/2.iventi.py` | **Rotated**; same pattern |
| Real recipient lists (10k+) | `outlook 批次寄信 (SSLVPN.自動批次每200封)/` | Not exposed; macro lives on a private Outlook install only |

Historical git commits in the private repos may still contain the old key shapes. Rotation is sufficient remediation (the keys no longer work). Going forward, every new credential is env-only, and a pre-push scan (documented in the private repo) catches new leaks.

---

## What to re-check before each publication

- New screenshots added → re-run the visual inspection.
- New evidence files added → re-run the grep gate.
- New diagrams → redraw without internal names.
- New code snippets → sanitized, no real hostnames, no real IDs.
- Any change to the assumed-JD section in `JD_ALIGNMENT.md` → just a text swap, nothing sensitive touched.

---

## Trust signal

This file exists to make the sanitization work **inspectable**, not to perform virtue. If a reviewer wants to validate the boundary, the steps above are mechanical and reproducible. If something leaks despite this process, the remediation path is also mechanical: rotate, grep, replace, document.
