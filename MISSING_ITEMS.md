# Missing Items Report

Tracking what still needs a human touch before this portfolio package can go fully live on GitHub. Items are listed in capture order (most load-bearing first).

**Updates 2026-04-15 (session 2):**
- ✅ GitHub handle confirmed: `islanderwalk`. All `<your-handle>` placeholders replaced.
- ✅ Sibling runnable demo already exists at <https://github.com/islanderwalk/enterprise-provisioning-workbench-demo>.
- ✅ Screenshot `00_github_profile_repos.png` received; needs a crop to remove the `Insyde` tab before it ships (see item below).
- ⏳ This evidence repo (`enterprise-helpdesk-automation-workbench-case`) still needs to be **created** on GitHub.
- ⏳ 本萬 JD + 3-message thread already harvested via Playwright into `docs/104_benwan_*.md` (gitignored).
- ⏳ `FOR_BENWAN.md` has Q6 added + 24-hour work receipt section.
- ⏳ `docs/reply_to_benwan.md` rewritten with psychology-calibrated tone (no emoji, no apologies, exit option).
- ⏳ Playwright scripts retrofitted to minimized-taskbar default (`PLAYWRIGHT_DEFAULTS.md`).

---

## ~~Missing item~~ RESOLVED 2026-04-15
本萬有限公司 JD text + 104 recruiter-chat transcript

## Resolution
Harvested directly via the 104 project's authenticated Playwright persistent context.
- JD (job 8qf2c): `docs/104_benwan_jd.md` / `.json`
- 3-message conversation thread: `docs/104_benwan_thread.md` / `.json`
- Frame analysis: `docs/_frame_analysis_benwan.md`
- Response draft: `docs/reply_to_benwan.md`
- Public-facing 5+1 answer: `FOR_BENWAN.md`

All four of those files are in `.gitignore` except `FOR_BENWAN.md`.

`JD_ALIGNMENT.md` has been fully rewritten against the real JD + recruiter context.

---

## Missing item
Public-safe execution screenshot of the companion demo (`03_entrypoint.png`, `04_execution_console.png`, `05_output_example.png`)

## Why missing
No terminal capture exists yet from the `enterprise-provisioning-workbench-demo/` runnable path. An SVG run-summary already exists at `../enterprise-provisioning-workbench-demo/docs/screenshots/run-summary.svg` but a PNG terminal capture is stronger evidence.

## Recommended fix
```bash
cd "../enterprise-provisioning-workbench-demo"
python -m demo_app --show-stages
# Capture terminal → save as portfolio_package/assets/screenshots/03_entrypoint.png

python -m demo_app sample --output sample-output/requests.json
python -m demo_app --input sample-output/requests.json --save-report sample-output/run-report.json
# Capture terminal → 04_execution_console.png
# Open run-report.json in VS Code → 05_output_example.png
```

---

## Missing item
Repository overview + folder structure screenshots (`01_repo_overview.png`, `02_folder_structure.png`)

## Why missing
Not captured. Trivial to produce.

## Recommended fix
- `01_repo_overview.png`: VS Code sidebar showing `104/portfolio_package/` tree expanded. Crop to relevant area.
- `02_folder_structure.png`: Windows Explorer showing the IT Automation Suite parent folder with the ~20 project folders visible. Crop to relevant area.

---

## Missing item
Rendered process-diagram PNG (`06_process_diagram.png`)

## Why missing
The Mermaid source is already in `docs/workflow-diagram.md`. A static PNG export has not been generated.

## Recommended fix
Either:
- Open `docs/workflow-diagram.md` in a Mermaid-capable viewer (GitHub renders it natively; VS Code with the Mermaid extension; Mermaid Live at `mermaid.live`), export to PNG, save as `assets/screenshots/06_process_diagram.png`.
- Or install `@mermaid-js/mermaid-cli` and run `mmdc -i docs/workflow-diagram.md -o assets/screenshots/06_process_diagram.png`.

---

## Missing item
GitHub README preview screenshot (`07_readme_preview.png`)

## Why missing
The package has not been pushed to a public GitHub repo yet.

## Recommended fix
1. Create a public repo named `enterprise-helpdesk-automation-workbench-case` (see Section 9 of `AGENT_REPORT.md` for naming alternatives).
2. Push the contents of `portfolio_package/` as the repo root.
3. Visit the GitHub page in a browser, capture the rendered README as `07_readme_preview.png`.

---

## Missing item
Synthetic training-certificate sample image (for the OCR demonstration screenshot `11_ocr_extraction.png`)

## Why missing
Real training certificates cannot be published (contain employee names + internal system references). A synthetic cert has not yet been built.

## Recommended fix
Create a PNG (e.g. via PowerPoint → export, or a quick HTML → Puppeteer screenshot) with:
- fake course name: `Information Security Awareness Training 2026 Q1`
- fake employee name: `Alice Chen`
- fake completion date
- layout similar to the real cert template (but not pixel-identical)

Then capture the `outlook_file_download` UI processing this synthetic cert → save as `11_ocr_extraction.png`.

---

## Missing item
Mock SSLVPN bounce-email review workbook (for `09_email_classifier_report.png`)

## Why missing
The real workbook contains real bounce emails with real user addresses. Cannot publish.

## Recommended fix
Build a small Excel workbook (`examples/sanitized_output/bounce_review_sample.xlsx`) with:
- synthetic email addresses (`user1@example.com`, `user2@example.com`, ...)
- column structure matching the real `SSLVPN_Account_Review_Final.xlsx` (Account, Last Seen, Bucket, Reason, AI Confidence)
- 10–20 rows covering the `DELETE / REVIEW / KEEP` buckets

Then open in Excel, capture as `09_email_classifier_report.png`.

---

## Missing item
Signed-off sanitization pass on the private `InQuire 3.0/` repo before any public code excerpt is lifted

## Why missing
The `InQuire 3.0/` repo contains internal hostnames, tenant IDs, and references to customer workflows. The current portfolio package references the architecture but does not lift any code.

## Recommended fix
- If any code excerpt gets lifted from `InQuire 3.0/` into a future version of this public package, run the sanitization gate (`SANITIZATION_NOTES.md` §"How the sanitization gate is applied") on that excerpt specifically.
- For now, the public demo at `../enterprise-provisioning-workbench-demo/demo_app/` provides the runnable code surface without needing to lift anything from private.

---

## Missing item
Measurable metrics for the operational improvement section

## Why missing
Runtime telemetry exists in the private repos (dated JSONL files under `data/`) but has not been aggregated into headline metrics.

## Recommended fix
Run a small aggregation over the last 4 weeks of telemetry logs to produce:
- rows processed per run (mean / p95)
- `create / review / skip` mix over time
- readback failure rate
- per-adapter execution time

Add the aggregated numbers to `CASE_STUDY.md §10 "Result / improvement"` as concrete figures. Do NOT fabricate — if the telemetry isn't aggregated yet, keep the current honest-operational-improvement language.

---

## Missing item
License file

## Why missing
`LICENSE` not added to `portfolio_package/`.

## Recommended fix
Copy `LICENSE` (MIT) from `../enterprise-provisioning-workbench-demo/LICENSE` into `portfolio_package/`, or point to it in the future GitHub repo's top level.
