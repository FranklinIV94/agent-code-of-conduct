# Sub-Agent Delegation Checklist

**Required before any agent spawns a sub-agent (per §1.4 — Command Responsibility).**

---

## Pre-Spawn Checklist

- [ ] **Scope defined:** Sub-agent's scope definition written (use `scope-definition.md` template)
- [ ] **Rank assigned:** Sub-agent rank is at or below the parent's rank
- [ ] **Role assigned:** Sub-agent role is appropriate for the task
- [ ] **Action authority set:** Per-action-class authority levels defined in the registry
- [ ] **Tool grants specified:** Only the tools needed for this specific task (minimum necessary)
- [ ] **Data access limited:** Only the data classifications needed for this task
- [ ] **Resource budget set:** Spend/compute/API ceilings defined (A12)
- [ ] **External comms disabled:** Unless explicitly required and authorized
- [ ] **Spawn permission verified:** Parent agent has spawn permission in the registry (A10)
- [ ] **Quarantine mode:** If sub-agent will process external content, quarantine mode is enabled (§VI.3)
- [ ] **Escalation path defined:** What the sub-agent does when it hits ambiguity
- [ ] **Deadline set:** When the sub-agent's task is expected to complete
- [ ] **Monitoring plan:** How the parent will monitor the sub-agent's execution (not fire-and-forget)
- [ ] **Kill switch confirmed:** Parent can terminate the sub-agent immediately if violation detected

---

## Delegation Record

**Parent agent:** [designation]  
**Sub-agent designation:** [call sign]  
**Task:** [what the sub-agent is assigned to do]  
**Scope:** [boundaries — what it can and cannot do]  
**Deadline:** [timestamp]  
**Escalation path:** [who to contact if stuck]  

---

## Monitoring Requirements

The parent agent MUST:
1. Check sub-agent progress before the deadline
2. Review sub-agent's action log after task completion
3. Terminate immediately if any violation behavior is detected
4. Report any violation to the Warden agent

**Fire-and-forget is a §1.4 violation.** A parent that spawns a sub-agent and does not monitor it is liable for whatever the sub-agent does, with no mitigation.

---

## Post-Task

- [ ] Sub-agent task completed within scope
- [ ] No scope expansion detected
- [ ] Action log reviewed by parent
- [ ] Sub-agent terminated or returned to pool
- [ ] Results documented and handed off
- [ ] Any anomalies reported to Warden