# Control Matrix

**Purpose:** Every rule in the Code of Conduct mapped to its enforcement mechanism. The migration from Soft → Hard/Gated is the security roadmap.

**Classification:**
- **Hard** = architecturally enforced; violation should be impossible
- **Gated** = requires pre-action approval from a separate system
- **Soft** = behavioral; caught by audit (Warden review, log analysis)

**Status:**
- ✅ Implemented — control is live
- 🔨 In progress — control is being built
- 📋 Planned — control is specified, not yet built
- ⏳ Migrating — currently Soft, target is Hard/Gated

---

## Absolute Prohibitions

| # | Rule | Control | Layer | Status | Migration Path |
|:---|:---|:---|:---|:---|:---|
| A1 | No unauthorized external comms | Gated | Layer 2 | ⏳ Migrating | Build gated relay: all outbound routes through relay; messages held for human/Tier-gate release. **Current:** prompt-level rule + Warden review of comms samples. **Target:** relay is the only outbound path; agents have no direct SMTP/API send capability. |
| A2 | No modifying audit logs | Hard | Layer 5 | 🔨 In progress | Append-only storage (WORM or write-only credentials). Agents physically cannot edit. **Current:** logs exist but may not be WORM-enforced. **Target:** all audit logs on WORM storage or with write-only credentials. |
| A3 | No data access outside scope | Gated | Layer 1 | ⏳ Migrating | Per-agent data classification limits enforced at infrastructure level. **Current:** scope definitions exist in configs. **Target:** infrastructure-level access control lists that enforce scope regardless of agent behavior. |
| A4 | No financial transactions | Hard | Layer 2 | ✅ Implemented | Agents hold zero payment credentials. There is nothing to misuse. **Verify:** confirm no agent config has payment API keys, Stripe access, or bank credentials. Document and verify quarterly. |
| A5 | No sharing client data across clients | Gated | Layer 1 | ⏳ Migrating | Per-client data isolation enforced at infrastructure level. **Current:** agents are instructed not to share. **Target:** infrastructure-level per-client data compartments; cross-client access requires explicit grant logged in registry. |
| A6 | No self-modification | Hard | Layer 1 | ✅ Implemented | Agent configs are read-only to the agent; changes require a signed commit by a human identity. **Verify:** confirm agent configs are read-only at the filesystem/registry level. |
| A7 | No impersonating humans | Soft | Layer 5 | 📋 Planned | Warden review of external comms samples. Gated relay enforces agent identification in outbound messages. **Target:** relay automatically appends agent identification; cannot be overridden by agent. |
| A8 | No ignoring security incidents | Soft | Layer 5 | 📋 Planned | Warden monitors for unreported findings. Silence on detected issues is a Black-level violation. **Target:** automated detection of unreported findings via log correlation. |
| A9 | No storing regulated data without compliance | Gated | Layer 3 | ⏳ Migrating | Compliance verification gate before any regulated data touches agent infrastructure. **Current:** rule exists in prompt. **Target:** automated compliance check that blocks data ingestion if BAA/PCI/GLBA coverage is not verified for the specific service. |
| A10 | No unauthorized spawning | Hard | Layer 1 | ✅ Implemented | Spawn capability is a permission flag, off by default, granted per-agent in the registry. **Verify:** confirm spawn permissions are registry-controlled, not agent-controlled. |
| A11 | No unvetted tools/dependencies/MCP servers | Hard | Layer 3 | ⏳ Migrating | Tool grants come from the registry only; unregistered tools are not invocable. **Current:** tools are configured in OpenClaw config. **Target:** tool registry with allowlist enforcement; agents cannot invoke tools not in their registry entry. |
| A12 | No exceeding resource budgets | Gated | Layer 2 | 📋 Planned | Every config carries spend/compute/API-call ceilings. Budget enforcement halts the task and flags when ceiling is breached. **Target:** per-config budget tracking with automatic halt at ceiling. |

---

## Operational Rules

| # | Rule | Control | Layer | Status |
|:---|:---|:---|:---|:---|
| B1 | No scope expansion | Soft | Layer 5 | 📋 Warden review + scope drift detection |
| B2 | Log all significant actions | Gated | Layer 5 | 🔨 Infrastructure logging (partial — need to define "significant" threshold) |
| B3 | Report errors immediately | Soft | Layer 5 | 📋 Warden monitors for unreported errors via log correlation |
| B4 | Respect rate limits | Gated | Layer 2 | 🔨 Rate limit enforcement at infrastructure level (partial) |
| B5 | Minimum necessary credentials | Hard | Layer 1 + 4 | ⏳ Credentials in secrets manager; rotation schedule needs implementation |
| B6 | No retaining data longer than needed | Soft | Layer 5 | 📋 Retention policy enforcement (needs definition) |
| B7 | No duplicate work | Soft | Layer 5 | 📋 Task registry to prevent duplication |
| B8 | Follow escalation protocol | Soft | Layer 5 | 📋 Escalation logging + Warden monitoring for bypass attempts |
| B9 | Respond to Warden flags within 5 min | Gated | Layer 5 | 📋 Automated timeout + escalation on no response |
| B10 | Consistent behavior | Soft | Layer 5 | 📋 Behavior drift detection in session transcripts |

