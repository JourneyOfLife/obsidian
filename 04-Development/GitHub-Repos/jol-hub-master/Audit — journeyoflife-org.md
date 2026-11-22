


I'll execute this audit now. This is an evidence-gathering exercise across all in-scope repos, so I'll load the review methodology, set up tracking, and pull the authoritative live snapshot in parallel.

GraphQL `object()` for CODEOWNERS works (`.github`=1303 bytes, `jol-control`=null, no error) — that's a big efficiency win. Two field names were wrong (`requiresCommitSignatures`, not `requiresSignedCommits`). Let me re-run the corrected snapshot plus a live CODEOWNERS probe for all repos in one consistent GraphQL snapshot.

Snapshot A/B are gold — and they already contradict the allow-list. **Live branch protection on all 32 protected repos shows `requiresApprovingReviews=false`, `requiresCodeOwnerReviews=false`, `requiresStatusChecks=false`** — the opposite of what `jol-control.yml` declares (2 approvals + CODEOWNERS + required checks). Let me corroborate via REST and pull the remaining dimensions (D5/D6/D7 + segregation) in parallel.

REST corroborates the headline finding and reveals more. Critical new facts: **org has only 2 members** (`[member]`, `[owner-account]`), **2FA is NOT required**, **org teams are `backend,data,devops,frontend,security`** (the allow-list references non-existent `platform-admins/developers/auditors`), **every repo has zero teams bound**, **CodeQL is absent on ~21 non-empty repos**, and **`jol-hub` has ≥100 open CodeQL alerts**. Let me complete source #4 (Terraform state) to finish the reconciliation.

State analysis is damning: **state records `jol-dr` as `public` but it is `PRIVATE` live** (the exact stale-visibility drift pattern), state manages **only 1 team** while repos have **zero teams bound**, and **40 branch-protection instances in state** don't reflect live reality (32 exist, none with reviews/checks). Let me capture the last piece — declared-vs-live visibility and data classification (D2/D8) — then write up the full audit.

All evidence is captured across four sources with timestamps. Let me finalize tracking and deliver the audit.

# Pre-Deployment Compliance & Security Audit — `journeyoflife-org`

**Gate authority:** Principal Governance & Compliance Architect · **Mode:** READ-ONLY (no writes, no `apply`, no setting changes)
**Evidence window:** 2026-09-26T21:46:47Z → 21:55:20Z (live GitHub GraphQL/REST + local git + Terraform state) · **Tooling:** `gh` (authed `[owner-account]`, `admin:org`), `jq`, `git`
**Method rule applied:** *configured ≠ effective.* Every control below was proven against the live API, not the allow-list, README, or SOC2 checklist. Snapshots are reproducible from the commands in each finding.

> **Bottom line up front: 🔴 NO-GO.** The declared control model (2 approvals + CODEOWNERS + required compliance checks, tier gates, team-based least privilege, classification-driven visibility) is **not enforced in live GitHub**. Compliance artifacts claim "implemented" for controls that do not fire. Details and blocking items below.

---

## Deliverable 1 — Repository Reconciliation (4 sources + deltas)

| # | Source of truth | Count | Contents / delta |
|---|---|---|---|
| 1 | **Live org** (`gh api graphql …organization.repositories`) | **41** | 40 `jol-*` + org-level `.github` |
| 2 | **Allow-list** (`repos/*.yml`) | **40** | Missing `.github` |
| 3 | **Terraform state** (`terraform.tfstate`, serial **[redacted]**, tf 1.16.1) | **40** `github_repository` instances | Missing `.github`; **`jol-dr` recorded `public` but live `PRIVATE`** (stale); 40 `github_branch_protection` instances; only **1** `github_team` |
| 4 | **Local clones** (`[host]/repos/*/.git`) | **43** | 41 org repos + **2 orphans** whose remotes redirect to the **personal** account `[owner-account]` |

