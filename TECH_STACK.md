# Technology Stack

Complete inventory of technologies actually used across the IT Automation Suite projects that feed this portfolio. Only technologies backed by real evidence are listed — no inflation.

"Public?" indicates whether the technology is shown in the public-safe evidence layer (this repo + `../enterprise-provisioning-workbench-demo/`). Items marked "Private only" live in the internal `InQuire 3.0/` repo and are referenced but not reproduced.

---

## Languages

| Technology | Role in project | Reason chosen | Public? |
|---|---|---|---|
| Python 3.11 | Main automation language — orchestration, AI, Outlook COM wrapping, Excel I/O, report generation | Mature Windows COM bindings, rich AI SDK ecosystem, zero-install `.venv` per project | Yes |
| VBA (Outlook / Word / Excel) | Bulk-mail templates, multi-PDF attachment loops, meeting-scheduling macros | Matches existing operator habits; no deployment needed — lives with Outlook | Yes (sanitized excerpts) |
| PowerShell 7 | Scheduled-task registration, MAPI export, elevation, Teams / OneDrive / Pulse Secure remediation | Native Windows scheduler, first-class access to AAD / WAM / registry | Yes |
| Windows Batch (`.bat`) | Launcher scripts and first-aid remediation (`Teams_Full_Reset.bat`, `fix_pulse_secure.bat`, `eset_cleanup_deep.bat`) | Zero dependency, runs on any domain-joined laptop | Yes |
| TypeScript (Next.js) | Operator web UI on the 104 side (not in scope for the 本萬 case but available as capability) | Modern React with App Router; pairs with FastAPI | Partial |

---

## Desktop / UI frameworks

| Technology | Role | Reason | Public? |
|---|---|---|---|
| PySide6 | InQuire 3.0 tabbed desktop shell (multi-domain operator workspace) | Official Qt6 bindings, commercial-friendly, strong signal/slot model | Private only (architecture described in demo) |
| PyQt6 + `pyqt6-webengine` | Legacy desktop surfaces | Already in place in earlier projects | Private only |
| Tkinter | Lightweight GUI for `outlook_file_download` (OCR + Excel auto-fill) | Zero dependency, ships with Python, fast to modify | Yes |

---

## Browser / UI automation

| Technology | Role | Reason | Public? |
|---|---|---|---|
| Playwright (persistent context, Chromium) | Modern web automation — EIP2 forms, Google Form booking, SSL-VPN admin panel, 104 scrape/apply flow | Session-aware, survives page reloads, modern selector engine | Yes |
| Selenium + `webdriver-manager` | Legacy EIP2 form paths (existing investment) | Migration cost vs benefit: keep where stable | Yes (pattern only) |
| Chrome DevTools Protocol over WebSocket | YouTube caption overlay (IELTS training side) — demonstrates low-level browser integration | Bypasses extension model when needed | Reference only |

---

## Automation / OS / IPC

| Technology | Role | Reason | Public? |
|---|---|---|---|
| `pywin32` / Outlook COM | Mailbox walk, folder enumeration, draft generation, attachment save | The only reliable path to live Outlook state | Yes |
| Windows registry + CLSID | `OneDrive_off.bat`, `win10右鍵舊版list/*.reg` | Native policy enforcement | Yes |
| Windows Credential Manager | `Teams_Full_Reset.bat` cleanup | Required for Teams / AAD remediation | Yes |
| Scheduled Task API (via PowerShell) | `launch_room_booking_at_open.ps1`, Ivanti report scheduling | Native scheduler integration | Yes |
| WSL bridge | `Secure_Key_Gen.bat` isolated builder | Reproducible constrained environment | Private only |

---

## AI / LLM workflows

| Technology | Role | Reason | Public? |
|---|---|---|---|
| OpenAI `gpt-4o-mini` | Email few-shot classification, 104 cover-letter assist | Cheap, fast, good-enough for extraction | Pattern public; keys private |
| Anthropic Claude | InQuire 3.0 helpdesk chat (RAG over internal KB) | Safer refusal behavior, better long-context | Private only |
| `chromadb` | Vector store for internal helpdesk KB | File-based, zero-ops, good for single-machine RAG | Private only |
| `sentence-transformers` | Embedding generation for helpdesk RAG | Offline capable | Private only |
| Few-shot / golden-sample matching | Firewall-immunity + `DELETE / REVIEW / KEEP` classification | Auditable, human-reviewable, cheap | Yes (pattern described) |

