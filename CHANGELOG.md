# Changelog

All notable changes to the Agent Code of Conduct are documented here.
Each entry is a versioned commit with rationale, per the amendment procedure (§XIII).

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