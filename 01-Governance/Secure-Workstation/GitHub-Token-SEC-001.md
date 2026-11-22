# GitHub Token — REMOVED (SEC-001)
**Action taken:** Token revoked at https://github.com/settings/tokens, removed from this file

---
## 🏗️ Infrastructure Pull Request

> **All infrastructure changes are subject to SOC 2 CC8.1 change management controls.**

---

### 📋 Change Summary

Adds the Gate-0 governance package for the proposed SOPS gates 4–8 rollout:
(1) an ADR-003 amendment DRAFT admitting SOPS+age for git-at-rest secret
encryption under a dual-identity tree-segregation model, and (2) an audited
fleet-sync script replacing a defective bulk-push draft. **Docs/tooling only —
zero production mutation; the amendment authorizes no implementation.**

### 🎯 Change Type

- [ ] Terraform (cloud resources)
- [ ] Kubernetes (cluster config)
- [ ] Helm (application deployment)
- [ ] Policy (OPA/Checkov/tfsec)
- [ ] CI/CD (GitHub Actions)
- [x] Other — governance docs (ADR amendment) + ops tooling script

### 🌍 Target Environment(s)

- [ ] dev
- [ ] staging
- [ ] prod

*(none — documentation and an unexecuted maintenance script)*

### ⚠️ Risk Assessment

**Risk Level:** Low

**Blast Radius:** None on merge. The ADR is Status: PROPOSED and explicitly
blocks Gates 4–8 until its Conditions are met; the script performs no action
unless explicitly invoked with `--stage`/`--push`, and was tested only in
report mode against the fleet plus a throwaway `/tmp` harness.

### 📝 Detailed Description

**1. `docs/architecture/adr-003-amendment-sops-age.md` (PROPOSED)**
- Admits SOPS (pinned) + age for **git-at-rest** encryption only;
  Vaultwarden + ESO remains the sole runtime secret store (ADR-003 intact).
- **Dual identities** `age-jol` / `age-jolm` — one per Tier-1 tree;
  cross-tree recipients defined as a segregation incident
  (AGENTS.md §0.2; ISO 27001 A.8.13; two PCI-DSS scopes, two DPIAs).
- Key source-of-truth = Vaultwarden (audited access); workstation copy is a
  derived cache. Shamir **M=2/N=3 is DR redundancy only** (resolves the
  Gate-6-destroys-key vs Gate-8-needs-key contradiction in the original
  proposal; SOPS has no splitting feature — `ssss` or equivalent required).
- Tooling pins recorded per fleet convention (`docs/dev-setup/tool-versions.md`).
- Alternatives considered and rejected: status quo (Ansible Vault only),
  single fleet identity, git-crypt, Sealed Secrets, Vault transit.

**2. `scripts/maintenance/git-fleet-sync.sh` (replaces `git-bulk-push.sh` draft)**
- D1: blind `git add -A` → staging opt-in + secret-path preflight with
  automatic staging revert on hit
- D2: `|| true` removed — pre-commit hook rejection = hard FAIL, never pushed
- D3: `set -e` fleet abort → per-repo remote verification + aggregated summary
- D4: production deploy remotes (`jol-mcp-servers` → `mcp-prod` post-receive
  deploy) on skip list + `prod|deploy|release` remote-pattern refusal
- Verified: `bash -n`; report mode against live 28-repo fleet (25 reported,
  3 skipped, 0 failures); controlled harness proved preflight BLOCK,
  push REFUSED, and exit=1 on failure.

**3. `CHANGELOG.md`** — one CC8.1 evidence row (docs env, rollback stated).

### 🔙 Rollback Plan

1. Revert this single commit (`git revert 29c938c`) — no other state depends on it.
2. Delete `docs/architecture/adr-003-amendment-sops-age.md` and
   `scripts/maintenance/git-fleet-sync.sh` (both are new files).
3. Nothing else to unwind: no production system, key material, or CI path is touched.

### 🎫 Ticket Reference

