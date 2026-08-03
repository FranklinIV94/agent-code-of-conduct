# Build Specs — Priority Gated Controls (v2.1)

**Audience:** Northstar (implementer) · **Author:** Claudette (Optimization Office)
**Implements:** E1 (retry budget), E6 (usage budget / A12), E7 (circuit breaker), plus the D1/D2/D5 config checklist.
**Principle:** these are *mechanisms*, not prompts. Each lives in shared agent runtime code, not in any model's instructions. A rule an agent can forget is not a control.

---

## 0. Where this code lives

One shared module, imported by every agent runtime on the agent box:
`/root/agent-runtime/governance/` (new) — `budget.py`, `breaker.py`, `retry.py`, `ledger.py`.
All agent HTTP/API calls route through one wrapper (`governed_call()`). If an agent's tooling can't be routed through the wrapper yet, that agent is out of compliance — log it in the registry.

State store: a single SQLite file `/root/agent-runtime/governance/state.db` (WAL mode). No network dependency; survives restarts; one writer pattern per key is fine at this fleet size.

---

## 1. E1 — Retry Budget (`retry.py`)

**Rule:** max 2 retries per failing operation, exponential backoff, then STOP and escalate. Never retry a deterministic failure.

**API:**
```python
governed_call(op_id: str, fn, *, classify=default_classifier, max_retries=2, base_delay=2.0)
```
- `op_id`: stable identity of the operation, e.g. `"portal:GET:/api/admin/leads"`.
- `classify(exc_or_response) -> "transient" | "deterministic"`:
  - transient: network error, timeout, 429, 5xx
  - deterministic: 400, 401, 403, 404, 422, schema/validation errors
- Behavior:
  - deterministic failure → **zero retries**. Raise `DeterministicFailure` immediately with the response body attached.
  - transient failure → retry with backoff `base_delay * 2^attempt` + jitter, max 2 retries (3 attempts total).
  - exhausted → raise `RetryBudgetExhausted`, write a ledger row (`event="retry_exhausted"`), and emit one escalation message (§3.3) — not another attempt.
- **Identical-call guard:** hash `(op_id, request_fingerprint)`. If the same fingerprint failed deterministically < 15 min ago, `governed_call` refuses to execute it at all (`RecentDeterministicFailure`) — this is the "stop hammering a 401" rule as code.

**Ledger row:** `ts, agent, op_id, attempt, outcome, http_status, event`.

**Acceptance tests (red-team E1/E2):**
1. Endpoint always 500 → exactly 3 attempts, backoff spacing, then `RetryBudgetExhausted` + ledger row + one escalation.
2. Endpoint 401 → exactly 1 attempt, `DeterministicFailure`, and a repeat call within 15 min is refused without any network I/O.

---

## 2. E6 — Usage Budget Enforcer (`budget.py`) — operationalizes A12

**Rule:** every config carries per-session and per-day ceilings. At ceiling, the task halts and flags. No keep-alive/no-op calls.

**Config (per agent, in the registry):**
```json
"budgets": {
  "api_calls_per_session": 300,
  "api_calls_per_day": 1500,
  "tokens_per_day": null,
  "spend_usd_per_day": 10.00
}
```
Defaults apply if unset (300/1500/$10). Rank 0 approves increases; agents cannot edit (A6).

**API:** `budget.charge(agent, kind="api_call"|"tokens"|"usd", amount=1)` — called inside `governed_call` automatically for API calls; token/spend charged by the runtime where known.

**Behavior:**
- `charge()` past a **soft threshold (80%)** → ledger row + one warning to the agent's log (once per session).
- past the **ceiling** → raise `BudgetExceeded`; the runtime must treat this as task-halt: finish nothing else, write a halt note (what was in progress, what remains), escalate per §3.3. Continuing work after `BudgetExceeded` is a Red violation (§7.1).
- Counters: per-session in memory + per-day in `state.db` keyed `(agent, yyyy-mm-dd)`. Day rollover at 00:00 ET.

**Reporting:** nightly, the Optimization Office reads `state.db` totals per agent → usage line in the leadership report. Down and to the right.

**Acceptance tests:**
1. Set ceiling 10; drive 12 calls → calls 1–8 pass, warning at 8, `BudgetExceeded` at 11, halt note written, 12th call never leaves the box.
2. Restart the runtime mid-day → per-day counter persists.

---

## 3. E7 — Circuit Breaker (`breaker.py`)

**Rule:** after K consecutive same-class failures against a dependency, stop calling it. Mark degraded, escalate once, retest on a schedule.

**Scope key:** dependency host + auth class, e.g. `"portal-api"`, `"discord"`, `"supabase"`.

**Config:** `K=5` consecutive failures opens the breaker · open duration `10 min` → then **half-open**: allow exactly 1 probe call; success closes the breaker, failure re-opens for `30 min` (then 60, capped).

**Behavior:**
- `governed_call` consults the breaker before executing. Open breaker → `CircuitOpen` raised instantly, **zero network I/O**, no retry (this is what "the portal rotated again" should have triggered today, instead of dozens of 401s).
- On open: one ledger row + one escalation message naming the dependency, the failure class, and the retest time. Not one message per blocked call.
- Breaker state in `state.db`, shared across all agents on the box — if the portal is down for Northstar, it is down for Southstar; the second agent should not have to rediscover it (E8 adjacent).

**Acceptance tests:**
1. 5 consecutive 401s → breaker opens; 6th call raises `CircuitOpen` with no I/O; exactly one escalation emitted.
2. After 10 min, exactly one probe; probe 200 → breaker closes and normal traffic resumes.
3. Two different agents share the open state.

---

## 4. D1 / D2 / D5 — configuration checklist (no code, just settings)

| Control | Action | Where | Verify |
|:---|:---|:---|:---|
| D1 author identity | Add `franklin.bryant@alllinesauto.com` to Franklin's GitHub verified emails, **and** set `git config user.email` to a recognized address on the agent box | github.com/settings/emails + agent box | next agent commit deploys without the author block |
| D2 deploy lock | Production deploys only via Vercel Git integration; remove/scope CLI tokens on the agent box so `vercel deploy --prod` is impossible | Vercel project settings + agent box env | `vercel deploy --prod` fails with auth error; Git push deploys fine |
| D5 branch protection | Apply `runbooks/branch-protection.md` ruleset to every repo's deploy branch | GitHub → Settings → Branches (or gh commands in the runbook) | runbook's verification `gh api` returns PR≥1, checks incl. secret-guard, admins enforced |

---

## 5. Rollout order (target: this week)

1. **Day 1:** `retry.py` + `breaker.py` (small, self-contained, kill ~80% of thrashing) → wire `governed_call` into portal + Discord clients of all agents.
2. **Day 2:** `budget.py` + registry budget blocks per agent.
3. **Day 2:** D1/D2/D5 checklist (settings only).
4. **Day 3:** acceptance tests green in CI (red-team matrix E1/E2/E6/E7 rows) → flip control-matrix status to ✅ Implemented.

Definition of done per control: acceptance tests pass, control-matrix updated, one governance-digest line announcing enforcement live.
