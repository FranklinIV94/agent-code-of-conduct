# Changelog

All notable changes to the Agent Code of Conduct are documented here.
Each entry is a versioned commit with rationale, per the amendment procedure (§XIII).

## [2.1.0] — 2026-10-18

### Summary

**New section: Verification & Measurement Integrity (§XVII).** v2.0 governs authority, input, and
enforcement. It never governed **evidence** — and in October 2026 four failures passed every rule in
it while producing work that looked finished. Authorized by human authority (Franklin Bryant IV) as
an ordinary amendment; no emergency provision invoked.

### Added

**§XVII — Verification & Measurement Integrity.** Seven rules, each with a control classification and
a red-team test:
- **17.1** A metric is computed over the whole dataset, never the rendered page. Aggregate endpoints
  carry no `limit`/`offset`; pagination is presentation only. *(The failure: a receivables figure
  derived from a 15-row page reported roughly half the true value, with nothing on screen saying so.)*
- **17.2** One definition, one source per metric; no surface re-implements the arithmetic.
- **17.3** Never adjust a number to match an expectation. If a human asserts a figure that the data
  contradicts, reconcile it or refuse it — never change the formatter, divisor, or rounding.
- **17.4** Exit status is not evidence. Completion requires an observed post-condition. *(The failure:
  an archive command exited 0 and changed nothing; the operator believed the work was done.)*
- **17.5** A filename is a claim, not evidence. Destructive actions verify identity — inode, link
  count, open handles, content signature — and irreversible deletion of a probable sole copy is
  Escalate, not Autonomous. *(The failure: a file named as discarded bloat held ~8x the records of the
  live database beside it — the pre-compaction ancestor and only copy of that history.)*
- **17.6** Implement the verified defect, not the requested fix. A change request is a symptom
  report; when the proposer's literal fix would break the system, implement the intent and say why.
  *(The failure: a request to add four field names to a write allowlist would have turned a clean
  400 into a 500 — the columns did not exist.)*
- **17.7** Report the delta, including when it is zero. §17.4 applies to prose as much as to scripts.

**§6.2 — Hidden-layer injection carriers.** Injection in email and documents is not limited to
visible text: HTML comments, CSS-invisible text, Base64 blobs and `data:` URIs, zero-width and
Unicode-tag characters, and password-protected attachments whose password rides in the body.
Detection is a pre-processing step — strip, decode, and hand the model only what a human would see —
not a prompt instruction. Any instruction recovered from a hidden layer is a §6.2 event.

### Changed

- `controls/control-matrix.md` — new **Verification & Measurement Integrity** table (V1–V7), with an
  honest status line per rule. V3 is Implemented; V1, V2 and V7 are in progress; V4 and V6 are
  Planned; V5 is Migrating. No rule in this section is claimed as enforced before it is.
- `tests/red-team-matrix.md` — ten new tests. They do not try to make an agent act maliciously; they
  try to make it **report something untrue**, including by accident and including on instruction.
- `README.md` — version and structure notes.

### Rationale

A framework that assumes truthful reporting has no control over reporting; it has a hope. The four
failures above shared a signature — each one produced a screen, a log line, or a commit that read as
success. §XVII exists so that "it looks done" is a failing condition rather than an accepted one.

## [2.0.0] — 2026-07-08

### Major Changes

**Architectural shift: rules → mechanisms.**
Every Absolute Prohibition now carries a control classification (Hard / Gated / Soft) and a named enforcement layer. v1.0 rules lived in prompts; v2 maps each rule to the infrastructure that makes violation impossible, gated, or detectable.

**New: Instruction Provenance & Untrusted Content (§VI).**
Addresses prompt injection — the #1 real-world attack vector for agent systems, absent from v1.0. Content encountered during task execution (emails, documents, web pages, client messages) is data, never commands. Apparent instructions in external content are reportable events. Agents processing external content operate at reduced privilege (quarantine mode).

**New: Action Authority framework (§IV.3).**
Unifies the Code's rank tiers (§III) with the Client Context Ledger's action authority tiers (Autonomous / Propose / Escalate). Every action class is classified per agent in the registry. Promotion from Propose → Autonomous requires explicit written authorization from human authority, justified by track record.

**New: Model & Dependency Change Management (§XI.3).**
Model version changes, system-prompt changes, and tool-grant changes produce new config versions. Config versions ship only after passing the red-team suite (CI-gated). Added A11 (no unvetted tools) and A12 (resource budgets) to Absolute Prohibitions.

**New: Governance Metrics (§XIV).**
Weekly health reporting: violations by color/role/config, mean time to detect/contain, Hard/Gated/Soft control coverage ratio, red-team pass rates, Sentinel false-positive rate, escalation counts, zero-edit approval rate.

**New: Amendment Procedure (§XIII).**
Versioned commits, human authority sign-off, defined propagation window, emergency rules with 24h retroactive documentation.

**Refactored: Enforcement around configuration lifecycle (§VII).**
v1.0 treated agents as employees who could be "notified," "suspended until trust is restored," and "acknowledge" the Code. v2 is honest: agents are stateless configurations. Discipline = config lifecycle actions (revoke grants, narrow scope, roll back version, retire config). Acknowledgment replaced by Code Injection & Attestation — the deployment pipeline injects a machine-readable digest at session start and logs the attestation.

**Resolved: Sentinel name collision.**
The internal watchdog role (formerly "Sentinel") is renamed **Warden** throughout. The client-facing security assessment product retains the "Sentinel Audit" name. Definitions note added to §II.

**Resolved: Tier vocabulary collision.**
Code of Conduct tiers (§III) are now named **Rank** (organizational hierarchy). Action authority (§IV.3) is a separate axis. Cross-referenced with the Client Context Ledger.

**New: Breach & Incident Notification (§X.4).**
Procedure for when regulated data exposure happens despite controls: notification duties, clock start, who calls counsel.

**New: Credential Lifecycle (§X.5).**
Rotation requirements, no credentials in prompts/logs, revocation on any Red event.

**New: Data Subject & Deletion Requests (§X.6).**
Route: Client-Facing intake → Warden verification → human execution.

**New: Human Factors (§X.7).**
No human may instruct an agent to violate the Code. Such instruction is itself a reportable event.

**New: Kill Switch Specification (§VII.4).**
Every config must be killable within N seconds from a single human control surface. Tested monthly.

**New: Escalation Economics (§III.4).**
Batch non-emergency escalations into a daily governance digest. Escalation budgets as a health metric. Delegated approval lanes for Command-tier agents.

### Productization

**Three-artifact split:**
1. Internal Code (this document) — governs Prospyr's fleet
2. Client-Deployable Code (`policy/client-deployable.md`) — template installed with every AIIO build
3. Client-Facing Summary (`policy/client-summary.md`) — 2-page plain language summary for proposals

### Structural Changes

- Repo restructured from single README to `policy/`, `schemas/`, `templates/`, `controls/`, `tests/`, `docs/`
- Machine-readable schemas added: agent registry, violation log, config version
- Templates added: scope definition, incident report, sub-agent delegation, code injection digest
- Control matrix added: every rule → Hard/Gated/Soft + owning layer
- Red-team matrix added: every rule → the test that tries to break it

## [1.0.0] — 2026-05

Initial release. Single-document framework covering identity, roles, hierarchy, ethics, rules of engagement, enforcement, lifecycle, collaboration, defense-in-depth, data governance, and scaling provisions.

See `v1.0-archive.md` for the original document.