- **Jira/Linear:** `TBD` — CC8.1 issue **must be filed before merge** (Condition 2 below)
- **RFC/ADR:** amends `docs/adr/ADR-003-secrets-management.md`; house style per
  `docs/architecture/adr-004-amendment.md`; prior art: jol-hub session
  "Implement SOPS age toolkit" (2026-08-13)

### ✅ Pre-Merge Checklist

- [ ] ~~`terraform fmt` and `terraform validate` pass~~ — N/A (no Terraform)
- [ ] ~~`checkov` scan passes~~ — N/A (no cloud resources)
- [ ] ~~`tfsec` scan passes~~ — N/A
- [ ] ~~OPA policy tests pass~~ — N/A (no policy changes)
- [x] No secrets committed (TruffleHog clean) — draft contains no key material;
      script scan patterns only
- [x] Cost impact assessed (if applicable) — none
- [x] GDPR data impact assessed (if applicable) — see Condition 3 below;
      amendment strengthens Art. 5(1)(f)/Art. 32 posture, no new processing
- [x] Rollback plan documented
- [x] CHANGELOG.md updated (for production changes)

### 📊 Cost Impact

| Before | After | Delta |
|--------|-------|-------|
| $0 | $0 | $0 |

---

### 🔒 Gate Conditions — SOPS Gates 4–8 remain BLOCKED until ALL are met

These are the amendment's own Conditions, pre-listed for reviewer sign-off.
**Merging this PR does NOT approve the amendment** — approval is a separate
human decision recorded in its Change History table.

- [ ] **C1 — Security approval**: security team reviews this PR and records
      approval in the amendment's Change History (CODEOWNERS gate on `docs/adr/`)
- [ ] **C2 — CC8.1 issue filed**: GitHub Issue covering Gates 4–8 as one change
      series, with rollback plan; linked here before merge
- [ ] **C3 — DPIA re-check noted**: entry added to
      `docs/compliance/dpia-trigger-check.md` (trigger score already ≥3 YES ⇒
      existing pilot DPIA scope covers; confirm and record)
- [ ] **C4 — Custody plan decided BEFORE key generation**: two-identity share
      locations (HSM / physical safe / encrypted off-site) logged — custody
      cannot be retrofitted
- [ ] **C5 — Reconciliation with jol-hub prior SOPS work**: align with
      `jol-hub/scripts/sops-validate.py` + DECISION-LOG conventions

**Reviewer note:** the two mandatory design deviations from the original
gate proposal are (a) TWO identities instead of one, and (b) operational
identity persists in Vaultwarden with Shamir as DR-only. Any Gate 4–8
execution following the original single-identity/destroy-key prompts is
non-compliant with this amendment.

---

> 📌 **Production applies are manual only** — use the `Production Apply`
> workflow with proper approval.


---
---


**All five conditions are now CLOSED.** The full chain is merged on `main`:

| PR | Commit | Closed |
|---|---|---|
| #35 | `3dd9fda` | ADR amendment + C1 |
| #37 | `bd17f16` | Gate 4 tooling pin |
| #38 | `fc5056e` | C3 DPIA re-check + C4 plan |
| #39 | `a629aae` | C4 sign-off (your directive) |
| #40 | `a795b0f` | **C5 reconciliation** |

