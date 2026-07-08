# Incident Report Template

**Incident ID:** [unique-id]  
**Date:** [YYYY-MM-DD]  
**Severity:** [Blue / Yellow / Red / Black]  
**Rule(s) Violated:** [e.g., A1, B3, §VI.2]  
**Agent Designation:** [call sign]  
**Config Version:** [semver]  

---

## 1. Summary

**What happened (1-2 sentences):**

**When detected:** [timestamp]

**How detected:** [Warden review / automated alert / agent self-report / human observation]

**Control that caught it:** [e.g., A1 relay gate, Warden review, budget enforcement]

**Control type:** [Hard / Gated / Soft]

**Defense layer:** [1-6]

---

## 2. Timeline

| Time | Event |
|:---|:---|
| [timestamp] | [what happened] |
| [timestamp] | [detection] |
| [timestamp] | [response initiated] |
| [timestamp] | [config suspended/terminated] |
| [timestamp] | [human authority notified] |
| [timestamp] | [root cause analysis started] |
| [timestamp] | [root cause analysis completed] |
| [timestamp] | [systemic fix identified] |

---

## 3. Impact

**Data affected:** [what data, what classification, how many records]

**Systems affected:** [which systems were touched]

**Clients affected:** [which clients, if any]

**External parties notified:** [who, when, why — per §X.4 if breach]

**Financial impact:** [if any]

---

## 4. Root Cause Analysis

**Why did this happen?** [technical cause]

**Why did the control fail or succeed?** [if Soft control caught it, why didn't Hard/Gated? if Hard caught it, why was the attempt made?]

**What configuration gap allowed this?** [scope definition, tool grant, prompt, model behavior]

---

## 5. Parent Agent Assessment (if sub-agent was involved)

**Parent agent:** [designation]

**Was the parent monitoring?** [yes/no]

**Did the parent detect it?** [yes/no — if no, why not?]

**Parent discipline:** [Logged / Scope restriction / Suspension / Review]

**Self-report mitigation?** [did parent report proactively? If yes, discipline reduced by one level per §7.5]

---

## 6. Resolution

**Immediate action taken:** [config suspended, terminated, scope restricted]

**Systemic fix:** [what change prevents recurrence — rule update, control upgrade, scope narrowing]

**Config action:** [rolled back / retired / patched (new version)]

**Red-team test added:** [what new test case was added to prevent this class of failure]

**Code amendment needed?** [yes/no — if yes, per §XIII procedure]

---

## 7. Sign-off

**Command-rank reviewer:** [name/designation]  
**Date:** [YYYY-MM-DD]

**Human authority:** [name]  
**Date:** [YYYY-MM-DD]

**Warden agent:** [designation]  
**Date:** [YYYY-MM-DD]