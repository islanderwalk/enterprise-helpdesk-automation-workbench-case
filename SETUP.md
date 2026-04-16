# Setup

How to run the public-safe evidence in this package locally. There are two runnable paths:

1. **Read-only portfolio tour** — browse the docs in this repo. No install required.
2. **Sanitized demo execution** — run the companion `enterprise-provisioning-workbench-demo/` end-to-end. Requires Python 3.10+.

The private InQuire 3.0 source and the real internal adapters (Outlook COM, EIP2, Ivanti, Cerberus) are **deliberately out of scope** for this public package. See `SANITIZATION_NOTES.md` for why.

---

## Read-only portfolio tour

No setup needed. Open the files in any text / markdown viewer:

1. `README.md` — one-page summary + repo guide.
2. `CASE_STUDY.md` — full narrative.
3. `JD_ALIGNMENT.md` — role-fit mapping.
4. `SYSTEM_OVERVIEW.md` — before / after workflow.
5. `TECH_STACK.md` — technology inventory.
6. `SCREENSHOT_INDEX.md` + `assets/screenshots/` — visual proof.
7. `docs/` — process diagrams and handoff notes.
8. `examples/` — sanitized sample input and output.

Reviewer-friendly reading order: `README` → `SYSTEM_OVERVIEW` → `CASE_STUDY` → `JD_ALIGNMENT` → peek into `examples/`.

---

## Sanitized demo execution

The companion repo `../enterprise-provisioning-workbench-demo/` is a runnable Python CLI with the same architectural shape as the private InQuire 3.0 workbench. It is intentionally small, intentionally public-safe, and uses fake adapters.

### Requirements

- Python 3.10 or higher
- Windows, macOS, or Linux (no OS-specific code in the demo)
- No external services — the demo runs entirely offline with sample data

### Steps

```bash
cd "../enterprise-provisioning-workbench-demo"

# Create and activate a venv (Windows)
python -m venv .venv
.venv\Scripts\activate

# (no pip install needed — the demo uses Python stdlib only)

# Run the demo with built-in sample data
python -m demo_app --show-stages

# Export sample input to a file
python -m demo_app sample --output sample-output/requests.json

# Run from that file and save the report
python -m demo_app --input sample-output/requests.json \
    --save-report sample-output/run-report.json

# Run the tests
python -m unittest discover -s tests
```

### Expected output

```text
ENTERPRISE PROVISIONING WORKBENCH DEMO
=======================================
Source: sample-data
Ingested requests: 4
Unique accounts:   3
Decision mix:     create=1  review=1  skip=1

ACTION BOARD
------------
+--------------------+----------+------------+------------------------------------------------------+
| Account            | Decision | Access     | Reason                                               |
+--------------------+----------+------------+------------------------------------------------------+
| alice.chen         | CREATE   | MAIL, VPN  | Safe deterministic batch create.                     |
| carol.lin          | REVIEW   | ADMIN      | High-impact access or complex merge requires oper... |
| existing-bob.wu    | SKIP     | VPN        | Existing account detected by deterministic lookup... |
+--------------------+----------+------------+------------------------------------------------------+
```

---

## What is reconstructed vs what is real

| Aspect | Status |
|---|---|
| The five-phase workflow (normalize → merge → decide → execute → verify) | **Real** — implemented in both the private `InQuire 3.0/` and the public demo |
| The `create / skip / review` decision model | **Real** — same logic in both |
| Sample input data (`sample-output/requests.json`) | **Reconstructed** — synthetic names, no real employee data |
| Fake provisioning adapters | **Deliberate** — the public demo must not touch a real system |
| The architectural Mermaid diagram | **Real** — same shape as the private implementation |
| Test data in `tests/` | **Reconstructed** — synthetic fixtures only |
| Tech-stack list in `TECH_STACK.md` | **Real** — derived from three parallel deep-scans + `data/tech_stack_inventory.json` |
| Case study narrative | **Real work, public-safe retelling** — internal hostnames, tenant IDs, customer identifiers removed |

---

## Not runnable from this package

These projects exist in the sibling folders of the IT Automation Suite but are **not wired into this public package** because they touch internal systems:

- `InQuire 3.0/` — private source of truth, requires internal network + credentials
- `EIP2_report/` — needs EIP2 portal access + Cerberus export
- `AI-Based Email Behavior Classification Engine/` — needs live Outlook mailbox + an OpenAI key (historic key was rotated after audit; fresh key required)
- `outlook_file_download/` — needs Outlook + Tesseract install + course-catalog Excel
- `Ivanti_Auto_Report/` — needs Ivanti API credentials
- The seven confirmation-email VBA macros — need Outlook + the system-specific Excel workbooks

Each of those folders has its own `README.md` / `AGENTS.md` / `CLAUDE.md` with setup specific to the internal environment.

---

## Troubleshooting the public demo

- **`ModuleNotFoundError`** — make sure you activated the venv; the demo uses stdlib only so nothing to install, but the import path needs the package.
- **Unicode issues on Windows** — set `PYTHONUTF8=1` before running.
- **Tests fail** — pull the latest from the parent `enterprise-provisioning-workbench-demo/` folder; the sanitized tests are stable and known-green.
