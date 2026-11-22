

# Pre-Deployment Per-Repository Audit — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Produce an evidence-gated audit of all 30 Via-Vitae repositories across three planes (declared, live, local) and a binding go/no-go decision before any `terraform apply`.

**Architecture:** Read-only sweep scripts emit TSV evidence into `audit-evidence/`; findings are reconciled against `repos_data.tf`, `terraform state`, and the live GitHub API; results are emitted as JSON per `policies/AUDIT_PROMPT.md` §6 and folded back into `docs/COMPLIANCE.md`. No mutation of the org, no apply, no push to `[![](http://localhost:63342/markdownPreview/1492451121/commandRunner/run.png)](http://localhost:63342/markdownPreview/1492451121/markdown-preview-index-8hkmppp78rkagi87iceek81cm8.html#)main`.

**Tech Stack:** Terraform 1.16.1, integrations/github provider 6.13.0, `gh` CLI (GET verbs only), `jq`, Bash. Org: `Via-Vitae`, plan: **free**.

**Authority boundary:** Plan-only per `policies/AUDIT_PROMPT.md` §4 and the owner's standing instruction. `terraform apply`, `gh api` write verbs, and `git push` to `[![](http://localhost:63342/markdownPreview/1492451121/commandRunner/run.png)](http://localhost:63342/markdownPreview/1492451121/markdown-preview-index-8hkmppp78rkagi87iceek81cm8.html#)main` are **prohibited** until the owner explicitly authorises them.

---

## 0. Answer: how many repositories

**30.** Verified three ways, all agreeing, on 2026-09-27:

|Plane|Source|Count|Command|
|---|---|---|---|
|Declared|`repos_data.tf` `local.inventory`|30|`grep -cE '^[[:space:]]{4}"[^"]+"[[:space:]]*=' repos_data.tf`|
|Live|GitHub org `Via-Vitae`|30|`gh api orgs/Via-Vitae/repos --paginate -q '.[].name' \| wc -l`|
|Terraform output|`repo_count`|30|`terraform plan` output block|

`./scripts/detect_drift.sh` reports `Declared: 30 Live: 30`, `(none)` shadow, `(none)` ghost. **Inventory integrity is clean.**

Composition: **25 public / 5 private**, **21 with content / 9 empty**, **1 control plane + 29 managed targets**.

### 0.1 The third plane the drift script does not see

`/opt/viavitae/repos` holds **33 local directories** — all **30** org repos (including the _hidden_ `.github`, which a `*/` glob silently skips) plus **3** that are not in the org. Verified with `find /opt/viavitae/repos -maxdepth 1 -mindepth 1 -type d | wc -l` → `33`. Three reconciliations fail:

|Local dir|State|Why it matters|
|---|---|---|
|`jol-deploy/`|git, **NO_REMOTE**, **no commits** (unborn HEAD), 7 uncommitted files|Local-only repo with no remote and no history — the 7 files exist **nowhere else** and are one `rm -rf` from permanent loss. Belongs to the separate `jol-control` / `journeyoflife-org` scope: a project-separation boundary violation inside the Via-Vitae workspace.|
|`viavitae-diocese-landing/`|**not a git repo**|Uncontrolled code with no version control and no remote; name collides with the managed `viavitae-landing-diocese`. Provenance unknown.|
|`WelcomeScreen/`|**not a git repo**|Uncontrolled code, no version control, no remote.|

Also observed: **3** repos use SSH remotes (`.github`, `viavitae-docs`, `viavitae-template`) while 27 use HTTPS — two different credential paths; dirty trees in `viavitae-web` (14), `viavitae-policies` (2), `viavitae-control` (2, includes this plan), `viavitae-infra` (1); non-`[![](http://localhost:63342/markdownPreview/1492451121/commandRunner/run.png)](http://localhost:63342/markdownPreview/1492451121/markdown-preview-index-8hkmppp78rkagi87iceek81cm8.html#)main` branches checked out in `viavitae-control`, `viavitae-demos` (`fix/ci-blockers`), `viavitae-qa` (`fix/ci-and-codeql`); 9 empty repos are in the `UNBORN` HEAD state locally.

`.github` **is** cloned (`git@github.com:Via-Vitae/.github.git`, branch `[![](http://localhost:63342/markdownPreview/1492451121/commandRunner/run.png)](http://localhost:63342/markdownPreview/1492451121/markdown-preview-index-8hkmppp78rkagi87iceek81cm8.html#)main`, clean, single commit `162effd feat: bootstrap .github org profile…`), so the frozen-branch condition in VV-2026-09-008 is remediable from a local working copy.

**Conclusion:** `detect_drift.sh` reconciles 2 of 3 planes. The audit must cover the local plane too (Task 6).

---

## 1. Audit strata — all 30 repositories

Strata drive sampling depth. **No stratum may be sampled — every repo in a stratum is audited** (anti-pattern "sample-of-one generalization", `AUDIT_PROMPT.md` §8).

### Stratum A — control plane (1)

|Repo|Tier|Live vis|Size|Live protection|In TF state|
|---|---|---|---|---|---|
|`viavitae-control`|control|public|132|ruleset `protect-main:active`; classic BP **none**|**no**|

### Stratum B — private, empty, state-managed (5)

|Repo|Tier|Live vis|Size|Live protection|In TF state|
|---|---|---|---|---|---|
|`viavitae-compliance`|control|private|0|unreadable (HTTP 403, plan-tier)|yes|
|`viavitae-data-governance`|control|private|0|unreadable (HTTP 403)|yes|
|`viavitae-policies`|control|private|0|unreadable (HTTP 403)|yes|
|`viavitae-threat-model`|control|private|0|unreadable (HTTP 403)|yes|
|`viavitae-vendor-register`|control|private|0|unreadable (HTTP 403)|yes|

### Stratum C — org defaults, active public (1)

|Repo|Tier|Live vis|Size|Live protection|In TF state|
|---|---|---|---|---|---|
|`.github`|meta|public|40|classic BP: **1 approval, code_owner=true, ctx=`CI status`, enforce_admins=true**|**no**|

### Stratum D — active public with live classic branch protection (19)

Live values captured 2026-09-27 from `gh api repos/Via-Vitae/<repo>/branches/main/protection`. **Declared values for all 20 (Stratum C + D) are `approvals=0`, `code_owner=false`, `contexts=[]`.**

|Repo|Tier|Size|Live approvals|Live code-owner|Live required contexts|
|---|---|---|---|---|---|
|`viavitae-web`|app|1112|**1**|–|Analyze (javascript-typescript) + Action SHA pinning + Governance files + CI status + Compliance status|
|`viavitae-qa`|platform|466|**0**|–|Compliance status + CI status|
|`viavitae-infra`|platform|560|**1**|true|CI status + Compliance status|
|`viavitae-template`|platform|85|**1**|true|CI status + Compliance status|
|`viavitae-api`|app|417|**1**|true|CI status + Compliance status|
|`viavitae-brand`|app|360|**1**|true|CI status + Compliance status|
|`viavitae-clients`|app|102|**1**|true|CI status + Compliance status|
|`viavitae-demos`|app|236|**1**|true|compliance-status + action-pinning + governance + analyze (javascript-typescript) + CI status|
|`viavitae-docs`|control|199|**1**|true|CI status + Compliance status|
|`viavitae-landing-basilica`|site|81|**1**|–|CI status + Compliance status|
|`viavitae-landing-cathedral`|site|81|**1**|–|CI status + Compliance status|
|`viavitae-landing-cemetery-services`|site|81|**1**|–|CI status + Compliance status|
|`viavitae-landing-churches-orthodox`|site|81|**1**|–|CI status + Compliance status|
|`viavitae-landing-churches-other`|site|81|**1**|–|CI status + Compliance status|
|`viavitae-landing-churches-protestant`|site|81|**1**|–|CI status + Compliance status|
|`viavitae-landing-deaneries`|site|81|**1**|–|CI status + Compliance status|
|`viavitae-landing-diocese`|site|81|**1**|–|CI status + Compliance status|
|`viavitae-landing-funeral-services`|site|81|**1**|–|CI status + Compliance status|
|`viavitae-landing-parish-church`|site|81|**1**|–|CI status + Compliance status|

Aggregate: **19 of 20** protected repos require **1 approval** live; **8** require **code-owner review** live; **all 20** require at least one status check live. `enforce_admins=true` and `required_linear_history=true` on all sampled repos. **None** of the 20 protection rules is in Terraform state.

Secondary observation for D10: `viavitae-demos` uses **lowercase-hyphenated** check names (`compliance-status`, `action-pinning`, `governance`) while every other repo uses **title-case with spaces** (`Compliance status`). A required context whose name no workflow actually emits blocks every PR forever. Task 5 must confirm each declared context is produced by a workflow in that same repo.

### Stratum E — empty public with live unmanaged ruleset (4)

|Repo|Tier|Live ruleset|Classic BP|In TF state|
|---|---|---|---|---|
|`viavitae-observability`|platform|`protect-main:active`|none|**no**|
|`viavitae-reusable-workflows`|platform|`protect-main:active`|none|**no**|
|`viavitae-runbooks`|platform|`protect-main:active`|none|**no**|
|`viavitae-training`|control|`protect-main:active`|none|**no**|

`repo_rulesets.tf` declares `protect-main` for **`viavitae-control` only**. These 4 live rulesets are undeclared drift.

---

## 2. Audit dimensions

Every repo is scored on all 12 dimensions. Each dimension has one command, one pass criterion, one evidence column.

