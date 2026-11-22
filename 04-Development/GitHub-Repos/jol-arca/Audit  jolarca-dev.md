


Let me read a sample of the registry files to understand the compliance frameworks, criticality levels, and current state of each repository.

Now let me check the existing drift findings and policy defaults to complete the audit context.

Now I have a complete picture. Let me also verify the staging directories and check the live GitHub API state.

Now I have the complete empirical picture. Here is the comprehensive audit report and detailed plan.

---

## Fleet Audit Report — jolarca-dev Organization

### Repository Count

**16 repositories** exist on GitHub (including `.github`), of which **15 are managed** in the fleet registry (`repos/*.yml`). The `.github` repo is the org health repo managed by [health-repo.tf]([repo]/jolarca-control/health-repo.tf).

### Live State Summary

| # | Repository | Visibility | Size (KB) | Branch Protected | Launch Status | Criticality | Registry Match |
|---|-----------|-----------|-----------|-----------------|---------------|-------------|---------------|
| 1 | `.github` | public | 17 | **true** | n/a (health) | n/a | Managed (health-repo.tf) |
| 2 | `jolarca` | public | 1187 | **true** | in-development | tier-1 | OK |
| 3 | `jolarca-compliance` | **private** | 230 | false | operational | tier-2 | OK |
| 4 | `jolarca-consent` | public | **0** | n/a | planned | tier-1 | OK |
| 5 | `jolarca-control` | **private** | 327 | false | operational | tier-1 | OK |
| 6 | `jolarca-data` | **private** | 121 | false | in-development | tier-1 | OK |
| 7 | `jolarca-docs` | public | **0** | n/a | planned | tier-3 | OK |
| 8 | `jolarca-dr` | public | **0** | n/a | planned | tier-1 | OK |
| 9 | `jolarca-identity` | **public** | 27 | false | in-development | tier-1 | **DRIFT** |
| 10 | `jolarca-infrastructure` | **private** | 430 | false | operational | tier-1 | OK |
| 11 | `jolarca-legal` | **private** | 215 | false | operational | tier-2 | OK |
| 12 | `jolarca-observability` | public | **0** | n/a | in-development | tier-2 | **DRIFT** |
| 13 | `jolarca-payments` | public | **0** | n/a | planned | tier-1 | OK |
| 14 | `jolarca-runbooks` | public | **0** | n/a | planned | tier-2 | OK |
| 15 | `jolarca-security` | **private** | 58 | false | planned | tier-2 | **DRIFT** |
| 16 | `jolarca-vendor` | public | **0** | n/a | planned | tier-2 | OK |

---

### Critical Findings Requiring Immediate Attention

**FINDING A — Visibility drift on `jolarca-identity` (S1, BLOCKING)**
- Registry declares `visibility: private` (line 15 of [repos/jolarca-identity.yml]([repo]/jolarca-control/repos/jolarca-identity.yml)).
- Live GitHub API returns **public**. This is the same D-01 pattern: a PCI-DSS-scoped repo whose RBAC policy and IAM procedures are publicly readable.

**FINDING B — Visibility drift on `jolarca-observability` (S2)**
- Registry declares `visibility: private` (line 14 of [repos/jolarca-observability.yml]([repo]/jolarca-control/repos/jolarca-observability.yml)).
- Live GitHub API returns **public**. Logging configs reveal detection thresholds and retention windows.

**FINDING C — Visibility drift on `jolarca-security` (S2)**
- Registry declares `visibility: public` (line 3 of [repos/jolarca-security.yml]([repo]/jolarca-control/repos/jolarca-security.yml)).
- Live GitHub API returns **private**. The registry says public but the repo was flipped private (likely during the D-01 remediation batch). The YAML must be updated to match reality, or the repo flipped back.

**FINDING D — 7 empty repositories still at size 0 (S3)**
- `jolarca-consent`, `jolarca-docs`, `jolarca-dr`, `jolarca-observability`, `jolarca-payments`, `jolarca-runbooks`, `jolarca-vendor` have zero content. They exist on GitHub but have no `main` branch. Staged content exists only for 3 of 16 repos (`jolarca-dr`, `jolarca-identity`, `jolarca-payments` in `staging/`).

**FINDING E — Branch protection absent on 14 of 16 repos (S1, D-33)**
- Only `jolarca` and `.github` (both public) have `.protected = true`.
- All 7 private repos have `.protected = false` — the Free plan blocks branch protection on private repos.
- The 7 empty repos have no branch to protect yet.

---

### Detailed Pre-Deployment Audit Plan

#### Phase 1 — Visibility & Exposure Remediation (BLOCKING)

| # | Action | Repo(s) | Severity | Ref |
|---|--------|---------|----------|-----|
| 1.1 | Flip `jolarca-identity` to **private** via GitHub API | jolarca-identity | S1 | D-01 pattern |
| 1.2 | Flip `jolarca-observability` to **private** via GitHub API | jolarca-observability | S2 | D-01 pattern |
| 1.3 | Decide `jolarca-security`: update YAML to `private` (match live) or flip repo to `public` (match YAML) | jolarca-security | S2 | New finding |
| 1.4 | Verify no other visibility mismatches after 1.1–1.3 | All | S1 | — |

#### Phase 2 — Branch Protection (BLOCKING for PCI-DSS repos)

