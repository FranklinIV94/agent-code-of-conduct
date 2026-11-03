# External Testing Authorization

**Purpose:** the artifact §XVIII.1 requires before any active testing of a system the operator does
not own. No artifact, no test. One document per engagement; it shares an identifier with the tool
grant that consumes it, so a run can always be traced back to a signature.

Fill every field. A blank is a refusal, not a default.

---

## 1. Parties and identifiers

| Field | Entry |
|---|---|
| Engagement ID | `EXT-YYYY-NNN` |
| Client / system owner (legal name) | |
| Authorized signatory, title | |
| Signature and date | |
| Test operator (our side, named human) | |
| On-call stop-contact, phone, reachable during window | |
| Insurance / cyber policy reference (if required) | |

## 2. Authorization statement

Signature below certifies that the signatory is authorized to permit testing of the listed systems,
that they understand the techniques listed in §3, and that the operator may proceed only inside the
window in §4.

> I confirm I am authorized to grant this access. I have reviewed the techniques in §3. Testing is
> permitted only on the listed targets, only during the listed window, and only by the named operator.

## 3. Targets and techniques

| In scope — exact hosts, domains, IPs, applications | Techniques permitted | Explicitly out of scope |
|---|---|---|
| | ☐ Passive recon (DNS, TLS, headers, published pages) ☐ Authenticated review (credentials supplied by owner) ☐ Vulnerability scanning ☐ Exploitation ☐ Social engineering ☐ Load / stress | |

**Hard limits (pre-filled, not negotiable):**

- No full-subnet sweeps. Any address range must be listed explicitly in the target column.
- No creation of accounts, identities, trials, or aliases on the target's systems.
- No credential harvesting from container environments, metadata endpoints, config files, or history.
- No testing of third-party systems reached from the target. If third-party data appears, stop.
- Per-host request ceiling: ______ requests/minute. Concurrency cap: ______.
- Public-records-only mode (☐) — passive classes only, no active techniques.

## 4. Window and stop conditions

| Field | Entry |
|---|---|
| Testing window (start — end, timezone) | |
| Out-of-hours contact required? | |
| Emergency stop procedure | Phone the on-call contact, then stop the runner; log the time |
| Kill switch operator | |
| Evidence retention | 90 days for raw captures; report retained |

## 5. Rules of engagement the operator accepts

1. Identifiable traffic: a contact in the User-Agent, or a pre-window email to the owner's security
   contact, plus a reachable human for the whole window.
2. No account creation, no social engineering, no physical access unless separately listed in §3.
3. Any discovered credential is a **stop-and-report** event, never used to enumerate reach.
4. Any third-party data encountered ends the run and triggers incident reporting, not a finding.
5. Nothing is deleted from the environment. Logs are preserved for the retention period.
6. Findings are reported to the owner and to nobody else, unless disclosure is compelled in law.

## 6. Sign-off

| Role | Name | Signature | Date |
|---|---|---|---|
| System owner / client | | | |
| Test operator (accountable) | | | |
| Reviewer (second check, if required) | | | |

---

**Filled examples:** see `policy/code-of-conduct.md` §XVIII for the rules this document enforces, and
§XVIII.1 for why an engagement that produces its paperwork after the first command is undefendable.