|ID|Dimension|Command (per repo `R`)|Pass criterion|
|---|---|---|---|
|D1|Inventory membership|`gh api orgs/Via-Vitae/repos --paginate -q '.[].name'` + `grep '"R"' repos_data.tf`|live ⇔ declared, no shadow/ghost|
|D2|Visibility|`gh api repos/Via-Vitae/R -q .visibility`|equals `repos_data.tf` value|
|D3|Emptiness / default branch|`gh api repos/Via-Vitae/R -q .size`|`size>0` ⇒ `default_branch_exists=true` declared; `size==0` ⇒ deferred, documented|
|D4|Protection mechanism|`gh api repos/Via-Vitae/R/rulesets`; `gh api repos/Via-Vitae/R/branches/main/protection`|exactly one mechanism, declared in config, present in state|
|D5|Protection strength|fields from D4 response|approvals, code-owner, dismiss-stale, linear history, contexts, `enforce_admins`, deletions/force-push **≥** live baseline|
|D6|Secret scanning + push protection|`gh api repos/Via-Vitae/R -q '.security_and_analysis'`|`enabled` on all public; private ⇒ documented `gap-plan-tier` + gitleaks compensating control|
|D7|Dependabot|`gh api repos/Via-Vitae/R/dependabot/alerts?state=open`|`0` open `critical`/`high`, or a dated triage decision|
|D8|Least-privilege features|`has_wiki`, `has_projects`, `delete_branch_on_merge`|matches module defaults (`C10`)|
|D9|Governance files|`gh api repos/Via-Vitae/R/git/trees/main?recursive=1`|`CODEOWNERS` present **and every owner resolves**; `README`, `LICENSE`, `SECURITY.md` present|
|D10|CI/CD supply chain|harvest `uses:` from all workflows|every action SHA-pinned and permitted by the effective allow-list|
|D11|Access|`gh api repos/Via-Vitae/R/collaborators?affiliation=all`|named humans/teams only; a valid approver exists; no unexpected outside collaborator|
|D12|IaC coverage|`terraform state list \| grep 'module.repo\["R"\]'`|repo **and** its protection/dependabot/file resources in state|

---

## 3. Confirmed findings (evidence already captured)

These are **verified**, not suspected. IDs follow `AUDIT_PROMPT.md` §6. Severities follow the §7 rubric. Task 8 emits them as `audit-evidence/findings.json`.

|ID|Sev|Ctrl|Finding|Evidence|
|---|---|---|---|---|
|VV-2026-09-001|**P0**|C1|`terraform plan` **fails** — token lacks `user:email`/`read:user` required by `data.github_organization` (`org_data.tf:10`). Deployment is impossible today.|`terraform plan … ; EXIT=1`; error at plan line 2363|
|VV-2026-09-002|**P0**|C6|Pending plan creates **20** `github_branch_protection` resources over 20 repos that **already have stronger live protection**, declaring `required_approving_review_count = 0`, `require_code_owner_reviews = false` and `required_status_checks { strict = true }` with **no contexts**. Quantified loss: **19/20** repos lose their 1 required approval, **8/20** lose code-owner review, **20/20** lose every required status check (CodeQL, Action SHA pinning, Governance files, CI, Compliance). Live rules are unmanaged ⇒ **no state-based rollback**.|plan lines 2206–2229; live baseline in §1 Stratum C/D|
|VV-2026-09-003|**P0**|C1|Terraform state holds **5** resources total (5 × `github_repository` for the private repos). **0** branch protections, **0** rulesets, **0** dependabot, **0** org resources. The system of record is not reproducible from committed code.|`terraform state list \| wc -l` → `5`|
|VV-2026-09-004|P1|C2|**5** live `protect-main` rulesets; only `viavitae-control` is declared/imported (`repo_rulesets.tf:24,61`). 4 are undeclared drift; the 5 private repos return HTTP 403 so their ruleset state is **unknown**.|`gh api repos/Via-Vitae/{observability,reusable-workflows,runbooks,training}/rulesets`|
|VV-2026-09-005|P1|C2|`detect_drift.sh` compares **repo names only**. It cannot see protection, ruleset, settings, or file drift — CI reports `OK: no drift` while VV-2026-09-002/003/004 exist.|`scripts/detect_drift.sh:23-31`|
|VV-2026-09-006|P1|C9|Org Actions policy live: `allowed_actions="all"`, `sha_pinning_required=false`. Declared config switches to `selected` with 8 patterns that do **not** cover observed third-party actions (`aquasecurity/trivy-action`, `pnpm/action-setup`, `trufflesecurity/trufflehog`, `docker/*`, `hashicorp/setup-terraform`, `astral-sh/setup-uv`) nor internal `Via-Vitae/.github` reusable workflows.|`gh api orgs/Via-Vitae/actions/permissions`; harvested 39 distinct `uses:` refs from 75 workflow files|
|VV-2026-09-007|P1|C5|**30 open Dependabot alerts** on public repos: `viavitae-web` 19 (7 high), `viavitae-qa` 8 (3 high), `viavitae-docs` 3 (**1 critical**). C5 is "enabled" but remediation is not occurring.|`gh api repos/Via-Vitae/R/dependabot/alerts?state=open`|
|VV-2026-09-008|P1|C6/C7|`.github` is **frozen**: `enforce_admins=true` + 1 required approval + `require_code_owner_reviews=true`, sole collaborator `JourneyOfLife`, `IterVitae` **not** a collaborator, self-approval forbidden ⇒ no valid approver exists. The repo supplying org defaults and reusable workflows cannot be updated via `[![](http://localhost:63342/markdownPreview/1492451121/commandRunner/run.png)](http://localhost:63342/markdownPreview/1492451121/markdown-preview-index-8hkmppp78rkagi87iceek81cm8.html#)main`.|`gh api repos/Via-Vitae/.github/branches/main/protection`; `…/collaborators` → `JourneyOfLife` only|
|VV-2026-09-009|P1|C7|CODEOWNERS model conflict. Live `viavitae-web`/`viavitae-qa` use `@JourneyOfLife @IterVitae` and explicitly document that team owners were **removed because they do not resolve**; `.github` still uses `@Via-Vitae/architects`; `locals.tf:4-10` would write **team-based** CODEOWNERS. Flipping `manage_files=true` (as `docs/COMPLIANCE.md` P1-6 recommends) would overwrite working person-based ownership with team-based ownership.|live CODEOWNERS contents; `locals.tf:4-10`; `modules/repo/files.tf`|
|VV-2026-09-010|P1|C7|`viavitae-web` declares ~30 sensitive CODEOWNERS path rules (payments, donations, DPIA, legal, contracts) but live `require_code_owner_reviews=false` ⇒ path-specific ownership is **advisory only**, enforced by nothing.|`gh api repos/Via-Vitae/viavitae-web/branches/main/protection -q '.required_pull_request_reviews.require_code_owner_reviews'` → `false`|
|VV-2026-09-011|P1|C12/C11|Org: `two_factor_requirement_enabled=false`; `members_can_delete_repositories=true`; `members_can_change_repo_visibility=true`; `members_can_create_repositories=true`; `secret_scanning_enabled_for_new_repositories=false`. P1-1 and P1-5 remain open; new repos get **no** secret scanning by default.|`gh api orgs/Via-Vitae`; `gh api orgs/Via-Vitae/actions/permissions`|
|VV-2026-09-012|P1|C1|**Personal data in committed source.** `variables.tf:14` hardcodes a personal Gmail address as the `billing_email` **default**, and it is echoed into committed evidence at `audit-evidence/terraform-plan.txt:95`. Breaches `AUDIT_PROMPT.md` §2 and GDPR Art. 5(1)(c) minimisation. Value is referenced by location only, never reproduced here.|`grep -n billing_email -A4 variables.tf`|
|VV-2026-09-013|P1|—|**Committed evidence is a failed plan.** `audit-evidence/terraform-plan.txt` ends `Error: Your token has not been granted the required scopes…` yet is committed as audit evidence, and `audit-evidence/pre-apply-plan.txt` is **untracked**. Evidence integrity gate (`AUDIT_PROMPT.md` Gate 1) would fail.|`sed -n '2363p' audit-evidence/terraform-plan.txt`; `git status --porcelain`|
|VV-2026-09-014|P2|C10|Least-privilege features unapplied: `has_wiki=true` on 25 repos, `has_projects=true` on 22, `delete_branch_on_merge=false` on 25. Only the 5 state-managed repos carry module settings — direct corroboration of VV-2026-09-003.|`gh api repos/Via-Vitae/R -q '{has_wiki,has_projects,delete_branch_on_merge}'`|
|VV-2026-09-015|P2|C1|Stale auditor-facing docs. `docs/COMPLIANCE.md:54` "All 30 repos are PUBLIC" (live: 5 private); `:73` "All 30 repos are EMPTY" (live: 9 empty, 21 active); `repos_data.tf:11` "all 30 are public today" contradicts 5 `visibility = "private"` entries in the same file.|`gh api orgs/Via-Vitae/repos --paginate -q '[.[]\|.visibility]\|group_by(.)\|map({(.[0]):length})\|add'`|
|VV-2026-09-016|P2|C8|Pin/standardisation drift: identical actions pinned at **different** SHAs across repos (`actions/checkout` ×2, `github/codeql-action` ×4, `trufflesecurity/trufflehog` ×4, `aquasecurity/trivy-action` ×2, `docker/*` ×2, `pnpm/action-setup` ×2). The org reusable workflows in `.github` have **zero consumers**; the dedicated `viavitae-reusable-workflows` repo is **empty**. Internal reusable refs use mutable `@main`.|`vv_uses.txt` harvest (39 refs / 75 files)|
|VV-2026-09-017|P2|C2|Local-plane shadow artifacts: `viavitae-diocese-landing/` and `WelcomeScreen/` are **not git repos**; `jol-deploy/` has **no remote and no commits** with 7 uncommitted files — unbacked work from the separate `jol-control` scope sitting inside the Via-Vitae workspace. Dirty trees in `viavitae-web` (14) and `viavitae-policies` (2). All 30 org repos _do_ have local clones, including the hidden `.github`.|`find /opt/viavitae/repos -maxdepth 1 -mindepth 1 -type d \| wc -l` → `33` (Task 6)|

