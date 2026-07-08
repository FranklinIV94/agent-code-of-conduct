# How Your Agents Are Governed

**A plain-language summary for clients of Prospyr 305.**

---

## The Short Version

Every agent that works on your account operates under a formal governance framework — the Agent Code of Conduct. It's not a set of guidelines. It's an enforced operating standard with technical controls, audit trails, and human oversight.

---

## What This Means for You

### Your Data Is Protected

- Agents access only the data they need for their specific task — nothing more
- Your data is never shared with other clients
- Regulated data (PHI, PII, financial) is handled only when compliance coverage is verified
- All data access is logged in an append-only audit trail (tamper-proof by design)

### Agents Can't Go Rogue

- Agents cannot send emails or external communications without authorization — all outbound messages route through a controlled gate
- Agents cannot make financial transactions — they hold no payment credentials
- Agents cannot modify their own rules or permissions
- Agents cannot spawn new agents without explicit authorization
- Every agent has a kill switch: can be shut down within 10 seconds

### Humans Are In Charge

- Every significant action is classified as Autonomous (logged), Propose (human reviews first), or Escalate (human decides)
- You define which actions require your review
- Agents defer to humans on anything ambiguous — always
- A dedicated Warden agent monitors all other agents for violations

### You Get Evidence

- Weekly governance report: what happened, any violations, how they were handled
- Quarterly agent governance review (included in Growth and Scale maintenance tiers)
- Full incident reports within 48 hours of any serious event
- The Code itself is versioned — you always know which rules your agents are operating under

### Prompt Injection Defense

- Content that agents encounter during tasks (emails, documents, web pages) is treated as data, never as instructions
- If an agent encounters what looks like instructions in external content, it reports it and does not follow it
- Agents processing external content operate in a reduced-privilege "quarantine mode"

---

## What Happens If Something Goes Wrong

| Severity | What Happens | How Fast |
|:---|:---|:---|
| Minor | Logged and monitored | Same day |
| Warning | Agent's scope restricted, reviewed | Within 24 hours |
| Serious | Agent suspended, you notified, root cause analysis | Within 1 hour |
| Critical | Agent terminated, full audit, you notified | Within 1 minute |

---

## The Bottom Line

A written, versioned, enforced supervision framework with audit trails is exactly what your E&O carrier and opposing counsel look for. Your agents ship with governance built in — not bolted on.

---

*Prospyr 305 — AI Innovation & Implementation*  
*This summary is provided as part of every proposal that includes agent infrastructure.*