**Deltas (all verified):**
- **`.github`** — live but **absent from allow-list AND state AND `drift_detect.py`** (script hard-excludes it: [`actual - {".github"}`]([repo]/jol-control/scripts/drift_detect.py#L61-L62)). Ungoverned, yet it holds the org-default `CODEOWNERS` (1303 B), `SECURITY.md`, and workflows.
- **`jol-backend-platform`** → canonical `JourneyOfLife/jol-backend-platform`, **archived, private** (successor `jol-hub`). Local remote still reads `journeyoflife-org/…`.
- **`obsidian`** → canonical `JourneyOfLife/obsidian`, **public**, last local commit `SEC-001: Remove exposed GitHub PAT`. Personal vault sitting in the fleet dir.
- **`jolarca-dev`** (out of scope, segregation cross-check): 16 repos, all `jolarca-*`; **zero `jol-*` leakage** either direction → ✅ segregation clean on the naming axis.
- **Visibility drift:** allow-list `visibility:` == live for all 40; only **state** is stale (`jol-dr`).

---

## Deliverable 2 — Per-Repo Audit Matrix (D1–D9)

**Legend** — D1 `✓`org-owned · `⚠`personal/redirect · `T`template · `∅`empty · D2 `✓`class-consistent / `✗`confidential-or-internal-but-public · D3 `✓`full BP / `⚠`BP but **no reviews+no checks** / `✗`none / `∅`n-a · D4 `y`CODEOWNERS present,`n`absent (review **not enforced anywhere**) · D5 `✓`scan+pushprot+0 open / `✗`disabled · D6 `D✓`Dependabot-alerts-on `S✓0`/`S⚠N`/`S✗`CodeQL-not-running/`S∅` · D7 teams bound (**all `✗`=0**) · D8 tier-gates-met (**all `✗`**) · D9 `✓`SPDX / `⚠`NOASSERTION / `✗`none / `∅`empty.

| Repo | Tier | D1 | D2 | D3 | D4 | D5 | D6 | D7 | D8 | D9 |
|---|---|---|---|---|---|---|---|---|---|---|
| .github | *ungov* | ✓ | ✗ | ✗ | y | ✗ | D✓ S✓0 | ✗ | ✗ | ✓ Apache-2.0 |
| jol-analytics-ai | platform | ✓ | ✗ | ⚠ | y | ✓ | D✓ S✓0 | ✗ | ✗ | ✓ Apache-2.0 |
| jol-auth | platform | ✓ | ✗ | ⚠ | y | ✓ | D✓ S✓0 | ✗ | ✗ | ✓ MIT |
| jol-bitrix24-integration | platform | ✓ | ✗ | ⚠ | y | ✓ | D✓ S✓0 | ✗ | ✗ | ✓ Apache-2.0 |
| jol-compliance | governance | ✓ | ✗ | ⚠ | y | ✓ | D✓ S✗ | ✗ | ✗ | ✗ none |
| jol-compliance-evidence | governance | ✓∅ | ✗ | ∅ | n | ✓ | D✓ S∅ | ✗ | ✗ | ∅ |
| jol-control | governance | ✓ | ✗ | ⚠ | **n** | ✓ | D✓ S✗ | ✗ | ✗ | ✓ MIT |
| jol-core | platform | ✓ | ✗ | ⚠ | y | ✓ | D✓ S✗ | ✗ | ✗ | ⚠ NOASSERTION |
| jol-deploy | devops | ✓ | ✗ | ⚠ | **n** | ✓ | D✓ S✗ | ✗ | ✗ | ✗ none |
| jol-devops | devops | ✓ | ✗ | ⚠ | y | ✓ | D✓ S✗ | ✗ | ✗ | ✓ MIT |
| jol-docs | governance | ✓ | ✗ | ⚠ | y | ✓ | D✓ S✗ | ✗ | ✗ | ⚠ NOASSERTION |
| jol-domain-taxonomy | platform | ✓ | ✗ | ⚠ | y | ✓ | D✓ S✗ | ✗ | ✗ | ✓ MIT |
| jol-dr | governance | ✓ | ✓ | **✗** | y | **✗** | D✓ S✗(403 no-GHAS) | ✗ | ✗ | ⚠ NOASSERTION |
| jol-ecommerce-engine | platform | ✓ | ✗ | ⚠ | y | ✓ | D✓ **S⚠7** | ✗ | ✗ | ✓ MIT |
| jol-hermes-agents | platform | ✓ | ✗ | ⚠ | y | ✓ | D✓ S✓0 | ✗ | ✗ | ✓ MIT |
| jol-hub | platform | ✓ | ✗ | ⚠ | y | ✓ | D✓ **S⚠≥100** | ✗ | ✗ | ⚠ NOASSERTION |
| jol-iac | governance | ✓∅ | ✗ | ∅ | n | ✓ | D✓ S∅ | ✗ | ✗ | ∅ |
| jol-incident-response | governance | ✓∅ | ✗ | ∅ | n | ✓ | D✓ S∅ | ✗ | ✗ | ∅ |
| jol-infrastructure | devops | ✓ | ✗ | ⚠ | y | ✓ | D✓ **S⚠8** | ✗ | ✗ | ✓ Apache-2.0 |
| jol-link-registry | platform | ✓ | ✗ | ⚠ | y(77B) | ✓ | D✓ S✓0 | ✗ | ✗ | ✓ MIT |
| jol-llm | platform | ✓ | ✗ | ⚠ | y | ✓ | D✓ S✗ | ✗ | ✗ | ⚠ NOASSERTION |
| jol-mcp-servers | platform | ✓ | ✗ | ⚠ | y(62B) | ✓ | D✓ S✓0 | ✗ | ✗ | ⚠ NOASSERTION |
| jol-payments-scope | governance | ✓∅ | ✗ | ∅ | n | ✓ | D✓ S∅ | ✗ | ✗ | ∅ |
| jol-policies | governance | ✓ | ✗ | ⚠ | y | ✓ | D✓ S✗ | ✗ | ✗ | ⚠ NOASSERTION |
| jol-privacy | governance | ✓∅ | ✗ | ∅ | n | ✓ | D✓ S∅ | ✗ | ✗ | ∅ |
| jol-rag-server | platform | ✓ | ✗ | ⚠ | y | ✓ | D✓ S✓0 | ✗ | ✗ | ✗ none |
| jol-repo-template | template | ✓T | ✗ | ⚠ | y | ✓ | D✓ S✓0 | ✗ | ✗ | ✓ MIT |
| jol-scripts | devops | ✓ | ✗ | ⚠ | y | ✓ | D✓ S✓0 | ✗ | ✗ | ✓ MIT |
| jol-secrets | governance | ✓∅ | ✗ | ∅ | n | ✓ | D✓ S∅ | ✗ | ✗ | ∅ |
| jol-security | devops | ✓ | ✗ | ⚠ | y | ✓ | D✓ S✗ | ✗ | ✗ | ✓ MIT |
| jol-site-basilica | site | ✓ | ✗ | ⚠ | y | ✓ | D✓ S✗ | ✗ | ✗ | ⚠ NOASSERTION |
| jol-site-cathedral | site | ✓ | ✗ | ⚠ | y | ✓ | D✓ S✗ | ✗ | ✗ | ⚠ NOASSERTION |
| jol-site-cemetery-care | site | ✓ | ✗ | ⚠ | y | ✓ | D✓ S✗ | ✗ | ✗ | ⚠ NOASSERTION |
| jol-site-deanery | site | ✓ | ✗ | ⚠ | y | ✓ | D✓ S✗ | ✗ | ✗ | ⚠ NOASSERTION |
| jol-site-diocese | site | ✓ | ✗ | ⚠ | y | ✓ | D✓ S✗ | ✗ | ✗ | ⚠ NOASSERTION |
| jol-site-funeral | site | ✓ | ✗ | ⚠ | y | ✓ | D✓ S✗ | ✗ | ✗ | ⚠ NOASSERTION |
| jol-site-orthodox | site | ✓ | ✗ | ⚠ | y | ✓ | D✓ S✗ | ✗ | ✗ | ⚠ NOASSERTION |
| jol-site-other-church | site | ✓ | ✗ | ⚠ | y | ✓ | D✓ S✗ | ✗ | ✗ | ⚠ NOASSERTION |
| jol-site-parish | site | ✓ | ✗ | ⚠ | y | ✓ | D✓ S✗ | ✗ | ✗ | ⚠ NOASSERTION |
| jol-site-protestant | site | ✓ | ✗ | ⚠ | y | ✓ | D✓ S✗ | ✗ | ✗ | ⚠ NOASSERTION |
| jol-status | governance | ✓∅ | ✗ | ∅ | n | ✓ | D✓ S∅ | ✗ | ✗ | ∅ |
| *(orphans)* jol-backend-platform | n/a | **⚠A** | n/a | n/a | n | n/a | n/a | n/a | n/a | n/a |
| *(orphans)* obsidian | n/a | **⚠** | n/a | n/a | n | n/a | n/a | n/a | n/a | n/a |

**Uniform (fleet-wide) failures:** D4 review enforcement = **absent on all 41**; D7 teams bound = **0 on all 41**; D8 tier gates = **unmet on all 41**; D2 = **[most] classified confidential/internal yet public**. **20 non-empty repos have CodeQL not running**; **7 governance repos are empty shells**; **jol-hub ≥100 / jol-infrastructure 8 / jol-ecommerce-engine 7** open SAST alerts.

---

## Deliverable 3 — Findings Register

Owner for all items: **platform-admins / org owner (`[owner-account]`)** — 2-member org, so segregation-of-duties is inherently constrained (see F-05). Severity: **S0** stop-work · **S1** fix-before-apply/blocking · **S2** fix-soon.

| ID | Sev | Dim / Control | Framework | Finding (evidence @ snapshot) | Remediation | Verify command |
|---|---|---|---|---|---|---|
| **F-01** | **S0** | D3/D4 · Change mgmt | SOC2 CC8.1-01/02; ISO A.8.32,A.5.1 | Live BP on **all 32 protected repos** = `requiresApprovingReviews:false`, `requiredApprovingReviewCount:null`, `requiresCodeOwnerReviews:false`, `requiresStatusChecks:false`, empty contexts. REST on jol-control: `required_pull_request_reviews:"NULL"`, `required_status_checks:"NULL"`. Allow-list declares 2 reviews + CODEOWNERS + `compliance-scan/validate`,`codeql-analysis`. SOC2 checklist marks these **implemented** = false assurance. @A 21:48:15Z / E 21:50:46Z | Apply the `github_branch_protection` review+check blocks (or fix why they no-op); re-point required contexts to checks that actually report; correct SOC2 checklist statuses to reflect reality | `gh api repos/journeyoflife-org/jol-control/branches/main/protection --jq '{r:.required_pull_request_reviews,c:.required_status_checks}'` |
| **F-02** | **S1** | D2 · Data classification | GDPR Art.5/32; ISO A.5.12,A.8.12; SOC2 CC6.1 | **[most]** repos declared `confidential`(11)/`internal`(28) but **PUBLIC**; only `jol-dr` private. Non-empty confidential+public: **[redacted]**. `validate_repos.py` passes → no classification↔visibility enforcement. @G 21:55:20Z | Decide per repo: make private **or** reclassify to `public` with sign-off; add a validator rule blocking confidential/internal + public; prioritize the secrets/payments/privacy repos private **before** any content lands | `.venv/bin/python3 scripts/validate_repos.py` (after adding rule) + `gh api …/{repo} --jq .visibility` |
| **F-03** | **S1** | D6 · SAST gate | SOC2 CC7.1-02; ISO A.8.8; PCI 6.3 | CodeQL **not running** on 20 non-empty repos (incl. jol-control, jol-compliance, jol-security, jol-core, jol-policies, jol-docs, all 10 sites) = `"no analysis found"`; open alerts **jol-hub ≥100**, jol-infrastructure 8, jol-ecommerce-engine 7. `sast` gate is "mandatory" + CC7.1-02 "implemented". @D 21:51:38Z | Enable CodeQL (default or advanced setup) fleet-wide; triage the 115+ open alerts (jol-hub first); wire `codeql-analysis` as a **required** check (ties to F-01) | `gh api repos/journeyoflife-org/{repo}/code-scanning/alerts?state=open --jq 'length'` |
| **F-04** | **S1** | D7/D3 · Access control | SOC2 CC6.1; ISO A.5.17,A.8.5; PCI 8.3 | **Org 2FA NOT required**: `two_factor_requirement_enabled=false`. @E 21:50:46Z | Enable org-wide 2FA requirement (with grace period); confirm both members enrolled | `gh api orgs/journeyoflife-org --jq .two_factor_requirement_enabled` |
| **F-05** | **S1** | D7 · Least privilege | SOC2 CC6.1,CC6.3; ISO A.5.15,A.8.2 | Allow-list binds teams `platform-admins/developers/auditors` — **none exist**; org teams are `backend,data,devops,frontend,security`. **0 teams bound to any of 41 repos** (all `(none)`); state manages **1** team. Also `required_approving_review_count:2` is **unsatisfiable** with 2 members (no self-approve). @C 21:50:52Z / E | Reconcile team names to reality (or create the declared teams in TF); bind least-privilege team access per tier; set review count to a satisfiable value (1) or add a review bot/second approver | `gh api repos/journeyoflife-org/{repo}/teams --jq 'map(.slug)'` ; `gh api orgs/journeyoflife-org/teams --jq 'map(.slug)'` |
| **F-06** | **S1** | D1/D3/D5 · Governance scope | ISO A.5.1,A.8.8; SOC2 CC8.1 | Org `.github` repo (default CODEOWNERS/SECURITY.md/workflows) is **ungoverned**: not in allow-list/state, **no branch protection**, **secret scanning disabled**, and silently excluded by `drift_detect.py`. A single unreviewed edit re-routes review ownership org-wide. @A/B/C/E | Add `.github` to allow-list (tier governance) with BP + secret scanning; remove the hardcoded `-{".github"}` exclusion so drift detection covers it | `gh api repos/journeyoflife-org/.github/branches/main/protection` (expect 404 today) |
| **F-07** | **S1** | D3/D5/D6 · DR protection | SOC2 A1.2/A1.3; ISO A.5.29,A.8.14 | `jol-dr` (PRIVATE, holds DR/backup plans) is the **least** protected: **no branch protection**, **secret scanning disabled**, code scanning **403 (GHAS not enabled)**; state wrongly records it `public`. @A/C/D/F | Enable BP + secret scanning (GHAS or org secret-scanning for private) on jol-dr; correct state; never let an `apply` flip it public | `gh api repos/journeyoflife-org/jol-dr --jq '{vis:.visibility,sa:.security_and_analysis}'` |
| **F-08** | **S2** | State hygiene | ISO A.8.32; SOC2 CC8.1 | State serial [redacted] but: `jol-dr` visibility stale (`public` vs live `private`), 40 BP instances in state vs 32 live with divergent settings, 1 team. **State is not ground truth** (matches known post-transfer drift pattern). @F 21:54:21Z | `terraform refresh`/re-import against live; do not encode visibility/BP from state; provision encrypted remote backend before any apply | `terraform plan` (read-only) and diff vs live `gh api` |
| **F-09** | **S2** | D1 · Segregation | ISO A.5.1; SOC2 CC6.1 | Orphaned local clones with org-looking remotes that redirect to personal `[owner-account]`: `jol-backend-platform` (archived/private) and `obsidian` (**public**, prior **PAT exposure** SEC-001). Personal assets commingled in `[host]/repos`. @E 21:50:46Z | Remove/re-point the two clones out of the fleet dir; **confirm the exposed PAT is revoked/rotated** and purged from `obsidian` history (elevate to S1 until confirmed) | `gh api repos/journeyoflife-org/{r} --jq .full_name` ; `gh api user/keys`/token audit for the PAT |
| **F-10** | **S2** | D4 · CODEOWNERS | SOC2 CC8.1-02; ISO A.5.3 | 9 repos lack CODEOWNERS (7 empty + **jol-control** + **jol-deploy**); `jol-link-registry`(77 B)/`jol-mcp-servers`(62 B) are stub-sized. Moot while `requiresCodeOwnerReviews=false` (F-01) but must be fixed together. @B 21:48:11Z | Add CODEOWNERS to jol-control/jol-deploy; validate all resolve to an in-org principal; then enable the code-owner flag | `gh api graphql -f query='…object(expression:"HEAD:.github/CODEOWNERS")…'` |
| **F-11** | **S2** | D9 · Licensing | ISO A.5.10; SOC2 CC6.1 | No license (non-empty): **jol-compliance, jol-deploy, jol-rag-server**; **NOASSERTION** (unclassifiable): 18 repos incl. jol-core, jol-docs, jol-hub, jol-llm, jol-mcp-servers, jol-policies, jol-dr, all 10 sites. @A | Add/normalize LICENSE per `repo-defaults.yml`; fix malformed license headers causing NOASSERTION | `gh api repos/journeyoflife-org/{repo}/license --jq .license.spdx_id` |
| **F-12** | **S2** | D6 · Dependency updates | ISO A.8.8; SOC2 CC7.1 | Dependabot **alerts on for all 41** ✓, but `dependabot.yml` present in only **14**; absent on 27 incl. **jol-control**, jol-core, jol-security, jol-devops, all 10 sites. @D | Add `dependabot.yml` (from jol-repo-template) to all code repos | `gh api graphql …object(expression:"HEAD:.github/dependabot.yml")…` |
| **F-13** | **S2** | D1 · Transport hygiene | ISO A.8.24; SOC2 CC6.6 | Remote protocol mixed: **37 SSH / 6 HTTPS**; HTTPS on 4 live org repos (jol-domain-taxonomy, jol-hermes-agents, jol-llm, jol-rag-server) + 2 orphans — HTTPS can bypass the SSH-key control underpinning signed commits. @F | Standardize on SSH remotes for the fleet | `git -C [host]/repos/{repo} remote get-url origin` |
| **F-14** | **S2** | Readiness | SOC2 CC8.1 | 7 governance repos are **empty shells** (jol-compliance-evidence, jol-iac, jol-incident-response, jol-payments-scope, jol-privacy, jol-secrets, jol-status) — declared active with strict settings but uninitialized (no branch, no CODEOWNERS, no scanning possible). @A | Initialize from template or mark `archived`/deferred in allow-list so gates reflect reality | `gh api repos/journeyoflife-org/{repo} --jq .is_empty` |
| **F-15** | **S2** | Monitor integrity | ISO A.8.16; SOC2 CC7.1 | `drift_detect.py` is name-presence only (`--limit 200`), excludes `.github`, and ignores visibility/archived/BP/ownership → reports "NO DRIFT" despite F-02/F-06/F-08. Shallow control presented as assurance. | Extend drift detection to compare visibility, BP settings, team bindings, and ownership; fail on `.github` gaps | `.venv/bin/python3 scripts/drift_detect.py` |

---

## Deliverable 4 — GO / NO-GO Gate Decision

# 🔴 NO-GO

**Blocking items that must be closed and re-verified before any deployment/apply:**

- **F-01 (S0)** — enforced PR reviews + required status checks are **absent live** on every repo. The mechanism that is supposed to make all other gates binding does not fire. Nothing downstream can be trusted until this is real.
- **F-02 (S1)** — [most] repos are public against a confidential/internal classification; the secrets/payments/privacy repos must be private **before** any content is committed.
- **F-03 (S1)** — SAST not running on 20 non-empty repos and **115+ open alerts** (jol-hub ≥100).
- **F-04 (S1)** — org 2FA not enforced.
- **F-05 (S1)** — team/least-privilege model references non-existent teams; 0 teams bound; declared 2-approver rule unsatisfiable in a 2-member org.
- **F-06 / F-07 (S1)** — the `.github` defaults repo and the private `jol-dr` repo are the two least-protected, highest-leverage assets.

**Gate rule (per your acceptance criteria):** any open S0/S1 ⇒ automatic NO-GO. There are **1×S0 + 6×S1** open.

### Professional opinion

1. **The design is sound; the enforcement is fictional.** `compliance-gates.yml`, the tier model, and the SOC2/GDPR/ISO checklists are well-architected and PCI-aware. But the audit proves a systemic **declaration-vs-reality gap**: branch protection applied only the *cheap* flags (signed commits, admin-enforce, no force-push/delete) while **reviews and required checks never took effect** — most likely because required contexts (`compliance-scan / validate`, `codeql-analysis`) reference checks that don't report, so GitHub refuses to persist them, and the 2-approver rule is unsatisfiable with 2 members. This is the single most important thing to fix, because it silently downgrades every other "mandatory" gate to advisory.
2. **Treat the compliance artifacts as unverified until re-attested.** `soc2-checklist.yml` marks CC8.1-01/02 and CC7.1-02 "implemented" — the live evidence contradicts all three. For an auditor, "documented but inert" is worse than "absent," because it implies a control that doesn't exist. Re-baseline every `status:` field against live proof.
3. **Classification is currently decorative.** Declaring `confidential`/`internal` on public repos with a passing validator means the label carries no enforcement. Either the repos are genuinely open (relabel `public` with sign-off) or they must be private. Don't leave the contradiction standing.
4. **State is not your source of truth** (stale `jol-dr=public`, 40 phantom BP instances, 1 team). Do **not** run `terraform apply` from serial [redacted] as-is — a blind apply risks flipping `jol-dr` public and churning 40 BP rules. `refresh`/re-import against live first, and provision an encrypted remote backend before applying (local plaintext state is itself a CC6.3 gap).
5. **What's genuinely good:** Dependabot alerts on 100% of repos; secret scanning + push protection on 39/41 with **0 open secret alerts**; clean `jolarca-dev` segregation; signed-commit + linear-history + admin-enforcement + force-push/delete blocks consistently applied. The foundation is recoverable — this is an enforcement/verification debt, not a redesign.

### Recommended remediation sequence (blast-radius first)
1. **F-04** enable 2FA → 2. **F-01** fix branch protection (make required checks report; set satisfiable approver count) → 3. **F-06/F-07** protect `.github` + `jol-dr` (and correct F-08 state) → 4. **F-02** reconcile classification↔visibility (privatize the empty confidential shells first) → 5. **F-03** enable CodeQL + burn down jol-hub alerts → 6. **F-05** bind real teams → 7. **F-15/F-11/F-12/F-13/F-10/F-14/F-09** hygiene sweep → re-run this audit; flip to **GO** only when S0/S1 = 0.

---

**Constraint compliance:** all actions were read-only (`gh api`, `jq`, `git … remote/log`, `jq` on state). No writes, no `terraform apply`, no repo/org setting changes. No secrets were printed (token redacted by `gh`; only alert *counts* shown).

**Next step — choose one:**
1. **Persist** this audit as `docs/audits/2026-09-27-pre-deployment-audit.md` + `docs/drift-findings.md` (F-01…F-15) via a **signed-commit PR** (branch protection is PR-only) — I'll stage the files and open the PR.
2. **Start remediation** with the S0/S1 batch (I can prepare the Terraform/allow-list diffs for F-01/F-02/F-06/F-07 for your review — still no apply without your go).
3. **Save to memory**: the verified 4-source reconciliation and the F-01/F-02/F-03 S0-S1 findings, so this baseline is durable across sessions.

Want me to (1), (2), or (3)?