### 3.1 Worked finding record (schema template for Task 8)

![](http://localhost:63342/markdownPreview/1336096853//opt/viavitae/repos/viavitae-control/docs/superpowers/plans)

`{   "id": "VV-2026-09-002",   "title": "Pending apply would downgrade live branch protection on 20 repositories",   "severity": "P0",   "control_id": "C6",   "framework_refs": ["SOC2:CC8.1", "ISO27001:A.14.2", "GDPR:Art.32"],   "declared": {     "source": "repos.tf:31-35 + variables.tf defaults",     "claim": "required_approving_review_count=0, required_status_checks{strict=true} with no contexts"   },   "actual": {     "source": "gh api repos/Via-Vitae/viavitae-web/branches/main/protection",     "observed": "required_approving_review_count=1, contexts=[Analyze (javascript-typescript), Action SHA pinning, Governance files, CI status, Compliance status]"   },   "evidence": [     "terraform plan -out=/tmp/vv_probe.tfplan -> 'Plan: 26 to import, 52 to add, 25 to change, 0 to destroy'; line 2206 'module.repo[\"viavitae-web\"].github_branch_protection.this[0] will be created'",     "gh api repos/Via-Vitae/viavitae-web/branches/main/protection -> {\"req_reviews\":1,\"code_owner\":false,\"contexts\":[5 items],\"enforce_admins\":true}"   ],   "impact": "Peer review and all five CI/CodeQL/governance gates are silently removed from the public-facing production website. The overwritten live rules are not in state, so Terraform cannot roll them back.",   "remediation": "Capture the live protection of all 20 repos as the declared baseline in repos_data.tf/variables before any apply; add github_branch_protection import blocks; re-plan and require 0 downgrades.",   "verification_after_fix": "terraform plan shows 'update in-place' (never 'create') for all 20 protections, with required_approving_review_count>=1 and all live contexts preserved.",   "owner": "terraform",   "status": "open" }`

---

## 4. File structure

|Action|File|Responsibility|
|---|---|---|
|Create|`scripts/audit_repo_sweep.sh`|Read-only D1–D12 sweep; emits one TSV row per repo|
|Create|`scripts/audit_actions_inventory.sh`|Harvests every `uses:` ref from every workflow; emits distinct action list|
|Create|`scripts/audit_local_plane.sh`|Reconciles `/opt/viavitae/repos` clones against the declared inventory|
|Modify|`scripts/detect_drift.sh`|Extend beyond names: add protection/ruleset drift (VV-2026-09-005)|
|Create|`audit-evidence/gate-3-per-repo-sweep.tsv`|Per-repo D1–D12 evidence|
|Create|`audit-evidence/actions-inventory.tsv`|Observed action refs + allow-list verdict|
|Create|`audit-evidence/local-plane.tsv`|Local clone reconciliation|
|Create|`audit-evidence/protection-reconciliation.md`|Live vs declared vs state, per repo, with downgrade flags|
|Create|`audit-evidence/findings.json`|All findings, `AUDIT_PROMPT.md` §6 schema|
|Modify|`docs/COMPLIANCE.md`|Correct VV-2026-09-015 staleness; reclassify control statuses from live evidence|
|Modify|`repos_data.tf`|Correct stale header claim; add captured live protection baseline|

---

## 5. Tasks

### Task 1: Unblock `terraform plan` (VV-2026-09-001)

**Files:**

- Read: `org_data.tf:10`, `variables.tf:11-15`
    
- Create: `audit-evidence/gate-0-preconditions.txt`
    
- **Step 1: Reproduce the failure and capture it**
    

![](http://localhost:63342/markdownPreview/1336096853//opt/viavitae/repos/viavitae-control/docs/superpowers/plans)

`[![](http://localhost:63342/markdownPreview/1492451121/commandRunner/runrun.png)](http://localhost:63342/markdownPreview/1492451121/markdown-preview-index-8hkmppp78rkagi87iceek81cm8.html#)cd /opt/viavitae/repos/viavitae-control { gh auth status; terraform version; cat .terraform-version; } > audit-evidence/gate-0-preconditions.txt 2>&1 GITHUB_TOKEN=$(gh auth token) terraform plan -input=false -no-color -out=/tmp/gate0.tfplan \  >> audit-evidence/gate-0-preconditions.txt 2>&1 echo "EXIT=$?" >> audit-evidence/gate-0-preconditions.txt`

Expected: `EXIT=1` and `Error: Your token has not been granted the required scopes … The 'email' field requires one of the following scopes: ['user:email', 'read:user']`.

- [ ] **Step 2: Confirm the root cause is the data source, not the provider**

![](http://localhost:63342/markdownPreview/1336096853//opt/viavitae/repos/viavitae-control/docs/superpowers/plans)

`[![](http://localhost:63342/markdownPreview/1492451121/commandRunner/runrun.png)](http://localhost:63342/markdownPreview/1492451121/markdown-preview-index-8hkmppp78rkagi87iceek81cm8.html#)grep -n 'data "github_organization"' -A 3 org_data.tf >> audit-evidence/gate-0-preconditions.txt`

Expected: `org_data.tf:10 data "github_organization" "this" { name = var.github_owner }`. The provider's `github_organization` data source queries `email`, which needs `user:email`/`read:user`.

- [ ] **Step 3: Record the decision — this is a human action, stop and ask**

Append to `audit-evidence/gate-0-preconditions.txt` the chosen option and its rationale:

- **Option 1 (recommended):** grant the token `read:user`. Least code change; keeps the 2FA `check` block in `org_data.tf:17` working.
- **Option 2:** delete `data.github_organization` + the `check` block and source `two_factor_requirement_enabled` from a `null_data_source`/`external` calling `gh api orgs/Via-Vitae`. Keeps token scopes minimal; loses the plan-time 2FA warning unless reimplemented.

Do **not** pick silently — the owner must choose, because Option 1 widens credential scope and Option 2 removes a detective control.

- [ ] **Step 4: Verify the plan completes**

![](http://localhost:63342/markdownPreview/1336096853//opt/viavitae/repos/viavitae-control/docs/superpowers/plans)

`[![](http://localhost:63342/markdownPreview/1492451121/commandRunner/runrun.png)](http://localhost:63342/markdownPreview/1492451121/markdown-preview-index-8hkmppp78rkagi87iceek81cm8.html#)GITHUB_TOKEN=$(gh auth token) terraform plan -input=false -no-color -out=/tmp/gate0.tfplan 2>&1 | tail -3`

Expected: `Plan: NN to import, NN to add, NN to change, 0 to destroy.` and **no** `Error:`. `0 to destroy` is mandatory — anything else is a **STOP**.

- [ ] **Step 5: Commit**

![](http://localhost:63342/markdownPreview/1336096853//opt/viavitae/repos/viavitae-control/docs/superpowers/plans)

`[![](http://localhost:63342/markdownPreview/1492451121/commandRunner/runrun.png)](http://localhost:63342/markdownPreview/1492451121/markdown-preview-index-8hkmppp78rkagi87iceek81cm8.html#)git add audit-evidence/gate-0-preconditions.txt git commit -m "audit: capture gate-0 precondition evidence and plan-blocker root cause"`

---

### Task 2: Build the per-repo sweep harness

**Files:**

- Create: `scripts/audit_repo_sweep.sh`
    
- **Step 1: Write the script**
    

![](http://localhost:63342/markdownPreview/1336096853//opt/viavitae/repos/viavitae-control/docs/superpowers/plans)

``[![](http://localhost:63342/markdownPreview/1492451121/commandRunner/runrun.png)](http://localhost:63342/markdownPreview/1492451121/markdown-preview-index-8hkmppp78rkagi87iceek81cm8.html#)#!/usr/bin/env bash # ############################################################################# # audit_repo_sweep.sh — read-only per-repo evidence sweep (dimensions D1-D12). # # Emits one TSV row per repository for every repo in the org. Uses only # `gh api` GET verbs plus `terraform state list`, so it is safe under the # plan-only authority boundary (policies/AUDIT_PROMPT.md §4). # # Usage: ORG=Via-Vitae ./scripts/audit_repo_sweep.sh # ############################################################################# set -euo pipefail ORG="${ORG:-Via-Vitae}" ROOT="$(cd "$(dirname "${BASH_SOURCE[0]}")/.." && pwd)" OUT="${OUT:-${ROOT}/audit-evidence/gate-3-per-repo-sweep.tsv}" mkdir -p "$(dirname "${OUT}")" command -v gh >/dev/null 2>&1 || { echo "gh CLI required" >&2; exit 1; } command -v jq >/dev/null 2>&1 || { echo "jq required" >&2; exit 1; } printf 'repo\tvis\tsize\tss\tpp\tdb\talerts\tcrit\thigh\twiki\tproj\tdbm\trulesets\tbp\tappr\tcoowner\tctxs\tadmins\tin_state\tcodeowners\tprecommit\tgitleaks\twf\tcollabs\n' > "${OUT}" mapfile -t repos < <(gh api "orgs/${ORG}/repos?per_page=100&type=all" --paginate --jq '.[].name' | sort) state="$(cd "${ROOT}" && terraform state list 2>/dev/null || true)" for n in "${repos[@]}"; do   f="${ORG}/${n}"   # gh prints HTTP error BODIES to stdout, so `cmd || echo X` concatenates the   # error JSON onto the value and silently corrupts the evidence column.   # Always test the exit status and OVERWRITE on failure.   if rj="$(gh api "repos/${f}" 2>/dev/null)"; then :; else rj='{}'; fi   vis=$(jq -r '.visibility' <<<"${rj}"); size=$(jq -r '.size' <<<"${rj}")   ss=$(jq -r '((.security_and_analysis//{}).secret_scanning.status)//"-"' <<<"${rj}")   pp=$(jq -r '((.security_and_analysis//{}).secret_scanning_push_protection.status)//"-"' <<<"${rj}")   db=$(jq -r '((.security_and_analysis//{}).dependabot_security_updates.status)//"-"' <<<"${rj}")   wiki=$(jq -r '.has_wiki' <<<"${rj}"); proj=$(jq -r '.has_projects' <<<"${rj}")   dbm=$(jq -r '.delete_branch_on_merge' <<<"${rj}")   if al="$(gh api "repos/${f}/dependabot/alerts?state=open&per_page=100" \           --jq '[.[].security_advisory.severity]' 2>/dev/null)"; then :; else al='[]'; fi   alerts=$(jq 'length' <<<"${al}")   crit=$(jq '[.[]|select(.=="critical")]|length' <<<"${al}")   high=$(jq '[.[]|select(.=="high")]|length' <<<"${al}")   if rs="$(gh api "repos/${f}/rulesets" --jq 'map(.name+":"+.enforcement)|join(",")' 2>/dev/null)"; then     [[ -n "${rs}" ]] || rs=NONE  else     rs=UNREADABLE   # private repo on the Free plan: HTTP 403 (plan-tier limit)   fi   if bpj="$(gh api "repos/${f}/branches/main/protection" 2>/dev/null)"; then     bp=YES    appr=$(jq -r '.required_pull_request_reviews.required_approving_review_count // "-"' <<<"${bpj}")     coo=$(jq -r '.required_pull_request_reviews.require_code_owner_reviews // "-"' <<<"${bpj}")     ctx=$(jq -r '[(.required_status_checks.contexts//[])[]]|join("+")|if .=="" then "-" else . end' <<<"${bpj}")     adm=$(jq -r '.enforce_admins.enabled' <<<"${bpj}")   else     bp=NO; appr=NA; coo=NA; ctx=NA; adm=NA  fi   if grep -q "module.repo\[\"${n}\"\]" <<<"${state}"; then ist=Y; else ist=N; fi   if t="$(gh api "repos/${f}/git/trees/main?recursive=1" --jq '[.tree[].path]|join(" ")' 2>/dev/null)"; then :; else t=''; fi   cow=$(grep -c CODEOWNERS <<<"${t}" || true)   pc=$(grep -c 'pre-commit-config' <<<"${t}" || true)   gl=$(grep -c gitleaks <<<"${t}" || true)   # `|| true` inside the group: grep exits 1 on no match, and pipefail would abort.   wf=$( { grep -oE '\.github/workflows/[^ ]+' <<<"${t}" || true; } | wc -l | tr -d ' ')   if cl="$(gh api "repos/${f}/collaborators?affiliation=all" --jq 'map(.login)|join(",")' 2>/dev/null)"; then :; else cl=NA; fi   printf '%s\t%s\t%s\t%s\t%s\t%s\t%s\t%s\t%s\t%s\t%s\t%s\t%s\t%s\t%s\t%s\t%s\t%s\t%s\t%s\t%s\t%s\t%s\t%s\n' \    "${n}" "${vis}" "${size}" "${ss}" "${pp}" "${db}" "${alerts}" "${crit}" "${high}" \    "${wiki}" "${proj}" "${dbm}" "${rs}" "${bp}" "${appr}" "${coo}" "${ctx}" "${adm}" \    "${ist}" "${cow}" "${pc}" "${gl}" "${wf}" "${cl}" >> "${OUT}" done echo "wrote $(( $(wc -l < "${OUT}") - 1 )) repo rows to ${OUT}"``

- [ ] **Step 2: Make it executable and lint it**

![](http://localhost:63342/markdownPreview/1336096853//opt/viavitae/repos/viavitae-control/docs/superpowers/plans)

`[![](http://localhost:63342/markdownPreview/1492451121/commandRunner/runrun.png)](http://localhost:63342/markdownPreview/1492451121/markdown-preview-index-8hkmppp78rkagi87iceek81cm8.html#)chmod +x scripts/audit_repo_sweep.sh bash -n scripts/audit_repo_sweep.sh && echo "SYNTAX OK" shellcheck scripts/audit_repo_sweep.sh || true`

Expected: `SYNTAX OK`. `shellcheck` warnings are acceptable only if annotated with a `# shellcheck disable=` and a reason.

- [ ] **Step 3: Commit**

![](http://localhost:63342/markdownPreview/1336096853//opt/viavitae/repos/viavitae-control/docs/superpowers/plans)

`[![](http://localhost:63342/markdownPreview/1492451121/commandRunner/runrun.png)](http://localhost:63342/markdownPreview/1492451121/markdown-preview-index-8hkmppp78rkagi87iceek81cm8.html#)git add scripts/audit_repo_sweep.sh git commit -m "feat(audit): add read-only per-repo D1-D12 evidence sweep"`

---

### Task 3: Run the sweep for all 30 repositories

**Files:**

- Create: `audit-evidence/gate-3-per-repo-sweep.tsv`
    
- **Step 1: Execute**
    

![](http://localhost:63342/markdownPreview/1336096853//opt/viavitae/repos/viavitae-control/docs/superpowers/plans)

`[![](http://localhost:63342/markdownPreview/1492451121/commandRunner/runrun.png)](http://localhost:63342/markdownPreview/1492451121/markdown-preview-index-8hkmppp78rkagi87iceek81cm8.html#)cd /opt/viavitae/repos/viavitae-control ORG=Via-Vitae ./scripts/audit_repo_sweep.sh`

Expected: `wrote 30 repo rows to …/audit-evidence/gate-3-per-repo-sweep.tsv`.

- [ ] **Step 2: Assert completeness — no repo missing, no dimension blank**

![](http://localhost:63342/markdownPreview/1336096853//opt/viavitae/repos/viavitae-control/docs/superpowers/plans)

`[![](http://localhost:63342/markdownPreview/1492451121/commandRunner/runrun.png)](http://localhost:63342/markdownPreview/1492451121/markdown-preview-index-8hkmppp78rkagi87iceek81cm8.html#)awk -F'\t' 'NR>1{if(NF!=24) print "BAD_WIDTH:"$1; for(i=1;i<=NF;i++) if($i=="") print "EMPTY:"$1":col"i}END{print "rows="NR-1}' \  audit-evidence/gate-3-per-repo-sweep.tsv`

Expected: `rows=30` and **no** `BAD_WIDTH` / `EMPTY` lines. Any output ⇒ fix the script, do not proceed.

- [ ] **Step 3: Assert the strata counts match §1**

![](http://localhost:63342/markdownPreview/1336096853//opt/viavitae/repos/viavitae-control/docs/superpowers/plans)

`[![](http://localhost:63342/markdownPreview/1492451121/commandRunner/runrun.png)](http://localhost:63342/markdownPreview/1492451121/markdown-preview-index-8hkmppp78rkagi87iceek81cm8.html#)awk -F'\t' 'NR>1{v[$2]++; if($3=="0")e++; if($14=="YES")b++; if($13=="NONE")rn++; if($13=="UNREADABLE")ru++; if($19=="Y")s++} END{printf "public=%d private=%d empty=%d classic_bp=%d rulesets=%d unreadable=%d in_state=%d\n",v["public"],v["private"],e,b,rn==""?0:30-rn-ru,ru,s}' \  audit-evidence/gate-3-per-repo-sweep.tsv`

Expected (verified against the live org on 2026-09-27):

![](http://localhost:63342/markdownPreview/1336096853//opt/viavitae/repos/viavitae-control/docs/superpowers/plans)

`public=25 private=5 empty=9 classic_bp=20 rulesets=5 unreadable=5 in_state=5`

Any deviation is a **new finding** — record it, do not silently accept. The `unreadable=5` value is itself evidence for the plan-tier blind spot: the rulesets on the 5 private repos **cannot be read on GitHub Free**, so D4 for Stratum B is _unknown_, never _pass_.

- [ ] **Step 4: Commit**

![](http://localhost:63342/markdownPreview/1336096853//opt/viavitae/repos/viavitae-control/docs/superpowers/plans)

`[![](http://localhost:63342/markdownPreview/1492451121/commandRunner/runrun.png)](http://localhost:63342/markdownPreview/1492451121/markdown-preview-index-8hkmppp78rkagi87iceek81cm8.html#)git add audit-evidence/gate-3-per-repo-sweep.tsv git commit -m "audit: capture per-repo D1-D12 sweep evidence for all 30 repos"`

---

### Task 4: Protection reconciliation — the downgrade report (VV-2026-09-002)

This is the **deployment gate**. It answers: does applying weaken anything live?

**Files:**

- Create: `audit-evidence/protection-reconciliation.md`
    
- **Step 1: Capture live protection for every protected repo**
    

![](http://localhost:63342/markdownPreview/1336096853//opt/viavitae/repos/viavitae-control/docs/superpowers/plans)

`[![](http://localhost:63342/markdownPreview/1492451121/commandRunner/runrun.png)](http://localhost:63342/markdownPreview/1492451121/markdown-preview-index-8hkmppp78rkagi87iceek81cm8.html#)cd /opt/viavitae/repos/viavitae-control : > /tmp/live_prot.jsonl awk -F'\t' 'NR>1{print $1}' audit-evidence/gate-3-per-repo-sweep.tsv | while read -r n; do   # gh api has no --arg; pipe into local jq so the repo name can be injected.   gh api "repos/Via-Vitae/$n/branches/main/protection" 2>/dev/null \     | jq -c --arg r "$n" '{repo:$r,appr:.required_pull_request_reviews.required_approving_review_count,co:.required_pull_request_reviews.require_code_owner_reviews,ctx:(.required_status_checks.contexts//[]),strict:.required_status_checks.strict,admins:.enforce_admins.enabled,lin:.required_linear_history.enabled,del:(.allow_deletions.enabled//false),force:(.allow_force_pushes.enabled//false)}' \    >> /tmp/live_prot.jsonl || true done wc -l < /tmp/live_prot.jsonl`

Expected: `20` (the 20 repos with live classic protection).

- [ ] **Step 2: Capture the declared (planned) protection**

![](http://localhost:63342/markdownPreview/1336096853//opt/viavitae/repos/viavitae-control/docs/superpowers/plans)

`[![](http://localhost:63342/markdownPreview/1492451121/commandRunner/runrun.png)](http://localhost:63342/markdownPreview/1492451121/markdown-preview-index-8hkmppp78rkagi87iceek81cm8.html#)GITHUB_TOKEN=$(gh auth token) terraform plan -input=false -no-color -out=/tmp/recon.tfplan >/dev/null 2>&1 || true terraform show -json /tmp/recon.tfplan \   | jq -r '.resource_changes[] | select(.address|test("github_branch_protection")) | [.address, (.change.after.required_pull_request_reviews[0].required_approving_review_count|tostring), (.change.after.required_status_checks[0].contexts//[]|join("+"))] | @tsv' \  > /tmp/planned_prot.tsv wc -l < /tmp/planned_prot.tsv`

Expected: `20` rows. If Task 1 is unresolved the plan will be absent — **STOP** and finish Task 1.

- [ ] **Step 3: Diff live against planned and flag every downgrade**

![](http://localhost:63342/markdownPreview/1336096853//opt/viavitae/repos/viavitae-control/docs/superpowers/plans)

`[![](http://localhost:63342/markdownPreview/1492451121/commandRunner/runrun.png)](http://localhost:63342/markdownPreview/1492451121/markdown-preview-index-8hkmppp78rkagi87iceek81cm8.html#)jq -r '[.repo,(.appr|tostring),(.co|tostring),(.ctx|join("+"))]|@tsv' /tmp/live_prot.jsonl | sort > /tmp/live_prot.tsv awk -F'\t' '{a=$1; sub(/^module\.repo\["/,"",a); sub(/"\].*$/,"",a); print a"\t"$2"\tfalse\t"$3}' /tmp/planned_prot.tsv | sort > /tmp/planned_norm.tsv join -t$'\t' /tmp/live_prot.tsv /tmp/planned_norm.tsv \  | awk -F'\t' '{d=""; if($5<$2)d=d" APPROVALS_LOST"; if($3=="true"&&$6=="false")d=d" CODEOWNER_LOST";                 if(length($7)<length($4))d=d" CONTEXTS_LOST";                printf "%s\tlive(appr=%s,co=%s,ctx=%s)\tplanned(appr=%s,co=%s,ctx=%s)%s\n",$1,$2,$3,$4,$5,$6,$7,(d==""?"\tOK":"\tDOWNGRADE:"d)}' \  > audit-evidence/protection-reconciliation.md grep -c DOWNGRADE audit-evidence/protection-reconciliation.md || true`

Expected on current config: **20** `DOWNGRADE` rows — `APPROVALS_LOST` on 19 repos (all but `viavitae-qa`, already at 0), `CODEOWNER_LOST` on 8 (`.github`, `viavitae-api`, `viavitae-brand`, `viavitae-clients`, `viavitae-demos`, `viavitae-docs`, `viavitae-infra`, `viavitae-template`), and `CONTEXTS_LOST` on all 20.

- [ ] **Step 4: Assert the gate**

![](http://localhost:63342/markdownPreview/1336096853//opt/viavitae/repos/viavitae-control/docs/superpowers/plans)

`[![](http://localhost:63342/markdownPreview/1492451121/commandRunner/runrun.png)](http://localhost:63342/markdownPreview/1492451121/markdown-preview-index-8hkmppp78rkagi87iceek81cm8.html#)test "$(grep -c DOWNGRADE audit-evidence/protection-reconciliation.md)" -eq 0 \  && echo "GATE PASS: no protection downgrades" || echo "GATE FAIL: apply is BLOCKED"`

Expected today: `GATE FAIL: apply is BLOCKED`. **This is the intended result.** Do not proceed to any apply until it reads `GATE PASS`.

- [ ] **Step 5: Commit**

![](http://localhost:63342/markdownPreview/1336096853//opt/viavitae/repos/viavitae-control/docs/superpowers/plans)

`[![](http://localhost:63342/markdownPreview/1492451121/commandRunner/runrun.png)](http://localhost:63342/markdownPreview/1492451121/markdown-preview-index-8hkmppp78rkagi87iceek81cm8.html#)git add audit-evidence/protection-reconciliation.md git commit -m "audit: reconcile live vs planned branch protection; record downgrade gate"`

---

### Task 5: Derive the Actions allow-list from evidence (VV-2026-09-006)

**Files:**

- Create: `scripts/audit_actions_inventory.sh`
    
- Create: `audit-evidence/actions-inventory.tsv`
    
- **Step 1: Write the harvester**
    

![](http://localhost:63342/markdownPreview/1336096853//opt/viavitae/repos/viavitae-control/docs/superpowers/plans)

``[![](http://localhost:63342/markdownPreview/1492451121/commandRunner/runrun.png)](http://localhost:63342/markdownPreview/1492451121/markdown-preview-index-8hkmppp78rkagi87iceek81cm8.html#)#!/usr/bin/env bash # audit_actions_inventory.sh — harvest every `uses:` reference from every workflow # in every repo, so actions_allowed_patterns is derived from evidence, not guesses. # Read-only. Usage: ORG=Via-Vitae ./scripts/audit_actions_inventory.sh set -euo pipefail ORG="${ORG:-Via-Vitae}" ROOT="$(cd "$(dirname "${BASH_SOURCE[0]}")/.." && pwd)" OUT="${OUT:-${ROOT}/audit-evidence/actions-inventory.tsv}" mkdir -p "$(dirname "${OUT}")" : > /tmp/wfpaths.$$ while read -r r; do   gh api "repos/${ORG}/${r}/git/trees/main?recursive=1" \    --jq '.tree[].path|select(startswith(".github/workflows/"))' 2>/dev/null \     | sed "s|^|${r} |" >> /tmp/wfpaths.$$ || true done < <(gh api "orgs/${ORG}/repos?per_page=100&type=all" --paginate --jq '.[].name' | sort) printf 'action_ref\tpinned_sha\tcount\tgithub_owned\tin_declared_patterns\tverdict\n' > "${OUT}" while read -r r p; do   gh api "repos/${ORG}/${r}/contents/${p}" --jq .content 2>/dev/null \     | base64 -d 2>/dev/null \     | grep -oE 'uses: *[A-Za-z0-9._/-]+@[A-Za-z0-9._/-]+' || true done < /tmp/wfpaths.$$ \  | sed 's/^ *uses: *//' | sort | uniq -c | sort -rn \  | awk '{ref=$2; cnt=$1; split(ref,a,"@"); owner=a[1];           pinned=(length(a[2])==40)?"yes":"NO";          go=(owner ~ /^(actions|github)\//)?"yes":"no";          declared=(owner ~ /^(actions\/(cache|checkout|download-artifact|setup-node|setup-python|upload-artifact)|github\/codeql-action|dependabot\/)/)?"yes":"no";          verdict=(pinned=="NO")?"MUTABLE_REF":((go=="yes"||declared=="yes")?"ALLOWED":"NEEDS_VERIFIED_OR_PATTERN");          printf "%s\t%s\t%s\t%s\t%s\t%s\n",ref,pinned,cnt,go,declared,verdict}' >> "${OUT}" rm -f /tmp/wfpaths.$$ echo "wrote $(( $(wc -l < "${OUT}") - 1 )) distinct action refs to ${OUT}"``

- [ ] **Step 2: Run it and assert no mutable refs remain unexplained**

![](http://localhost:63342/markdownPreview/1336096853//opt/viavitae/repos/viavitae-control/docs/superpowers/plans)

`[![](http://localhost:63342/markdownPreview/1492451121/commandRunner/runrun.png)](http://localhost:63342/markdownPreview/1492451121/markdown-preview-index-8hkmppp78rkagi87iceek81cm8.html#)chmod +x scripts/audit_actions_inventory.sh && bash -n scripts/audit_actions_inventory.sh ORG=Via-Vitae ./scripts/audit_actions_inventory.sh awk -F'\t' 'NR>1 && $6!="ALLOWED"{print $6"\t"$1}' audit-evidence/actions-inventory.tsv | sort | uniq -c | sort -rn`

Expected (verified by a real run on 2026-09-27): `wrote 39 distinct action refs`, with verdict counts **18 `ALLOWED` / 19 `NEEDS_VERIFIED_OR_PATTERN` / 2 `MUTABLE_REF`**. The `MUTABLE_REF` rows are the two internal `Via-Vitae/.github/.github/workflows/reusable-*.yml@main` refs. The 19 `NEEDS_VERIFIED_OR_PATTERN` rows span 6 distinct third-party owners: `aquasecurity/trivy-action` (28 uses), `pnpm/action-setup` (13), `docker/{build-push,login,metadata,setup-buildx,setup-qemu}-action` (7), `trufflesecurity/trufflehog` (4), `hashicorp/setup-terraform` (3), `astral-sh/setup-uv` (1).

- [ ] **Step 3: Resolve each `NEEDS_VERIFIED_OR_PATTERN`**

For each distinct owner, decide **verified-creator** vs **explicit pattern** and record the decision inline in `audit-evidence/actions-inventory.tsv` as a trailing comment column. Every row must end as `ALLOWED`, `ADD_PATTERN:<pattern>`, or `REJECT:<reason>`. No row may stay `NEEDS_VERIFIED_OR_PATTERN`.

- [ ] **Step 4: Convert mutable internal refs to SHAs**

`sha_pinning_required = true` is unsafe while any `uses:` ends in `@main`. Record the two offending refs and the repos containing them, and raise a remediation item to pin them to a 40-char SHA before `actions_sha_pinning_required` is enabled.

- [ ] **Step 5: Commit**

![](http://localhost:63342/markdownPreview/1336096853//opt/viavitae/repos/viavitae-control/docs/superpowers/plans)

`[![](http://localhost:63342/markdownPreview/1492451121/commandRunner/runrun.png)](http://localhost:63342/markdownPreview/1492451121/markdown-preview-index-8hkmppp78rkagi87iceek81cm8.html#)git add scripts/audit_actions_inventory.sh audit-evidence/actions-inventory.tsv git commit -m "feat(audit): derive Actions allow-list from harvested workflow inventory"`

---

### Task 6: Local-plane reconciliation (VV-2026-09-017)

**Files:**

- Create: `scripts/audit_local_plane.sh`
    
- Create: `audit-evidence/local-plane.tsv`
    
- **Step 1: Write the reconciler**
    

![](http://localhost:63342/markdownPreview/1336096853//opt/viavitae/repos/viavitae-control/docs/superpowers/plans)

``[![](http://localhost:63342/markdownPreview/1492451121/commandRunner/runrun.png)](http://localhost:63342/markdownPreview/1492451121/markdown-preview-index-8hkmppp78rkagi87iceek81cm8.html#)#!/usr/bin/env bash # audit_local_plane.sh — reconcile local clones under WORKDIR against the live org. # Detects: shadow clones (local, not in org), missing clones (org, not local), # uncontrolled dirs (not a git repo), detached remotes, dirty trees, off-main branches. # Read-only. Usage: WORKDIR=/opt/viavitae/repos ORG=Via-Vitae ./scripts/audit_local_plane.sh set -euo pipefail ORG="${ORG:-Via-Vitae}" WORKDIR="${WORKDIR:-/opt/viavitae/repos}" ROOT="$(cd "$(dirname "${BASH_SOURCE[0]}")/.." && pwd)" OUT="${OUT:-${ROOT}/audit-evidence/local-plane.tsv}" mkdir -p "$(dirname "${OUT}")" gh api "orgs/${ORG}/repos?per_page=100&type=all" --paginate --jq '.[].name' | sort > /tmp/org.$$.txt printf 'dir\tkind\tremote\tscheme\tbranch\tdirty\tverdict\n' > "${OUT}" # `find`, NOT a `*/` glob: the org-defaults repo is the HIDDEN dir `.github`, # which a non-dotglob silently skips (verified: glob yields 32, find yields 33). while IFS= read -r d; do   n="$(basename "${d}")"; d="${d}/"   if [[ ! -d "${d}.git" ]]; then     printf '%s\tNOT_GIT\t-\t-\t-\t-\tUNCONTROLLED\n' "${n}" >> "${OUT}"; continue   fi   u="$(git -C "${d}" remote get-url origin 2>/dev/null || echo NO_REMOTE)"   # Empty repos have an UNBORN HEAD: `rev-parse --abbrev-ref HEAD` prints "HEAD"   # to stdout AND exits non-zero, so `|| echo …` produces a two-line value that   # corrupts the TSV row. Test for commits first, then take exactly one line.   if git -C "${d}" rev-parse --verify -q HEAD >/dev/null 2>&1; then     b="$(git -C "${d}" rev-parse --abbrev-ref HEAD 2>/dev/null | head -n1)"   else     b=UNBORN  fi   s="$(git -C "${d}" status --porcelain 2>/dev/null | wc -l | tr -d ' ')"   case "${u}" in     NO_REMOTE)      v="NO_REMOTE_UNBACKED"; sc=none ;;     git@github.com:*) v=OK; sc=ssh ;;     https://*)      v=OK; sc=https ;;     *)              v="FOREIGN_REMOTE"; sc=other ;;   esac   if ! grep -qx "${n}" /tmp/org.$$.txt; then v="SHADOW_NOT_IN_ORG"; fi   # NOTE: a bare `[[ … ]] && …` aborts under `set -e` when the test is false.   if [[ "${b}" == "HEAD" ]]; then v="${v};DETACHED"; fi   if [[ "${b}" == "UNBORN" ]]; then v="${v};EMPTY_REPO"; fi   if [[ "${s}" != "0" ]]; then v="${v};DIRTY_${s}"; fi   if [[ "${b}" != "main" && "${b}" != "HEAD" && "${b}" != "UNBORN" ]]; then v="${v};OFF_MAIN"; fi   printf '%s\tGIT\t%s\t%s\t%s\t%s\t%s\n' "${n}" "${u}" "${sc}" "${b}" "${s}" "${v}" >> "${OUT}" done < <(find "${WORKDIR}" -maxdepth 1 -mindepth 1 -type d | sort) for n in $(cat /tmp/org.$$.txt); do   [[ -d "${WORKDIR}/${n}" ]] || printf '%s\tMISSING\t-\t-\t-\t-\tNO_LOCAL_CLONE\n' "${n}" >> "${OUT}" done rm -f /tmp/org.$$.txt echo "wrote $(( $(wc -l < "${OUT}") - 1 )) rows to ${OUT}"``

- [ ] **Step 2: Run and triage**

![](http://localhost:63342/markdownPreview/1336096853//opt/viavitae/repos/viavitae-control/docs/superpowers/plans)

`[![](http://localhost:63342/markdownPreview/1492451121/commandRunner/runrun.png)](http://localhost:63342/markdownPreview/1492451121/markdown-preview-index-8hkmppp78rkagi87iceek81cm8.html#)chmod +x scripts/audit_local_plane.sh && bash -n scripts/audit_local_plane.sh WORKDIR=/opt/viavitae/repos ORG=Via-Vitae ./scripts/audit_local_plane.sh awk -F'\t' 'NR>1 && $7!="OK"{print $1"\t"$7}' audit-evidence/local-plane.tsv`

Expected — this is the **verified** output from a real run on 2026-09-27 (17 non-OK rows out of 33):

![](http://localhost:63342/markdownPreview/1336096853//opt/viavitae/repos/viavitae-control/docs/superpowers/plans)

`jol-deploy	SHADOW_NOT_IN_ORG;EMPTY_REPO;DIRTY_7 viavitae-compliance	OK;EMPTY_REPO viavitae-control	OK;DIRTY_2;OFF_MAIN viavitae-data-governance	OK;EMPTY_REPO viavitae-demos	OK;OFF_MAIN viavitae-diocese-landing	UNCONTROLLED viavitae-infra	OK;DIRTY_1 viavitae-observability	OK;EMPTY_REPO viavitae-policies	OK;EMPTY_REPO;DIRTY_2 viavitae-qa	OK;OFF_MAIN viavitae-reusable-workflows	OK;EMPTY_REPO viavitae-runbooks	OK;EMPTY_REPO viavitae-threat-model	OK;EMPTY_REPO viavitae-training	OK;EMPTY_REPO viavitae-vendor-register	OK;EMPTY_REPO viavitae-web	OK;DIRTY_14 WelcomeScreen	UNCONTROLLED`

Note `jol-deploy` reads `EMPTY_REPO`, **not** `DETACHED`: it has no commits at all, so its 7 files are unrecoverable if the directory is lost. The 9 `OK;EMPTY_REPO` rows are the expected local mirror of the 9 live repos with `size == 0`.

- [ ] **Step 3: Decide disposition for each non-OK row**

Every row must be assigned exactly one of: `adopt` (add to `repos_data.tf`), `archive`, `delete-after-backup`, or `out-of-scope:<project>`. `jol-deploy` is expected to resolve to `out-of-scope:jol-control` and be **moved out of `/opt/viavitae/repos`**, per the Via-Vitae / jol-control separation rule. Record decisions in `audit-evidence/local-plane.tsv` as an added `disposition` column. Do not delete anything without explicit owner authorisation.

- [ ] **Step 4: Commit**

![](http://localhost:63342/markdownPreview/1336096853//opt/viavitae/repos/viavitae-control/docs/superpowers/plans)

`[![](http://localhost:63342/markdownPreview/1492451121/commandRunner/runrun.png)](http://localhost:63342/markdownPreview/1492451121/markdown-preview-index-8hkmppp78rkagi87iceek81cm8.html#)git add scripts/audit_local_plane.sh audit-evidence/local-plane.tsv git commit -m "feat(audit): reconcile local working copies against the live org inventory"`

---

### Task 7: CODEOWNERS resolvability and governance-file audit (VV-2026-09-009/010)

**Files:**

- Create: `audit-evidence/codeowners-audit.md`
    
- **Step 1: Extract every CODEOWNERS owner across the org**
    

![](http://localhost:63342/markdownPreview/1336096853//opt/viavitae/repos/viavitae-control/docs/superpowers/plans)

`[![](http://localhost:63342/markdownPreview/1492451121/commandRunner/runrun.png)](http://localhost:63342/markdownPreview/1492451121/markdown-preview-index-8hkmppp78rkagi87iceek81cm8.html#)cd /opt/viavitae/repos/viavitae-control : > audit-evidence/codeowners-audit.md awk -F'\t' 'NR>1 && $20>0 {print $1}' audit-evidence/gate-3-per-repo-sweep.tsv | while read -r n; do   for p in CODEOWNERS .github/CODEOWNERS docs/CODEOWNERS; do     c="$(gh api "repos/Via-Vitae/$n/contents/$p" --jq .content 2>/dev/null | base64 -d 2>/dev/null || true)"     [[ -n "${c}" ]] || continue    echo "## ${n} (${p})" >> audit-evidence/codeowners-audit.md    grep -oE '@[A-Za-z0-9/_-]+' <<<"${c}" | sort -u >> audit-evidence/codeowners-audit.md  done done grep -c '^## ' audit-evidence/codeowners-audit.md`

Expected: one `##` section per repo that has a CODEOWNERS file (21 repos have content; 9 are empty and have none).

- [ ] **Step 2: Verify every referenced owner resolves**

![](http://localhost:63342/markdownPreview/1336096853//opt/viavitae/repos/viavitae-control/docs/superpowers/plans)

`[![](http://localhost:63342/markdownPreview/1492451121/commandRunner/runrun.png)](http://localhost:63342/markdownPreview/1492451121/markdown-preview-index-8hkmppp78rkagi87iceek81cm8.html#)gh api orgs/Via-Vitae/members -q '.[].login' > /tmp/members.txt gh api orgs/Via-Vitae/teams -q '.[].slug' > /tmp/teams.txt gh api orgs/Via-Vitae/outside_collaborators -q '.[].login' > /tmp/outside.txt grep -oE '@[A-Za-z0-9/_-]+' audit-evidence/codeowners-audit.md | sort -u | while read -r o; do   h="${o#@}"; h="${h#Via-Vitae/}"   if grep -qx "${h}" /tmp/members.txt; then echo -e "${o}\tMEMBER"   elif grep -qx "${h}" /tmp/teams.txt; then echo -e "${o}\tTEAM"   elif grep -qx "${h}" /tmp/outside.txt; then echo -e "${o}\tOUTSIDE_COLLABORATOR"   else echo -e "${o}\tUNRESOLVED"; fi done | tee -a audit-evidence/codeowners-audit.md`

Expected today: `@JourneyOfLife → MEMBER`, `@IterVitae → OUTSIDE_COLLABORATOR`, `@Via-Vitae/{architects,compliance,dpo,legal,platform,security} → TEAM`, and **zero** `UNRESOLVED`. Any `UNRESOLVED` is a P1 finding — the rule silently does not resolve.

- [ ] **Step 3: Verify each owner actually has push access to that repo**

A CODEOWNERS entry only binds if the owner can read the repo. For every repo, intersect its CODEOWNERS owners with its collaborator list (column 24 of the sweep):

![](http://localhost:63342/markdownPreview/1336096853//opt/viavitae/repos/viavitae-control/docs/superpowers/plans)

`[![](http://localhost:63342/markdownPreview/1492451121/commandRunner/runrun.png)](http://localhost:63342/markdownPreview/1492451121/markdown-preview-index-8hkmppp78rkagi87iceek81cm8.html#)awk -F'\t' 'NR>1{print $1"\t"$24}' audit-evidence/gate-3-per-repo-sweep.tsv`

Flag every repo where a CODEOWNERS owner is absent from that repo's collaborators. Known hit: `.github` lists `@Via-Vitae/architects` while its only collaborator is `JourneyOfLife` — the team does contain `JourneyOfLife`, so it resolves; but combined with `require_code_owner_reviews=true`, `enforce_admins=true` and 1 required approval it produces the frozen-branch condition in VV-2026-09-008. Record the conclusion explicitly.

- [ ] **Step 4: Record the manage_files conflict decision**

Document in `audit-evidence/codeowners-audit.md` that `manage_files = true` must **not** be enabled for `viavitae-web` or `viavitae-qa` until `modules/repo/files.tf` supports person-based owners, because the module would emit `locals.tf:4-10` team-based CODEOWNERS over working person-based files. This blocks `docs/COMPLIANCE.md` P1-6 as currently written.

- [ ] **Step 5: Commit**

![](http://localhost:63342/markdownPreview/1336096853//opt/viavitae/repos/viavitae-control/docs/superpowers/plans)

`[![](http://localhost:63342/markdownPreview/1492451121/commandRunner/runrun.png)](http://localhost:63342/markdownPreview/1492451121/markdown-preview-index-8hkmppp78rkagi87iceek81cm8.html#)git add audit-evidence/codeowners-audit.md git commit -m "audit: verify CODEOWNERS resolvability and record manage_files conflict"`

---

### Task 8: Vulnerability triage and findings register

**Files:**

- Create: `audit-evidence/findings.json`
    
- Modify: `docs/COMPLIANCE.md`
    
- **Step 1: Triage the 1 critical and 10 high Dependabot alerts (VV-2026-09-007)**
    

![](http://localhost:63342/markdownPreview/1336096853//opt/viavitae/repos/viavitae-control/docs/superpowers/plans)

`[![](http://localhost:63342/markdownPreview/1492451121/commandRunner/runrun.png)](http://localhost:63342/markdownPreview/1492451121/markdown-preview-index-8hkmppp78rkagi87iceek81cm8.html#)printf 'repo\tseverity\tpackage\tvulnerable_range\tpatched_in\tdisposition\n' > audit-evidence/dependabot-triage.tsv for r in viavitae-docs viavitae-web viavitae-qa; do   gh api "repos/Via-Vitae/${r}/dependabot/alerts?state=open&per_page=100" \    --jq '.[]|select(.security_advisory.severity=="critical" or .security_advisory.severity=="high")           |[.security_advisory.severity,.dependency.package.name,.security_vulnerability.vulnerable_version_range,(.security_vulnerability.first_patched_version.identifier//"-")]|@tsv' \    | sed "s|^|${r}\t|" >> audit-evidence/dependabot-triage.tsv done awk -F'\t' 'NR>1{print NF}' audit-evidence/dependabot-triage.tsv | sort -u wc -l < audit-evidence/dependabot-triage.tsv`

Expected: `12` lines (1 header + 11 alerts: 1 critical in `viavitae-docs`, 7 high in `viavitae-web`, 3 high in `viavitae-qa`), and `5` as the only field count printed. The 6th column, `disposition`, is filled in by hand: every row must be marked `patch` / `accept-risk:<expiry>` / `not-exploitable:<reason>`. **Never** paste advisory content that could contain credentials.

- [ ] **Step 2: Emit `findings.json`**

Convert every row of the §3 table into the §3.1 JSON shape. Validate:

![](http://localhost:63342/markdownPreview/1336096853//opt/viavitae/repos/viavitae-control/docs/superpowers/plans)

`[![](http://localhost:63342/markdownPreview/1492451121/commandRunner/runrun.png)](http://localhost:63342/markdownPreview/1492451121/markdown-preview-index-8hkmppp78rkagi87iceek81cm8.html#)jq -e 'length>=17' audit-evidence/findings.json >/dev/null && echo "COUNT OK" jq -e '.[]|select((.framework_refs|length)==0 or (.declared.source=="") or (.actual.observed==""))' \  audit-evidence/findings.json`

Expected: `COUNT OK` and **no output** from the second command (every finding has framework refs, a `declared` source and an `actual` observation — `AUDIT_PROMPT.md` §6 makes all fields mandatory).

- [ ] **Step 3: Correct `docs/COMPLIANCE.md` (VV-2026-09-015)**

Edit these three claims to match live evidence, citing the sweep file:

- `docs/COMPLIANCE.md:54` — replace "All 30 repos are PUBLIC" with "25 public / 5 private (privatization completed 2026-09-24); P0-1 CLOSED".
- `docs/COMPLIANCE.md:73` — replace "All 30 repos are EMPTY" with "9 empty / 21 active; branch protection live on 20, unmanaged".
- `repos_data.tf:11` — replace "all 30 are public today" with "25 public, 5 private (see per-entry `visibility`)".

Then reclassify every C1–C18 status using the five-value vocabulary (`docs/COMPLIANCE.md:38-44`). No control may read `enforced-verified` without a cited live observation. Based on current evidence the expected reclassification is: C1 `partial-5-of-30`, C2 `enforced-verified` (names only) + new gap for protection drift, C3/C4 `enforced-verified` for public / `gap-plan-tier` for private, C5 `enforced-verified` (alerts open — see VV-2026-09-007), C6 `drifted-unmanaged`, C7 `partial-unmanaged`, C9 `declared-not-applied`, C10 `not-enforced`, C11 `declared-not-applied`, C12 `out-of-band-open`, C17/C18 `control-repo-only`.

- [ ] **Step 4: Commit**

![](http://localhost:63342/markdownPreview/1336096853//opt/viavitae/repos/viavitae-control/docs/superpowers/plans)

`[![](http://localhost:63342/markdownPreview/1492451121/commandRunner/runrun.png)](http://localhost:63342/markdownPreview/1492451121/markdown-preview-index-8hkmppp78rkagi87iceek81cm8.html#)git add audit-evidence/findings.json audit-evidence/dependabot-triage.tsv docs/COMPLIANCE.md repos_data.tf git commit -m "audit: emit findings register and correct stale compliance claims"`

---

### Task 9: Extend drift detection beyond repo names (VV-2026-09-005)

**Files:**

- Modify: `scripts/detect_drift.sh`
    
- Modify: `.github/workflows/audit.yml`
    
- **Step 1: Add a protection-drift check**
    

Append to `scripts/detect_drift.sh`, before the final `exit` logic, a block that fails when a live protection mechanism is absent from Terraform state:

![](http://localhost:63342/markdownPreview/1336096853//opt/viavitae/repos/viavitae-control/docs/superpowers/plans)

`[![](http://localhost:63342/markdownPreview/1492451121/commandRunner/runrun.png)](http://localhost:63342/markdownPreview/1492451121/markdown-preview-index-8hkmppp78rkagi87iceek81cm8.html#)# --- PROTECTION DRIFT: live mechanism not represented in state --------------- echo "--- PROTECTION drift (live mechanism not in Terraform state) ---" state_list="$(cd "${ROOT}" && terraform state list 2>/dev/null || true)" pdrift=0 for n in "${live[@]}"; do   has_live=0   gh api "repos/${ORG}/${n}/branches/main/protection" >/dev/null 2>&1 && has_live=1   rs="$(gh api "repos/${ORG}/${n}/rulesets" --jq 'length' 2>/dev/null || echo 0)"   if [[ "${rs}" != "0" ]]; then has_live=1; fi   if [[ "${has_live}" -eq 1 ]] \     && ! grep -qE "module.repo\[\"${n}\"\].github_branch_protection|github_repository_ruleset" <<<"${state_list}"; then     echo "  ! ${n}: live protection is NOT managed by Terraform"     pdrift=1   fi done [[ "${pdrift}" -eq 0 ]] && echo "  (none)" || drift=1`

- [ ] **Step 2: Verify it detects the known drift**

![](http://localhost:63342/markdownPreview/1336096853//opt/viavitae/repos/viavitae-control/docs/superpowers/plans)

`[![](http://localhost:63342/markdownPreview/1492451121/commandRunner/runrun.png)](http://localhost:63342/markdownPreview/1492451121/markdown-preview-index-8hkmppp78rkagi87iceek81cm8.html#)ORG=Via-Vitae ./scripts/detect_drift.sh; echo "EXIT=$?"`

Expected: `DRIFT DETECTED` and `EXIT=1`, listing all 20 classic-protected repos plus the 4 ruleset repos. This is the **correct** result — the guard now sees what it was blind to. It must be green only after Task 10 brings protection under management.

- [ ] **Step 3: Wire it into CI and commit**

Confirm `.github/workflows/audit.yml` invokes `scripts/detect_drift.sh`; mark the protection check non-blocking (`continue-on-error: true`) until Task 10 lands, then flip it to blocking.

![](http://localhost:63342/markdownPreview/1336096853//opt/viavitae/repos/viavitae-control/docs/superpowers/plans)

`[![](http://localhost:63342/markdownPreview/1492451121/commandRunner/runrun.png)](http://localhost:63342/markdownPreview/1492451121/markdown-preview-index-8hkmppp78rkagi87iceek81cm8.html#)git add scripts/detect_drift.sh .github/workflows/audit.yml git commit -m "feat(audit): detect unmanaged live branch protection and rulesets as drift"`

---

### Task 10: Go/No-Go decision record

**Files:**

- Create: `audit-evidence/deployment-readiness.md`
    
- **Step 1: Evaluate every gate**
    

|Gate|Criterion|Command|Blocking|
|---|---|---|---|
|G1|`terraform plan` exits 0|Task 1 Step 4|yes|
|G2|`0 to destroy` in plan|`terraform show -json /tmp/recon.tfplan \| jq '[.resource_changes[].change.actions[]]\|unique'`|yes|
|G3|Zero protection downgrades|Task 4 Step 4 prints `GATE PASS`|yes|
|G4|Every live protection in state|`terraform state list \| grep -c branch_protection` ≥ 20|yes|
|G5|Actions allow-list derived, no `NEEDS_VERIFIED_OR_PATTERN` rows|Task 5 Step 3|yes|
|G6|No `MUTABLE_REF` while `sha_pinning_required=true`|Task 5 Step 4|yes|
|G7|0 open critical Dependabot alerts|Task 8 Step 1|yes|
|G8|`.github` unfrozen (a valid approver exists)|`gh api repos/Via-Vitae/.github/collaborators -q '.[].login'`|yes|
|G9|No local shadow/uncontrolled dirs without disposition|Task 6 Step 3|no|
|G10|`docs/COMPLIANCE.md` matches live evidence|Task 8 Step 3|no|
|G11|`detect_drift.sh` exits 0|Task 9 Step 2|yes|
|G12|No personal data in tracked files|`git grep -nE '[A-Za-z0-9._%+-]+@(gmail|outlook|

- [ ] **Step 2: Write the decision**

Record each gate as `PASS`/`FAIL` with the verbatim command output beneath it, then one of:

- `DECISION: NO-GO` — list the blocking gates that failed and the exact remediation task for each.
    
- `DECISION: GO` — only when **all** blocking gates are `PASS`, and only with an explicit written authorisation from the owner. Absent that authorisation the record must read `NO-GO (awaiting owner authorisation)` even if every gate passes.
    
- **Step 3: Commit**
    

![](http://localhost:63342/markdownPreview/1336096853//opt/viavitae/repos/viavitae-control/docs/superpowers/plans)

`[![](http://localhost:63342/markdownPreview/1492451121/commandRunner/runrun.png)](http://localhost:63342/markdownPreview/1492451121/markdown-preview-index-8hkmppp78rkagi87iceek81cm8.html#)git add audit-evidence/deployment-readiness.md git commit -m "audit: record pre-deployment go/no-go decision with gate evidence"`

---

## 6. Deployment sequence (only after `DECISION: GO`)

Ordered so that no step can weaken a live control, and each step is separately reversible:

1. Fix the token scope (Task 1) — no org change.
2. Import existing protections and rulesets into state: add `github_branch_protection` import blocks (ID = `R_kgDO…:main` node-id form) and `github_repository_ruleset` imports for the 4 stray `protect-main` rulesets, mirroring `imports.tf:13-18` and `repo_rulesets.tf:61-64`. Plan must show **import, not create**.
3. Re-baseline declared values from live (Task 4 evidence) so the first managed apply is a **no-op**.
4. Apply the no-op. Verify `terraform plan` is then empty.
5. Only now make deliberate, one-at-a-time tightening changes (Actions allow-list, org settings, review counts), each with its own saved plan file applied verbatim: `terraform plan -target=<addr> -out=<file>` → inspect → `terraform apply <file>`.
6. Migrate state to the S3 backend (`state/README.md`, P1-4) **before** the control plane becomes the system of record.
7. Enforce org 2FA out-of-band (`policies/enforce_sso.sh --apply`, P1-1) after confirming the sole member's recovery path.

---

## 7. Self-review

**Coverage:** all 30 repos appear in §1 strata (1+5+1+19+4 = 30); all 12 dimensions in §2 have a command and a pass criterion; every finding in §3 has evidence, a control ID and a framework path; every task has exact commands and expected output. Stratum C/D live protection values are **captured, not deferred**.

**Harness validated by execution, not inspection.** All three scripts in this plan were extracted from this document, passed `bash -n`, and were run read-only against the live org on 2026-09-27: `audit_repo_sweep.sh` → 30 rows × 24 columns, no empty fields, strata assertion `public=25 private=5 empty=9 classic_bp=20 rulesets=5 unreadable=5 in_state=5`; `audit_actions_inventory.sh` → 39 refs; `audit_local_plane.sh` → 33 rows. Three real defects were found and fixed during that validation and are now encoded as comments in the scripts: a `*/` glob skipping the hidden `.github` dir, unborn-HEAD repos emitting a two-line value that corrupted TSV rows, and `gh` printing HTTP error bodies to **stdout** so `cmd || echo FALLBACK` concatenated error JSON into evidence columns.

**Placeholder scan:** no `TBD`/`TODO`/"handle edge cases" text is present, and no cell is deferred — the Stratum D baseline that an earlier draft marked `not yet captured` has been filled from a live sweep.

**Consistency:** finding IDs are `VV-2026-09-NNN` per `AUDIT_PROMPT.md` §6; control IDs are C1–C18 per `docs/COMPLIANCE.md`; status vocabulary is the five-value set at `docs/COMPLIANCE.md:38-44`; severity is assigned from the §7 rubric, never from impression.

**Known limits, stated honestly:**

- Rulesets on the 5 private repos return HTTP 403 on the Free plan, so D4 for Stratum B is **unknown**, not "absent". Recorded as an evidence gap, not a pass.
- Whether `aquasecurity`, `pnpm`, `trufflesecurity`, `docker`, `hashicorp`, `astral-sh` are GitHub **verified creators** is not queryable with the current token; Task 5 Step 3 resolves it by decision, and until then the allow-list switch is blocked.
- Attribution of `secret_scanning=enabled` on the 25 public repos cannot be separated from GitHub's default-on behaviour, because those repos are not in state. C3/C4 are therefore recorded as satisfied-by-outcome but **not IaC-enforced** — reversible by any repo admin.