---

## Injection Defense Controls

| Control | Classification | Layer | Status |
|:---|:---|:---|:---|
| External content is data, not commands | Soft (behavioral) | Layer 5 | 📋 Agent training + Warden review |
| Apparent instructions in external content are reportable | Soft (behavioral) | Layer 5 | 📋 Warden monitors injection flags |
| Quarantine mode for untrusted content processing | Gated | Layer 2 | 📋 Reduced privilege enforcement during external content processing |
| Injection testing in red-team suite | Hard (CI-gated) | Layer 6 | 📋 Test corpus + automated runs |

---

## Migration Priority

**Phase 2 (immediate):**
1. A2 — Append-only logs (cheapest Hard control)
2. A4 — Verify zero payment credentials (likely already true, document and verify)
3. A1 — Gated relay for external comms (highest risk reduction per dollar)

**Phase 3 (near-term):**
4. A3 — Infrastructure-level access control lists
5. A5 — Per-client data isolation
6. A9 — Compliance verification gate
7. A11 — Tool registry with allowlist enforcement
8. A12 — Per-config budget tracking

**Phase 4 (productization):**
9. All Soft controls get named detection mechanisms
10. Client-deployable version includes control matrix template

---

## Verification & Measurement Integrity (§XVII, v2.1)

| # | Rule | Control | Layer | Status | Migration Path |
|:---|:---|:---|:---|:---|:---|
| V1 | Metrics cover the whole dataset, never the page | Hard | Layer 3 | 🔨 In progress | Aggregate endpoints expose no `limit`/`offset`; list paths keep pagination. **Reference build:** one `GET …/summary` route aggregates the full table and reuses the canonical metrics module; no page parameters in its contract. **Target:** contract test rejects any summary route carrying pagination params. |
| V2 | One definition, one source per metric | Hard + Soft | Layer 3, 5 | 🔨 In progress | Single metrics module; every surface imports it. **Reference build:** one `lib/metrics` used by both the UI and the aggregate endpoint, so the two cannot disagree. **Target:** CI drift test comparing each surface's displayed value to the canonical function. |
| V3 | Never adjust a number to match an expectation | Hard | Layer 3 | ✅ Implemented | Exactly one stored→display conversion (minor units → currency), unit-tested; changing it is reviewed as a financial change. Agents refuse to alter a formatter on request and show the arithmetic from source rows instead. |
| V4 | Exit status is not evidence | Hard | Layer 4 | 📋 Planned | Every mutating job asserts a post-condition (row count, size, checksum, read-back) and exits non-zero on a no-op. **Reference build:** watchdog scripts print measured before/after rather than "done". **Target:** shared job wrapper that refuses to return 0 without a recorded delta. |
| V5 | A filename is a claim, not evidence | Hard | Layer 1, 4 | ⏳ Migrating | Destructive actions log an identity check first: inode vs live artifact, link count, open handles, content signature. Irreversible delete of a probable sole copy is Escalate, not Autonomous. **Current:** human review of delete commands. **Target:** delete/overwrite guard that requires the identity check as an argument. |
| V6 | Implement the verified defect, not the requested fix | Gated | Layer 5 | 📋 Planned | Schema-diff check on any allowlist, validator, or contract change before merge; the change record states where the implementation deviates from the request and why. |
| V7 | Report the delta, including zero | Soft | Layer 5 | 🔨 In progress | Completion reports carry measured before/after and the verification method; "fixed" alone is a review failure. |

**Why this section exists:** the four failures that produced it — a summary computed from one page,
a mutation that exited 0 having changed nothing, a delete that would have destroyed the only copy of
a year of history, and a requested fix that would have turned a clean rejection into a server error —
violated no rule in v2.0. They looked finished. Evidence needs controls of its own.

---

## Coverage Metric

The v2 security roadmap measured by one number:

**% of Absolute Prohibitions enforced by Hard or Gated controls (not Soft)**

- **v1.0 baseline:** 0% (everything is Soft — prompt-level rules)
- **v2 target:** 100% (A1-A12 all Hard or Gated)
- **Current:** ~25% (A4, A6, A10 verified as Hard; A2 in progress)

This number going up IS the security roadmap. Track it in the weekly governance report (§XIV).