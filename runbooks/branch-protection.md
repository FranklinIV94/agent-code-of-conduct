# Branch Protection & Deploy-Lock Standard

**Owner:** Command-rank (Northstar) · **Audited by:** Optimization Office (Claudette)
**Enforces:** Code of Conduct §XVII D2, D3, D5 · **Version:** 1.0 · **Effective:** Aug 2026

> The deploy branch is production's front door. Aug 3: uncommitted work, direct pushes, and CLI deploys made "what's live" unknowable. This standard makes the deploy branch the single, protected, reproducible source of production.

## Scope — apply to the deploy branch (`master`/`main`) of every active repo

| Repo | Deploy branch | Live surface |
|:---|:---|:---|
| `client-onboarding-portal-1` | `master` | onboarding.simplifyingbusinesses.com (portal) |
| `albs-portal` | `master` | albs-portal.vercel.app |
| `peo-automation-1` (mypeo-forms) | `master` | mypeo forms / lambda |
| `Lead-Management-System` | `master` | lead mgmt |
| `agent-code-of-conduct` | `master` | governance source |

## Required ruleset (every repo above)

1. **Require a pull request before merging.** No direct pushes to the deploy branch.
   - Require **≥1 approving review**. (Solo/agent context: require review from a human identity or the Warden config; approval may not come from the PR author.)
   - **Dismiss stale approvals** when new commits are pushed.
   - **Require conversation resolution** before merge.
2. **Require status checks to pass before merging** — and require branches to be **up to date** first:
   - `secret-guard` (fails on tracked `.env` / secret patterns — D7)
   - `build` / typecheck (no broken build reaches prod)
   - `migrations-check` (schema change ⇒ accompanying migration present — D6) *(add once CI migrate step exists)*
3. **Require linear history** (no messy merge commits; forces clean, reviewable history).
4. **Do not allow force pushes.** **Do not allow deletions.** (Protects against history loss.)
5. **Include administrators** — the rule applies to everyone, no bypass. (Emergency break-glass is a documented, logged exception only.)
6. **Restrict who can push to matching branches** — merges happen only via PR; no human or agent pushes commits straight to the branch.

## Apply via GitHub UI

`Repo → Settings → Branches → Add branch ruleset` (or classic *Add rule*) → Branch name pattern `master` → enable each item above → Save. Repeat per repo.

## Apply via `gh` CLI (scriptable — run once per repo)

```
REPO="FranklinIV94/client-onboarding-portal-1"   # repeat for each repo
gh api -X PUT "repos/$REPO/branches/master/protection" \
  -H "Accept: application/vnd.github+json" \
  -f "required_status_checks[strict]=true" \
  -F "required_status_checks[contexts][]=secret-guard" \
  -F "required_status_checks[contexts][]=build" \
  -F "enforce_admins=true" \
  -F "required_pull_request_reviews[required_approving_review_count]=1" \
  -F "required_pull_request_reviews[dismiss_stale_reviews]=true" \
  -F "required_pull_request_reviews[require_last_push_approval]=true" \
  -F "required_linear_history=true" \
  -F "allow_force_pushes=false" \
  -F "allow_deletions=false" \
  -F "required_conversation_resolution=true" \
  -F "restrictions=" 
```
> `restrictions` must be present (null = no user/team push allowlist beyond PR flow). Adjust review count if a human reviewer isn't always available; never drop below the status-check gates.

## Deploy-lock companion settings (Vercel — D2)

Branch protection secures the *repo*; these secure the *deploy*:
1. **Production deploys only from the Git integration.** In Vercel → Project → Settings → Git, keep production wired to the deploy branch. Treat `vercel deploy` to production as banned (revoke/withhold prod-scope CLI tokens from agent boxes).
2. **Git commit-author verification stays ON** (this is what blocked the bad deploys — that's correct behavior). The fix is D1: make commit authors recognized, not disabling the check. Add the author email to a GitHub account or fix the agents' git config.
3. **Preview required before promote (D4).** Promotion to production only from a preview that passed checks.

## Verification (receipts)

Per repo, after applying:
```
gh api "repos/$REPO/branches/master/protection" --jq '{pr:.required_pull_request_reviews.required_approving_review_count, checks:.required_status_checks.contexts, admins:.enforce_admins.enabled, force:.allow_force_pushes.enabled}'
```
Expect: PR review ≥1, checks include `secret-guard`+`build`, admins enforced true, force pushes false.
