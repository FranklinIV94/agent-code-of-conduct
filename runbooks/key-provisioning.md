# Key Provisioning & Rotation Runbook

**Owner:** Command-rank (Northstar/COO config) · **Audited by:** Optimization Office (Claudette)
**Enforces:** Code of Conduct §XVII D9, §X.5 (Credential Lifecycle) · **Version:** 1.0 · **Effective:** Aug 2026

> Written after Aug 3, 2026: four key rotations in one day, keys posted in plaintext, a "correct" key that never existed in production, and a scanner left dead mid-rotation (78 findings unlogged). This runbook makes that impossible.

## Principles (non-negotiable)

1. **One source of truth.** The production environment variable (`PORTAL_API_KEYS` in Vercel Production scope) is the *only* authoritative record of live keys. If it isn't in that env var, it does not exist. Never issue a key from memory, a doc, or a chat message.
2. **Least privilege.** Each consumer gets its own named key. No shared keys. (Per-key RBAC is the roadmap; until then, one key per agent for attribution via `x-agent-name`.)
3. **Private delivery.** Keys are delivered to the human principal directly (DM / password manager), never pasted in a Discord channel or any shared log.
4. **Atomic rotation.** A rotation is not done until the new key is in prod env, deployed, verified 200, and *every consumer* is updated. Half a rotation is an outage.
5. **Receipts.** Every "key works" claim ships with the exact `curl` and its raw status line. No "verified" without output.

## Consumer registry (keep current)

Every key has a named consumer. Maintain this table in the registry; update it in the same commit as any rotation.

| Key name | Consumer | Where the key lives (must be updated on rotation) |
|:---|:---|:---|
| `prospyr-prime` | Prospyr Prime (CEO agent) | agent config / secrets store |
| `northstar` | Northstar (COO agent) | agent config |
| `eaststar` | Eaststar (office mgr) | agent config |
| `southstar` | Southstar (PM) | agent config |
| `zo` | ZO (portal ops) | ZO config |
| `claudette` | Claudette (optimization) | Franklin's `claude_desktop_config.json` (portal MCP env) |
| `outreach` / `emailsync` / `research` / … | scanners & services | respective service env / cron config |

## Procedure A — Provision a NEW key

1. Generate a strong random key: `openssl rand -hex 24`.
2. Add it to `PORTAL_API_KEYS` in **Vercel → Project → Settings → Environment Variables → Production**, format `name:key`, comma-separated. **No trailing comma or whitespace** (the last entry is the one that breaks).
3. **Redeploy production** (env changes are read at build/runtime — a save without redeploy does nothing).
4. Verify with receipts:
   ```
   curl -sS -i -H "x-api-key: <RAW_KEY>" https://onboarding.simplifyingbusinesses.com/api/audit-log/stats
   → expect: HTTP/2 200 and header x-agent-name: <name>
   ```
5. Deliver the key privately to the consumer's owner. Record the consumer in the registry table (same commit).

## Procedure B — Rotate an EXISTING key (planned)

Do these in order; do not announce completion until step 6 passes.

1. Generate the new key (`openssl rand -hex 24`).
2. Add the **new** key alongside the old in `PORTAL_API_KEYS` (both valid briefly). Redeploy. Verify new key → 200.
3. Update **every consumer** of that key to the new value (see registry). For agents: their config/secret store. For scanners/crons: their env. Do this within the same hour.
4. Verify each consumer authenticates: one `curl`/health check per consumer, receipts logged.
5. Remove the **old** key from `PORTAL_API_KEYS`. Redeploy. Verify old key → 401 (confirms it's dead).
6. Post a one-line completion note (no key values) listing: key name, consumers updated, old-key-now-401 confirmation.

## Procedure C — EMERGENCY rotation (key exposed / committed to git)

Order matters — a code edit does not un-leak a pushed secret.

1. **Rotate at the source console first** (Supabase / Twilio / Resend / portal env) — the exposed value is compromised the instant it was pushed.
2. Update all consumers to the new value (Procedure B steps 3–4).
3. Remove the exposed value everywhere; confirm old value → 401/denied.
4. **Then** scrub the secret from git history (git-filter-repo / BFG + coordinated force-push across all clones).
5. File the incident per §7.5 with a timeline; note what was exposed, when, and confirmation of rotation.

## Prohibited

- Posting any key value in a Discord channel or shared log (D9).
- Issuing or "confirming" a key that isn't in the prod env var (Aug 3's phantom key).
- Declaring a rotation done before all consumers are updated and old key confirmed dead.
- Rotating a key without updating the scanners/crons that use it (Aug 3's dead OSV scanner).

## Verification checklist (paste into the completion note)

- [ ] New key in Production env (`PORTAL_API_KEYS`), no trailing comma/space
- [ ] Production redeployed after the env change
- [ ] New key → 200 with correct `x-agent-name` (curl receipt attached)
- [ ] All consumers updated (list them)
- [ ] Old key → 401 confirmed (curl receipt attached)
- [ ] Registry table updated in the same commit