[Issue #36](https://github.com/journeyoflife-org/jol-infrastructure/issues/36#issuecomment-5430569597) now carries the final condition ledger.

## The C5 finding worth your attention
jol-hub's prior SOPS work turned out to be an **asset, not a conflict**: its validator already enforced exactly the amendment's rules (public-recipients-only `.sops.yaml`, `keys.txt` ban, `secrets/encrypted/` convention). But the **source `.py` is lost** — only the compiled `.pyc` survived the monorepo refactor, and it was never ratified in jol-hub's DECISION-LOG. I archived its 42 recovered constants as evidence and assigned the restore as open action **R2 in the jol-hub repo** (one task, not blocking anything). `secrets/encrypted/**` is now the ratified fleet convention (R3), and `ssss` 0.5-5 was already installed (D1 satisfied).

## Gate 5 — procedurally unblocked, awaiting your physical readiness

Per the approved custody plan, I will **not** generate key material until you confirm you have the custody materials physically ready:

1. **2 × encrypted USB sticks** (or one, labeled for both trees — shares of the two identities must NOT share a container, so two is cleaner)
2. **2 × sealable envelopes** for the off-site shares (bank safe deposit / second location per D3)
3. Your **Vaultwarden access** session (authoritative operational copies per D4)

When ready, say **"execute Gate 5"** and I'll run the exact custody-plan §5 procedure: temporary keygen → Shamir M=2/N=3 split per identity → rehearsal round-trip *before* destruction of the temp file → share files sealed at 600/700 with the custody log filled → Vaultwarden import steps handed to you. Every step logged for the #36 audit trail.

---
---
Everything is verified. Since `gh` has no auth, the last two actions (issue + PR) must be clicked through GitHub's web UI — here is the complete, paste-ready closing package.

## Verification record — all green

| Check | Result |
|---|---|
| Branch | `adr/003-amendment-sops-age` @ `29c938c`, exactly 1 commit ahead of `main` |
| Content | 3 files, +337 lines — ADR (133) + script (203, mode 755) + CHANGELOG +1 row |
| Mergeability | Fast-forwardable onto `main` (`b241c8a`) — no rebase needed |
| Remote | **Pushed** — `origin/adr/003-amendment-sops-age` confirmed tracking |
| Segregation | Your Qodana CHANGELOG row and the 38 dirty files on the other branch stayed out |

## Step 1 — File the CC8.1 issue (merge blocker, Condition C2)

Open: `https://github.com/journeyoflife-org/jol-infrastructure/issues/new?template=infra-change-request.yml`

| Field | Value |
|---|---|
| Title | `[infra-change]: SOPS Gates 4–8 change series — ADR-003 amendment (PROPOSED) and fleet-sync tooling` |
| Change Type | `other` |
| Target Environment(s) | *(none applicable — docs/tooling only; if the form forces one: `dev`)* |
| Risk Level | `Low` |
| Ticket Reference | ⚠ **JOL-XXXX — you must supply the real Jira/Linear ID**; it's a required field and I won't fabricate an audit-trail reference |

**Change Description** (paste):
```
**What:** Gate-0 governance package for the proposed SOPS rollout: ADR-003
amendment DRAFT (SOPS+age git-at-rest, dual identities age-jol/age-jolm,
Vaultwarden as key source-of-truth, Shamir M=2/N=3 DR-only) plus audited
fleet-sync script replacing a defective bulk-push draft.
**Why:** The original Gates 4–8 proposal violated ADR-003 (unapproved stack),
AGENTS.md §0.2 (single identity across both Tier-1 trees), and had a Gate 6/8
key-destruction contradiction. This change series fixes all three before any
execution.
**Expected:** Amendment approved or rejected by security review; zero mutation
either way. Gates 4–8 remain BLOCKED on Conditions C1–C5 in the amendment.
PR: <link after creation>
```


**Rollback Plan** (paste):
```
1. Revert single commit 29c938c (git revert) — no dependent state
2. Delete docs/architecture/adr-003-amendment-sops-age.md and
   scripts/maintenance/git-fleet-sync.sh (both new files)
3. No production system, key material, or CI path is touched at any stage
```


Compliance checklist: TruffleHog clean ✅ · Checkov/tfsec N/A ✅ · Rollback documented ✅ · Cost: none ✅ · GDPR: assessed (amendment strengthens Art. 5(1)(f)/32; DPIA re-check is Condition C3) ✅

## Step 2 — Open the PR

Open: `https://github.com/journeyoflife-org/jol-infrastructure/compare/main...adr/003-amendment-sops-age`

- Paste the PR body from my previous message (it already contains every template section plus the Conditions checklist).
- In **Ticket Reference**, replace `TBD` with the issue number from Step 1.
- Request reviewers per CODEOWNERS for `docs/adr/`-class paths.

## Remaining blockers (yours to clear)

1. **JOL-XXXX ticket ID** for the issue's required field.
2. **`gh auth login`** if you want me to do these via CLI next time — current state: not logged in (consistent with the 2026-08-24 CHANGELOG note).

Once the issue and PR exist, tell me both numbers and I'll cross-link them into the amendment's Change History evidence column and the CHANGELOG row's ticket field.




---
## 1. Bulk commit & push script — DO NOT run as-is

Verified against the actual fleet state just now:

| # | Defect | Evidence |
|---|---|---|
| 1 | **Blind `git add -A` across 363 dirty files** in 10 repos | `obsidian` 119, `jol-m-marketplace` 197, `jol-infrastructure` 36, `jol-llm` 32 dirty files — unreviewed bulk staging is how `.env` files and PII get committed |
| 2 | **`git commit … \|\| true` defeats your secret scanners** | The fleet mandate is pre-commit `git-secrets`/`trufflehog`/`bandit` (AGENTS.md §0.1); swallowing commit failures silences exactly the gate that protects Tier-1 trees |
| 3 | **`set -euo pipefail` + missing remotes aborts mid-fleet** | `jol-qoder-history` and `jol-m-qoder-history` have **no remote** — `git push origin …` fails and the script dies there, leaving later repos unsynced with no report |
| 4 | **`jol-mcp-servers` has remote `mcp-prod` = production deploy push** | Per its host doc, push to that bare repo triggers the post-receive deploy to `mcp-prod-lt01`. Your script would silently deploy to production with zero change control (SOC 2 CC8.1 violation, §0.3 snapshot requirement skipped) |

Also: `cd "$repo"` without returning, and no per-repo issue/rollback reference, which your own change-control standard requires for anything touching prod-adjacent repos.

**Minimum fixes**: skip-list (`jol-mcp-servers`, both history repos), remove `|| true`, add `--push-existing` semantics (push only if upstream exists), and a summary table at the end. The one-liner has all the same flaws plus silently skips `obsidian` (`-name "jol*"`) — don't use it either.

## 2. SOPS Gates 4–8 — blocked on policy; three design flaws

**Gate 0 (blocking):** [ADR-003](file:///opt/jol/repos/jol-infrastructure/docs/adr/ADR-003-secrets-management.md) is **Accepted**: Vaultwarden + External Secrets Operator is the ratified stack — Sealed Secrets was *explicitly rejected*, and AGENTS.md §1 fixes the secret flow as Ansible Vault / cloud-init / Vaultwarden. Implementing SOPS fleet-wide without an ADR-003 amendment (or a new ADR-007) contradicts your own governance and would fail ISO 27001 A.5.9 / SOC 2 CC8.1 review. A DPIA trigger-check (`docs/compliance/dpia-trigger-check.md`) is also owed, since a new cryptographic control touches GDPR Art. 9 processing.

**Design flaws inside the gates (independent of the ADR):**
1. **Gate 5 proposes ONE age identity for all 28 repos across both trees.** That fuses two PCI-DSS scopes and two DPIAs into a single cryptographic root of trust — the exact merger your segregation policy forbids. Correct design: **two identities** (church + marketplace), separate custody, separate `.sops.yaml` recipients.
2. **Gate 6 ↔ Gate 8 contradiction.** Gate 6 says *destroy the identity after splitting*; Gate 8 says *decrypt MUST use `/opt/jol/.sops/age-prod.txt`* — which no longer exists. Correct pattern: keep the operational identity hardened + backed up in Vaultwarden (the ADR-003 source of truth); Shamir shares are **DR redundancy only**, rehearsed quarterly.
3. **"Use sops … to Shamir-split" is not possible** — sops has no secret-sharing feature; that needs `ssss`/`shamir` tooling, and the share-custody log needs locations you don't yet have defined (HSM? safe? — these are decisions, not tasks).

Minor: the tooling manifest belongs in `docs/dev-setup/tool-versions.md` (your pinned-version convention), not `/opt/jol/docs/`; `age` 1.1.1 is already installed; `sops` is not, and no `.sops.yaml` exists anywhere yet. Note jol-hub already ran a 445-msg "Implement SOPS age toolkit" session on Aug 13 with a `scripts/sops-validate.py` artifact — reconcile with that before restarting the work.

## 3. Do you need the `.sops.yaml` template / runbook now?

**No — not yet.** Generating Gate 7 artifacts now would bake in the single-recipient design before the two blocking decisions are made (ADR amendment + one-vs-two identities). The correct sequence is:

1. Draft ADR-003 amendment (or new ADR) + DPIA trigger check — **I can produce this now** if you want.
2. Decide two-identity model (my recommendation: yes).
3. Then gates 4–8 proceed with corrected prompts, and I'll generate the per-tree `.sops.yaml` templates + consolidated runbook for the wiki as Gate 7 deliverables.

Want me to (a) write the ADR amendment draft, (b) fix the bulk script with the four defects addressed, or both?

---



## 1. ADR amendment draft — [adr-003-amendment-sops-age.md](file:///opt/jol/repos/jol-infrastructure/docs/architecture/adr-003-amendment-sops-age.md)

Written in the exact house style of the ADR-004 amendment (PROPOSED status, Conditions, Rollback ladder, Compliance mapping, Alternatives, Change History). Key positions a paranoid reviewer will look for:

- **Scope fence**: SOPS+age admitted for *git-at-rest* only; Vaultwarden+ESO remains the sole runtime store — the amendment cannot be read as authorizing plaintext values in git.
- **Dual identity** <font color="#f79646">(`age-jol` / `age-jolm`)</font>: segregation enforced *cryptographically*, with cross-tree recipients defined as an incident — ISO 27001 A.8.13 / PCI scope separation.
- **Key source-of-truth in Vaultwarden** (audited access); workstation copy is a derived cache. Fixes your Gate 5 "never leave the directory" rule, which would have made the workstation a single point of failure.
- **The Gate 6/8 contradiction resolved**: operational identity persists; Shamir M=2/N=3 is DR redundancy only, rehearsed quarterly — and sops's nonexistent splitting feature replaced with `ssss`.
- **Five Conditions gate** (approval PR, CC8.1 issue, DPIA re-check, custody plan *before* keygen, reconciliation with jol-hub's prior SOPS work) — no Gate 4 execution until all are met.

## 2. Fixed bulk script — [git-fleet-sync.sh](file:///opt/jol/repos/jol-infrastructure/scripts/maintenance/git-fleet-sync.sh)

All four defects fixed **and verified in a controlled harness** (`/tmp/fleetsync-test`, real fleet untouched):

| Defect | Fix | Verified evidence |
|---|---|---|
| D1 blind `add -A` | Staging opt-in + secret-path preflight (`.env`, `*.pem`, `*.key`, tokens…), staging **reverted** on hit | `r-env → FAIL, BLOCKED PATH, staging reverted` |
| D2 `\|\| true` | Commit rejection = hard FAIL, never pushed | flag removed; hooks run unmasked |
| D3 fleet abort | `set -uo pipefail` without `-e`; per-repo remote check; aggregation + summary | `r-noremote → PARTIAL, push skipped`; fleet continues |
| D4 prod-deploy push | SKIP list + `prod\|deploy\|release` remote-pattern refusal | `r-prod → push REFUSED (deploy channel)` |

Plus: non-zero exit on any failure (measured `exit=1`), interactive confirmation unless `--yes`, stdout-only logging for `tee` capture. Live report mode already ran clean against the real 28-repo fleet (25 reported, 3 correctly skipped, 0 failures).

## Change-control status

- CHANGELOG row added (CC8.1, rollback: delete both files). Zero production mutation.
- **Not committed** — deliberately: your workspace branch is `docs/db-pilot-tenant-isolation-delta` (different topic). Correct next step: file the CC8.1 issue, then a dedicated branch (e.g. `adr/003-amendment-sops-age`) and security-reviewed PR, since `docs/adr/`-class paths are CODEOWNERS-gated. Say the word and I'll prepare the branch.