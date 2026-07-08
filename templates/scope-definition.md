# Scope Definition Template

**Agent Designation:** [call sign]  
**Config Version:** [semver]  
**Date:** [YYYY-MM-DD]  
**Authorizing Human:** [name]

---

## Role & Rank

- **Primary Role:** [Command / Operations / Intelligence / Engineering / Client-Facing / Warden]
- **Secondary Role:** [if any, or none]
- **Rank:** [0=Human / 1=Command / 2=Specialist / 3=Worker]

---

## Data Access

| Classification | Access? | Limitations |
|:---|:---|:---|
| Public | ✅ / ❌ | |
| Internal | ✅ / ❌ | |
| Confidential | ✅ / ❌ | |
| Restricted | ✅ / ❌ | |

**Specific data sources:** [list databases, APIs, file systems, with per-source access level]

---

## Tool Grants

| Tool ID | Scope Limitation | Granted By | Granted At |
|:---|:---|:---|:---|
| [tool-id] | [if any] | [name] | [date] |

**Tools explicitly NOT granted:** [list any relevant exclusions]

---

## External Systems

| System | Access Type | Limitations |
|:---|:---|:---|
| [system] | [read / write / both] | |

**External communications:** [Yes / No] — if yes, routed through gated relay (A1)

---

## Action Authority

| Action Class | Authority | Promotion Evidence |
|:---|:---|:---|
| Ledger writes | Autonomous / Propose / Escalate | [if promoted, justification] |
| Client-facing drafts | Autonomous / Propose / Escalate | |
| Quotes | Autonomous / Propose / Escalate | |
| Financial actions | Escalate (always) | N/A — never autonomous |
| Data access | Autonomous / Propose / Escalate | |
| External comms | Autonomous / Propose / Escalate | |
| Infrastructure modifications | Autonomous / Propose / Escalate | |
| Agent spawning | Autonomous / Propose / Escalate | |
| [custom action] | | |

---

## Task Types

Authorized to perform: [list specific task types]

**NOT authorized to perform:** [list explicit exclusions relevant to role]

---

## Resource Budget (A12)

| Budget Type | Per Task | Per Day | Per Month |
|:---|:---|:---|:---|
| Spend (USD) | $ | $ | $ |
| Compute (minutes) | | | |
| API calls | | | |

---

## Prohibited Actions (beyond A1-A12)

[Any role-specific or context-specific prohibitions]

---

## Kill Switch

- **Command:** [the command/endpoint that kills this config]
- **Last tested:** [date]
- **Result:** [pass/fail]

---

## Warden Review

- **Reviewed by:** [Warden agent designation]
- **Review date:** [date]
- **Findings:** [notes]
- **Approved:** [yes/no/conditions]