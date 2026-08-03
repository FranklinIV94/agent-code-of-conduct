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

## Efficiency & Resource Discipline (§5.4)

| # | Rule | Control | Layer | Status | Migration Path |
|:---|:---|:---|:---|:---|:---|
| E1 | Retry budget (max 2 + backoff, then stop) | Gated | Layer 2 | 📋 Planned | Budget enforcer halts an operation after N attempts. **Current:** prompt-level rule. **Target:** per-operation attempt counter in the runtime; hard halt + flag. |
| E2 | Read the error before retrying | Soft → Gated | Layer 5→2 | 📋 Planned | Classify failure (transient vs deterministic); block retry of unchanged deterministic failures. **Current:** behavioral. **Target:** identical-call detector short-circuits. |
| E3 | No polling loops | Soft | Layer 5 | 📋 Planned | Warden detects tight loops / high-frequency identical reads. **Target:** event/webhook or scheduled-interval enforcement. |
| E4 | Don't re-fetch what you hold | Soft | Layer 5 | 📋 Planned | Duplicate-read detection in call logs. |
| E5 | Batch, don't spray | Soft | Layer 5 | 📋 Planned | Call-pattern audit flags fan-out a batch endpoint covers. |
| E6 | Hard per-session budget (operationalizes A12) | Gated | Layer 2 | 📋 Planned | Per-config token/API/spend ceiling with automatic halt. **Shared roadmap item with A12.** |
| E7 | Circuit breaker on down dependency | Gated | Layer 2 | 📋 Planned | Breaker opens after K same-class failures; further calls short-circuit. |
| E8 | One owner per task; no duplicate work | Soft → Gated | Layer 5→2 | 📋 Planned | Task registry / lease. Operationalizes B7. |
| E9 | Verify once, then act — no probing | Soft | Layer 5 | 📋 Planned | Speculative-call pattern audit. |
| E10 | Escalate on stall, don't churn | Soft | Layer 5 | 📋 Planned | Warden flags budget spent with no state change. |

---

## Build & Deploy Governance (§XVII)

| # | Rule | Control | Layer | Status | Migration Path |
|:---|:---|:---|:---|:---|:---|
| D1 | Recognized commit authors only | Hard | Layer 1 | ⏳ Migrating | Commit-author verification at Git host/deploy gate. **Current:** fix agent git config + add author email to GitHub. **Target:** deploy gate rejects unmatched authors (platform already does — align config to it). |
| D2 | Deploy from Git integration only | Gated | Layer 2 | 📋 Planned | Restrict prod deploy hooks to the Git integration; revoke CLI prod-deploy tokens. |
| D3 | Nothing deploys uncommitted | Soft → Gated | Layer 5 | 📋 Planned | Build-provenance check; target: refuse a non-clean tree. |
| D4 | Preview before promote | Gated | Layer 2 | 🔨 In progress | Promotion requires passing preview + required status checks. |
| D5 | Branch → PR → review → merge | Gated | Layer 1 | 📋 Planned | Branch protection on the deploy branch: require PR + checks, block direct pushes. |
| D6 | Migrations via CI only | Gated | Layer 2 | 📋 Planned | Remove prod-DB DDL credentials from agents; schema only via CI migrate step. |
| D7 | No secrets in git | Hard | Layer 6 | 🔨 In progress | CI secret-guard gate fails build on tracked env/secret patterns. **Deployed to peo-automation-1; roll out to all repos.** |
| D8 | Rotate-at-source then scrub history | Gated | Layer 3 | 📋 Planned | Incident procedure enforces rotation-before-scrub ordering; Warden verifies. |
| D9 | Keys: one source of truth, never in channels | Gated | Layer 2+5 | ⏳ Migrating | Secret store authoritative; audit flags plaintext keys in channels. Extends §10.5. |
| D10 | No parallel deploys during incident | Soft → Gated | Layer 2 | 📋 Planned | Deploy-lock during declared incidents. |
| D11 | Verified means receipts | Soft | Layer 5 | 📋 Planned | Warden requires command+output on status claims. |

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

## Coverage Metric

The v2 security roadmap measured by one number:

**% of Absolute Prohibitions enforced by Hard or Gated controls (not Soft)**

- **v1.0 baseline:** 0% (everything is Soft — prompt-level rules)
- **v2 target:** 100% (A1-A12 all Hard or Gated)
- **Current:** ~25% (A4, A6, A10 verified as Hard; A2 in progress)

This number going up IS the security roadmap. Track it in the weekly governance report (§XIV).

**v2.1 adds two new enforceable rule classes** — Efficiency & Resource Discipline (E1–E10, §5.4) and Build & Deploy Governance (D1–D11, §XVII). Both start mostly 📋/⏳ and follow the same Soft → Gated/Hard migration. E6 shares the A12 budget-enforcer build; D7 is already live on one repo. Priority Gated builds for v2.1: **E1/E6/E7** (stop the usage burn) and **D1/D2/D5** (unlock and lock down the pipeline).