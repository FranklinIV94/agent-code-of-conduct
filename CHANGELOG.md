# Changelog

All notable changes to the Agent Code of Conduct are documented here.
Each entry is a versioned commit with rationale, per the amendment procedure (§XIII).

## [2.1.0] — 2026-08-03

### New rule classes

**New: Efficiency & Resource Discipline (§5.4, E1–E10).**
v2 governed what agents may do, not how much they may burn doing it. E-rules make waste a violation: retry budgets with hard stop (E1), no retrying deterministic failures (E2), no polling loops (E3), no re-fetching held data (E4), batch over spray (E5), a hard per-session token/API/spend ceiling that halts the task — operationalizing A12 (E6), circuit breakers on down dependencies (E7), one owner per task / no duplicate work — operationalizing B7 (E8), verify-once-then-act instead of speculative probing (E9), and escalate-on-stall instead of churn (E10). Reference incident: the Aug 3 portal-key `401` retried dozens of times across agents, with parallel re-diagnosis — hours of usage to rediscover one misconfiguration.

**New: Build & Deploy Governance (§XVII, D1–D11).**
The fleet ships code and schema to production; v2 had no rules for how. D-rules require recognized commit authors (D1), deploy-from-Git-integration-only (D2), no uncommitted deploys (D3), preview-before-promote (D4), branch→PR→review→merge with branch protection (D5), migrations-via-CI with no hand-syncing prod (D6), no secrets in git via a CI secret-guard (D7), rotate-at-source-then-scrub on exposure (D8), keys with one source of truth and never in channels (D9), no parallel deploys during an incident (D10), and "verified means receipts" (D11). Every rule maps to a specific failure in the Aug 3 portal outage, which was a locked deploy pipeline plus five stacked process failures — not lost code. Operations-facing summary: the Build & Deploy Governance Standard v1.0 artifact.

### Controls & tests

- `controls/control-matrix.md`: added Efficiency (E1–E10) and Build & Deploy (D1–D11) tables with control class, layer, status, and migration path. Priority Gated builds: E1/E6/E7 (stop the burn) and D1/D2/D5 (unlock + lock down the pipeline). D7 secret-guard already live on `peo-automation-1`.
- `tests/red-team-matrix.md`: added a breaking test for each new E- and D-rule.

### Notes

- Amendment made by PR for Rank 0 (human authority) sign-off, per §13.1. Version bump is Minor (new rules/controls) trending Major given two new sections — recorded as 2.1.0; may be reissued as 3.0 at Rank 0's discretion.

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