---

## Data / storage

| Technology | Role | Reason | Public? |
|---|---|---|---|
| SQLite (`applied_history.db`, `ielts_vocab.db`) | Idempotency key store, audit trail | File-based, zero-ops, transactional | Yes (schema only) |
| `openpyxl` | Excel read/write, formula translation, sheet structure | Primary format across every target system | Yes |
| `pandas` | Bounce-email dataframe manipulation, merge policy | Convenient for normalize / dedupe | Yes |
| JSON / JSONL | Telemetry, config, sample fixtures | Easy to diff, easy to replay | Yes |
| CSV | Cerberus FTP export, EIP2 monthly reports | Matches source-system exports | Yes |

---

## OCR / vision

| Technology | Role | Reason | Public? |
|---|---|---|---|
| Tesseract (`pytesseract`) | Training-certificate text extraction | Free, offline, predictable | Yes |
| OpenCV (`opencv-python`) | Image preprocessing before OCR | Thresholding, cropping for OCR robustness | Yes |
| Pillow (PIL) | Image format handling | Standard Python imaging | Yes |

---

## HTTP / networking

| Technology | Role | Reason | Public? |
|---|---|---|---|
| `requests` + custom `RetrySession` | EIP2 scrape, Cerberus export, Ivanti API | Exponential back-off for Cloudflare 403, retry-safe | Yes |
| `httpx` | Async calls where needed | Drop-in for async paths | Yes |
| WebSocket (`websocket-client`) | Chrome DevTools integration for YouTube caption tool | Low-level browser eventing | Reference |

---

## Backend / web (used where full-stack matters)

| Technology | Role | Reason | Public? |
|---|---|---|---|
| FastAPI | 104-side API surface (Phase 0/1 skeleton) | Async-native, OpenAPI for free | Yes (on 104 side) |
| Uvicorn | ASGI runtime | Standard pairing with FastAPI | Yes |
| Pydantic | Request / response models | Validation at the boundary | Yes |
| Next.js 16 App Router | 104-side operator UI (Home, Pending, Apply log, Funnel, CV analysis) | Modern React with server components | Yes (on 104 side) |
| shadcn / Tailwind CSS | Component system on the 104-side UI | Accessible, themeable, minimal config | Yes |

---

## Testing / quality

| Technology | Role | Reason | Public? |
|---|---|---|---|
| `pytest` | Regression suite (587/587 on 104 side; 90+ on InQuire 3.0) | Mature, fast, great fixtures | Yes |
| `unittest` | Public demo tests (`tests/` in `enterprise-provisioning-workbench-demo`) | Stdlib only — no install required to review | Yes |
| Semgrep (`AIcyberchallenge/inquire2_rules.yaml`) | Static security-rule experiments | Policy-as-code pattern | Yes (reference) |

---

## Deployment / packaging

| Technology | Role | Reason | Public? |
|---|---|---|---|
| Python virtualenv (`.venv`) | Per-project isolation | Default Windows-friendly isolation | Yes |
| `pyinstaller` | Packaging of desktop shells for end-user delivery | Single-folder bundles for operator machines | Yes (reference) |
| Cloud Run / Dockerfile (InQuire 3.0) | Optional cloud-hosted helpdesk chat endpoint | Exploration of hybrid desktop+cloud | Private only |

---

## What is *not* in this stack (on purpose)

- Kubernetes, microservices, Kafka — unnecessary for the target ops size; adding them would be cargo-cult.
- Mobile frameworks — the operator is at a Windows laptop.
- Heavy data-science stack beyond embeddings — the work is transactional, not analytical.

---

## Inventory source

This list is derived from:

- Three parallel deep-scan reports across the IT Automation Suite sub-projects.
- `104/data/tech_stack_inventory.json` (harvested 2026-04-12, 16 projects scanned).
- Direct inspection of `InQuire 3.0/`, `EIP2_report/`, `AI-Based Email Behavior Classification Engine/`, `outlook_file_download/`, `enterprise-provisioning-workbench-demo/`.
- Memory records: `project_sslvpn_pipeline.md`, `feedback_ssr_broken.md` for context on real operational incidents.
