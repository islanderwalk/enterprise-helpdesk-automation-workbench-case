# Screenshot Index

Every screenshot that supports this portfolio — what it shows, which claim it backs, and whether it has been redacted. Files live in `assets/screenshots/`.

---

## Required screenshot set

| File | What it shows | What claim it supports | Redaction status | Source |
|------|---------------|------------------------|------------------|--------|
| `00_github_profile_repos.png` | `github.com` profile `islanderwalk` top repos: `enterprise-provisioning-workbench-demo`, `InQuire-3.0`, `Outlook-Email-Behavior-Classification-Engine`, `BMC_BIOS_1213`, `EnglishAI-Toolkit`, `ai-short-video-mvp`, `islanderwalk/islanderwalk` | Multiple related public + private repos are real and cross-reference; the person in this case study actually maintains these | ⚠️ **Crop the other browser tab before publishing** — the adjacent tab title in the raw capture reads `Insyde - 6.8helpdesk_troubleshoo...` which identifies the employer. Crop to just the GitHub viewport. | Chrome on github.com |
| `01_repo_overview.png` | Root file tree of this portfolio repo inside the `104/` folder | This is an organized evidence package, not a file dump | NOT yet captured (action item — see below) | VS Code / Explorer |
| `02_folder_structure.png` | IT Automation Suite parent folder showing the ~20 project folders that feed this portfolio | Breadth of real work across helpdesk / email / provisioning / endpoint domains | NOT yet captured | Windows Explorer |
| `03_entrypoint.png` | `python -m demo_app --show-stages` running in terminal, plus the ACTION BOARD output | Public demo is runnable end-to-end with sample data | Available as SVG at `../enterprise-provisioning-workbench-demo/docs/screenshots/run-summary.svg`; PNG version needs capture | Terminal run |
| `04_execution_console.png` | Full terminal run of the sanitized demo with a JSON-file input path and `--save-report` output | `create / skip / review` decisions are backed by a real report, not only architecture notes | NOT yet captured | Terminal run |
| `05_output_example.png` | The generated `sample-output/run-report.json` opened in an editor, showing reasons per decision | Deterministic decisions carry readable `reason` strings; audit-friendly | NOT yet captured | VS Code viewing `run-report.json` |
| `06_process_diagram.png` | Rendered Mermaid diagram of the "before / after" workflow (from `docs/workflow-diagram.md`) | Clear, non-technical visual of the process improvement | Can be rendered from the Mermaid block; needs static PNG export | Mermaid Live → PNG |
| `07_readme_preview.png` | GitHub rendering of this repo's `README.md` | Public presentation is clean and guides a reviewer | NOT yet captured (requires pushing to GitHub first) | GitHub view |

---

## Optional supporting screenshots (stronger case)

| File | What it shows | Supports | Redaction | Notes |
|------|---------------|----------|-----------|-------|
| `08_inquire3_shell.png` | InQuire 3.0 PySide6 tabbed desktop shell (from private repo) | The flagship is real, working desktop software | **HEAVY redaction required** — blur tab titles that reveal internal system names, blank out real user rows | Private only; reconstruct by running the sanitized demo desktop skeleton if one exists, or substitute with the Mermaid shell diagram |
| `09_email_classifier_report.png` | `SSLVPN_Account_Review_Final.xlsx` opened in Excel with risk bucket columns | Firewall-immunity + AI classification produces an auditable bucketed report | Blank out all real email addresses and account names; show only the column headers + 2-3 mock rows | Generate mock workbook with dummy data |
| `10_eip2_report_terminal.png` | Terminal showing the 5-stage pipeline run (`1.fetch_cerberus_list.py` → `5.sign_off_forms.py`) | The provisioning pipeline is a single end-to-end run | Redact hostnames in the URLs; use `EIP2_URL=<redacted>` in shown output | Run against a dev/mock environment |
| `11_ocr_extraction.png` | `outlook_file_download` UI showing a certificate image on the left and the fuzzy-matched roster row on the right | OCR + fuzzy match actually works on a real certificate layout | Replace real certificate with a synthetic sample (generate a PNG with a fake course name + fake employee name) | Build a synthetic sample cert |
| `12_bat_toolkit_output.png` | `Teams_Full_Reset.bat` running in an elevated shell, showing the step counter and cleanup results | Helpdesk first-aid toolkit is real and readable | None required — the script output is generic | Capture from a test machine |
| `13_tests_passing.png` | `pytest tests --tb=short -q` showing 587 passed on the 104 side and/or `unittest discover` passing on the public demo | Regression safety net is real | None required | Terminal capture |

---

## Naming convention

`NN_short_name.png` where `NN` is a two-digit sequence. Required screenshots use 01–07; optional supporting screenshots use 08 onward. Kebab case OK; camelCase not preferred.

---

## Capture priority

When time is limited, capture in this order:

1. `00_github_profile_repos.png` — already captured, needs crop (see redaction above). Ship this first because it's the strongest social-proof asset: multiple related repos under the same profile.
2. `01_repo_overview.png` + `02_folder_structure.png` — prove the package exists and has real scope.
3. `03_entrypoint.png` + `04_execution_console.png` + `05_output_example.png` — prove the public demo runs.
4. `06_process_diagram.png` — give the reviewer a visual anchor.
5. `13_tests_passing.png` — prove the regression net is real.
6. Everything else is supporting depth.

---

## What each screenshot must not contain

Cross-check every capture against `SANITIZATION_NOTES.md`. Specifically:

- No internal hostnames (`eip2.*`, `vpn.*`, `sys.*`, `ftp.*` and any company-specific subdomain).
- No real email addresses, account IDs, or employee numbers.
- No API keys, tokens, cookies, or session identifiers.
- No customer / client names.
- No raw HTML captures that could be used to reverse-engineer an internal admin UI.
- No visible tenant IDs or Azure / AAD GUIDs.

If a screenshot cannot be taken without revealing the above, either (a) redact with a solid black rectangle, (b) substitute with a sanitized mock, or (c) omit the screenshot and describe it in text instead.

---

## Action items before publishing

- [x] **`00_github_profile_repos.png`** — captured (received 2026-04-15). ⚠️ **Crop** before publishing to remove the `Insyde - 6.8helpdesk_troubleshoo...` tab title. Save the cropped version to `assets/screenshots/00_github_profile_repos.png`.
- [ ] Capture `01_repo_overview.png` (VS Code sidebar showing this `portfolio_package/` tree).
- [ ] Capture `02_folder_structure.png` (Windows Explorer showing the IT Automation Suite parent folder).
- [ ] Run `python -m demo_app --show-stages` in `../enterprise-provisioning-workbench-demo/` and capture `03_entrypoint.png`.
- [ ] Run the demo with `--input` and `--save-report`, capture `04_execution_console.png` and `05_output_example.png`.
- [ ] Export the Mermaid block in `docs/workflow-diagram.md` to PNG → `06_process_diagram.png` (use Mermaid Live or `mmdc` CLI).
- [ ] Push to GitHub, capture `07_readme_preview.png`.
- [ ] Optional: build the synthetic cert + mock workbook to enable 09 / 11 without leaking internal data.