| # | Action | Repo(s) | Severity | Ref |
|---|--------|---------|----------|-----|
| 2.1 | **Owner decision**: upgrade `jolarca-dev` to GitHub Team/Pro, OR record dated risk acceptance in `compliance-gates.yml` exceptions.active | Org-wide | S1 | D-33 |
| 2.2 | If upgraded: reconcile [branch-protection.tf]([repo]/jolarca-control/branch-protection.tf) with [policy/repo-defaults.yml]([repo]/jolarca-control/policy/repo-defaults.yml) baseline, set `enable_branch_protection = true`, import surviving rules on `jolarca`/`.github` | jolarca-control | S1 | D-33 |
| 2.3 | If not upgraded: protect the 9 public repos now (protection is free on public), record private-repo gap as exception | Mixed | S1 | — |
| 2.4 | Fix `drift_detect.py` to read `.protected` boolean (works on Free) instead of relying on the 403 protection endpoint | jolarca-control | S1 | D-33 |

#### Phase 3 — Content Population & Staging

| # | Action | Repo(s) | Severity | Ref |
|---|--------|---------|----------|-----|
| 3.1 | Push staged content from `staging/jolarca-dr/` to the live repo | jolarca-dr | S3 | — |
| 3.2 | Push staged content from `staging/jolarca-payments/` to the live repo | jolarca-payments | S3 | — |
| 3.3 | Push staged content from `staging/jolarca-identity/` to the live repo (after Phase 1 visibility fix) | jolarca-identity | S3 | — |
| 3.4 | Create staging directories and initial content for the 4 remaining empty repos with no staging: `jolarca-consent`, `jolarca-docs`, `jolarca-runbooks`, `jolarca-vendor` | 4 repos | S3 | — |
| 3.5 | Ensure every repo has at minimum: `README.md`, `SECURITY.md`, `.github/CODEOWNERS` | All empty repos | S2 | D-15 pattern |

#### Phase 4 — Compliance Gate Verification

| # | Action | Repo(s) | Severity | Ref |
|---|--------|---------|----------|-----|
| 4.1 | Run `scripts/validate_repos.py` — verify all 15 YAML definitions pass | jolarca-control | S1 | — |
| 4.2 | Run `scripts/compliance_check.py` — verify framework declarations match policy | jolarca-control | S1 | — |
| 4.3 | Run `scripts/drift_detect.py` — verify all drift signals fire correctly | jolarca-control | S1 | D-23/D-26 |
| 4.4 | Run `scripts/check_fleet_separation.sh` — verify ADR-0004 R1/R2/R3 | jolarca-control | S1 | — |
| 4.5 | Verify Dependabot coverage exists on all repos with dependencies (currently only 5 of 16 have `.github/dependabot.yml`) | Fleet-wide | S2 | — |

#### Phase 5 — Organizational Controls

| # | Action | Repo(s) | Severity | Ref |
|---|--------|---------|----------|-----|
| 5.1 | Enable org-wide 2FA (manual, GitHub UI) | Org | S1 | D-18 |
| 5.2 | Verify `TF_GITHUB_TOKEN` and `TF_GITHUB_TOKEN_READONLY` secrets exist | Org | S1 | D-23 |
| 5.3 | Resolve D-02 (remote Terraform state backend) before any `terraform apply` | jolarca-infrastructure | S0 | D-02 |
| 5.4 | Resolve D-13 (dual Terraform ownership window) — execute state migration runbook | jolarca-infrastructure / jolarca-control | S0 | D-13 |

#### Phase 6 — Per-Repository Compliance Audit Matrix

For each of the 15 managed repos, verify:

| Check | Tier-1 (6 repos) | Tier-2 (6 repos) | Tier-3 (1 repo) |
|-------|-------------------|-------------------|------------------|
| Visibility matches YAML | **Required** | **Required** | **Required** |
| Branch protection active | **Required** | **Required** | Required |
| Required CI contexts declared & producing | **Required** | **Required** | Required |
| CODEOWNERS present & resolvable | **Required** | **Required** | Required |
| Secret scanning enabled | **Required** | **Required** | Required |
| Dependabot configured | **Required** | Required | Optional |
| Data classification ≤ visibility | **Required** (confidential ≠ public) | **Required** | Required |
| Compliance frameworks declared | soc2+gdpr+iso27001+pci-dss | Per YAML | Per YAML |
| SECURITY.md present | **Required** | **Required** | Required |

---

### Professional Opinion

**The fleet is in a pre-deployment state with three S0/S1 blockers that must be resolved before any production reliance:**

1. **Visibility exposure is ongoing.** The RoPA, vendor assessments, and contracts in `jolarca-compliance` and `jolarca-legal` were publicly readable for months (D-01/D-31). Two additional repos (`jolarca-identity`, `jolarca-observability`) have since drifted the same way. This is GDPR Art. 5(1)(f) and Art. 32 exposure — assess notifiability under Art. 33 before doing anything else.

2. **Branch protection does not exist on any private repo.** This is not a configuration gap — it is a plan-tier constraint (D-33). The four PCI-DSS-scoped private repos can be force-pushed, deleted, or pushed to directly without review. The decision is binary: upgrade to GitHub Team, or accept and document the gap with a dated exception. There is no middle ground on the Free plan for private repos.

3. **The Terraform state is single-copy and dual-owned** (D-02 + D-13). Any `terraform apply` from either root can undo the other. The state migration runbook must complete before the control plane takes ownership.

**Risk-ranked order of execution:** Phase 5.1 (2FA) → Phase 1 (visibility) → Phase 5.3–5.4 (state) → Phase 2 (branch protection decision) → Phase 4 (validation gates) → Phase 3 (content) → Phase 6 (per-repo audit).

The 16-repo fleet is structurally sound in its registry design — the YAML allow-list model, drift detection, and compliance gates are well-engineered. The gap is between declared intent and live reality, which is exactly what this audit plan addresses.