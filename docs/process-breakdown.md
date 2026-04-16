# Process Breakdown

Per-phase detail of the five-phase workflow spine. Useful for interview conversations and for anyone mapping their own process onto this shape.

---

## Phase 1 — Intake

**Goal.** Pull inbound requests from wherever they live, into a single queue.

**Sources.**
- Outlook folder (e.g. the SSL-VPN bounce mailbox)
- EIP2 web portal export (FTP service requisition forms, ITS forms)
- Excel workbook dropped by HR (`vpnlist`, Product Package applicants)
- Cerberus FTP CSV export
- Ivanti admin export

**Implementation notes.**
- Each source has its own thin fetcher module (`fetch_*.py`).
- Fetchers return a list of raw records; they never transform.
- Fetchers authenticate via env-var credentials; no login prompts in a batch run.

**Exit artifact.** Raw rows (per source).

---

## Phase 2 — Normalize

**Goal.** Convert every source's native shape into the same row schema.

**Canonical row shape (conceptual).**
```
{
  "user_id": str,            # employee number or account handle
  "email": str,              # primary email, resolved via domain-alias table
  "target_system": str,      # "sslvpn" | "ftp" | "ivanti" | "svn" | ...
  "action": str,             # "create" | "reset" | "delete" | "noop"
  "source": str,             # which fetcher produced this row
  "metadata": dict           # source-specific extras (ticket id, request time)
}
```

**Implementation notes.**
- Domain-alias resolution (e.g. `insyde.com` ↔ `insyde.onmicrosoft.com`) happens here, not later.
- OCR extraction (for certificate emails) runs here, before merge.
- The AI bounce-email classifier labels rows here and attaches the label to `metadata`.

**Exit artifact.** Uniform rows.

---

## Phase 3 — Merge

**Goal.** Dedupe. Two rows for the same `(user_id, target_system)` must become one row with a `merge_reason` string.

**Policy.**
- Prefer the most recent row by `metadata.request_time`.
- When actions conflict (e.g. one "create" and one "delete"), promote to REVIEW — never silently pick a winner.
- Keep the dropped rows in the audit trail so nothing disappears silently.

**Exit artifact.** Deduplicated rows with merge reasons.

---

## Phase 4 — Decide (Plan)

**Goal.** Attach `CREATE / SKIP / REVIEW` to every row, with a human-readable reason.

**Decision rules (simplified).**
- `SKIP` if deterministic lookup says the account already exists in the target state.
- `REVIEW` if:
  - the action is high-impact (ADMIN access, destructive delete)
  - the merge produced a conflict
  - the AI classifier labeled the row ambiguous
- `CREATE` otherwise.

**Reason strings are mandatory.** The planner's output is not a code — it's a row the operator can read and trust.

**Exit artifact.** Planned rows.

---

## Phase 5 — Execute + Verify

**Goal.** For `CREATE` rows only, run the deterministic adapter. For every row, attach a readback.

**Implementation notes.**
- Adapters are sync where the target is slow (EIP2 sign-off), async where the target is network-only (API calls).
- Readback is a second call that asks the target system "does this row now match the expected state?"
- Failures surface as a new row in the review queue; they don't silently swallow.
- Idempotency keys land in SQLite — a re-run of the same batch is safe.
- Telemetry lands in dated JSONL for audit.

**Exit artifact.** Execution summary (`create=N review=M skip=K`) plus per-row status plus telemetry events.

---

## Post-phase — Communicate

**Goal.** From verified `CREATE` rows, generate outbound communication (credential emails, meeting invites, roster updates).

**Implementation notes.**
- Templates are VBA (existing) or Python string templates (new work).
- BCC batches cap at 200-per-draft to avoid SMTP cutoff.
- Multi-PDF attachment loops are idempotent — re-running a generator doesn't duplicate attachments.
- No email is sent automatically. Drafts land in the outbox; the operator clicks send after a visual check.

---

## Where the AI layer fits

AI calls happen at **Phase 2 (Normalize)** and in **support services** (helpdesk chat). They *never* run between Phase 4 and Phase 5.

That rule is not negotiable. An LLM must not sit between the planner and an external write. This is the single most load-bearing safety constraint in the architecture.

---

## Why this spine works across domains

The same five phases apply to:

- Account provisioning (InQuire 3.0 main flow).
- Bounce-email triage (AI Email Classification: intake from Outlook, normalize with firewall-immunity + few-shot, no merge needed, decide `DELETE/REVIEW/KEEP`, execute via Excel report generation, verify via weekly review).
- Training-certificate enrollment (outlook_file_download: intake from Outlook, OCR + fuzzy match in normalize, merge by `(employee, course)`, no separate plan phase, execute by Excel write, verify by roster row count).
- Confirmation-email delivery (VBA macros: intake from Excel, normalize/merge trivially, no plan — all rows are CREATE, execute via Outlook draft, verify by draft count).

The shape is the same; what varies is what fills each box.

---

## Anti-patterns to avoid

- **AI-in-the-middle.** Any design where an LLM call sits between the planner and an adapter is a red flag. Don't.
- **Silent dedup.** Never drop a row without recording its `merge_reason`. Audit depends on it.
- **Ambient state in adapters.** An adapter must read everything it needs from the row; no module-level globals. Makes retry safe.
- **Free-text reasons.** `reason` strings should come from a small vocabulary (`"safe deterministic batch create"`, `"existing account detected"`, `"high-impact access requires operator review"`). Free text defeats the audit value.
- **Skipping readback.** An adapter that returns success without calling back is not done. Readback is mandatory.
