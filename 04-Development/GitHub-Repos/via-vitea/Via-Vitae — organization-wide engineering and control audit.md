


# Via-Vitae — organization-wide engineering and control audit

Date: 29 October 2026. Scope: all 31 accessible organization repositories. This is a point-in-time source/control assessment with local tests and selected CI evidence; it is not a penetration test, production-system inspection, legal opinion, SOC 2 report or ISO certification.

## Decision

**Do not treat the organization as production-ready or expand Qoder into customer-data workflows yet.** First repair the misleading verification gates, reconcile the website/API contracts, implement real deployment/restore paths, and triage dependency alerts. No new source changes, merges, deployment actions or organization-setting mutations were made in this audit phase.

The earlier API and Hermes fixes remain draft PRs. Their passing branch checks do not change the default branches audited here or establish production deployment. See [API PR #12](https://github.com/Via-Vitae/viavitae-api/pull/12) and [Hermes PR #12](https://github.com/Via-Vitae/viavitae-hermes-agents/pull/12).

## Coverage and limits

- **31/31** repositories received a live control/metadata sweep: 26 public, 5 private. Pagination limit 100 exceeded every returned repository/alert population; no additional pages were needed for those queries.
- **23/23 nonempty repositories** were downloaded and default-branch snapshots scanned with Gitleaks; zero findings were reported. These were shallow snapshots, not complete Git history.
- **10 landing repositories and the template repository** contain governance scaffolding, not a completed application. **Eight repositories are empty.** The dedicated compliance repository has one tracked register file. Empty repositories do not prove absence of controls elsewhere: policies, DPIAs, runbooks and monitoring configuration also exist in docs/infra.
- Static inspection covered manifests, workflows, repository controls and targeted high-risk code paths. Executable checks covered web, API/Hermes previously, QA, demos, brand, clients, docs and control-plane guards. This does not mean every line or behavior in every repository was exhaustively tested.
- No customer data, live cluster, production database, signed processor agreements, identity-provider administration or authoritative Terraform state was supplied. Effectiveness of those operational controls remains unverified.

The [repository matrix](REPOSITORY-MATRIX.md) lists every repository and exact inspected revision. The JSON files retain machine-readable evidence. Framework references in the findings are proposed control-intent mappings for accountable review, not determinations of legal compliance.

## Controls positively observed

- Organization 2FA requirement enabled; default GitHub Actions token permissions read-only; workflow self-approval disabled; selected-action policy enabled.
- All 26 public repositories report native secret scanning and push protection enabled. Dependabot security updates report enabled across the inventory.
- Public branch protection/ruleset responses were inspected individually. The control repository requires provenance; Hermes is the exception with no required checks.
- Website: lint, typecheck, 125 tests, locale/SEO/header checks passed on Node 22. Dependencies were installed from the lockfile with install scripts disabled. No browser E2E or production build was run in this phase.
- Brand: token validation, 126/126 declared contrast pairs, and the default Cyrillic check passed. This does not prove complete rendered-page accessibility or full binary-font coverage.
- Docs: frontmatter, locale parity and internal-link checks passed. Markdown/Hugo production build and live links were not fully audited.
- Clients: structural/privacy-pattern checks passed, but the main validator checked zero real tenants; see finding 112.
- Control plane: pinned Terraform version matched, formatting and validate passed, both guard-test scripts passed. Infrastructure Terraform formatting passed; provider/live infrastructure validation was not performed.
- QA: 7 unit/5 contract tests and typecheck passed, but 7 assertions are placeholders. Demos: typecheck passed; root seed verification failed.

## Prioritized findings

| ID | Priority | Finding |
|---|---|---|
| VV-2026-10-101 | P1 | Open dependency alerts need risk-based triage |
| VV-2026-10-102 | P1 | Website and API disagree on assessment and quote contracts |
| VV-2026-10-103 | P1 | QA can report success without substantive verification |
| VV-2026-10-104 | P1 | Disaster-recovery script emits Executed without restoring data |
| VV-2026-10-105 | P1 | Network validation is a placeholder and contradicts the policy CIDR |
| VV-2026-10-106 | P1 | Documented deployment reviewer gates are not configured |
| VV-2026-10-107 | P1 | Platform GitOps paths deploy placeholders rather than workloads |
| VV-2026-10-108 | P1 | Demo reset and seed verification are incomplete |
| VV-2026-10-109 | P1 | Hermes CI is not a mandatory merge barrier |
| VV-2026-10-110 | P1 | Independent code review is not enforced for active public repositories |
| VV-2026-10-111 | P1 | Private compliance repositories lack verified platform protection |
| VV-2026-10-112 | P1 | Client consent checker accepts existence instead of evidence validity |
| VV-2026-10-113 | P2 | Control-plane state reconciliation could not be verified |
| VV-2026-10-114 | P2 | Organization action pinning has exceptions |

P1 means a release/control blocker or materially misleading evidence. P2 means a verification/reproducibility gap requiring remediation or accountable disposition. No P0 exploit was demonstrated. GitHub advisory severities are separate from this audit prioritization.

## VV-2026-10-101 — Open dependency alerts need risk-based triage

**Observed:** 106 open alert records: 12 critical, 39 high, 47 medium, 8 low across web, brand, docs, QA and API. These records cover 86 distinct advisory IDs and can duplicate advisories across manifests; they are not 106 distinct exploitable vulnerabilities.

**Why it matters:** Build/developer tools as well as product dependencies may be affected. Runtime reachability was not demonstrated. Critical web alerts include testing/tooling packages; brand alerts concern image tooling.

**Repair sequence:** Triage by actual locked version, transitive path, input exposure, runtime/build scope and advisory prerequisites. Update or replace affected dependencies in separate reviewable changes; record justified exceptions with an expiry.

**How a student can verify the repair:** Rebuild with lockfiles, run relevant tests, verify patched versions and recheck alert state. Do not equate a successful CRITICAL-only CI scanner with closure of every GitHub alert.

**Evidence:** see the named JSON/log artifacts and exact commands in findings.json.

## VV-2026-10-102 — Website and API disagree on assessment and quote contracts

**Observed:** Website-shaped synthetic quote and assessment payloads are rejected by the API models. Quote tiers differ; the website expects PDF/200 while API defines JSON/201. The assessment shape and expected response also differ.

**Why it matters:** The purchase/assessment funnel cannot interoperate as committed, even though isolated frontend tests pass.

**Repair sequence:** Choose one versioned producer-owned contract, reconcile fields and response types, generate consumer types and validate real producer responses. Document any intentional intermediate adapter.

**How a student can verify the repair:** Run the website proxy against the actual API in a local/staging integration test with valid and invalid payloads; do not mock away the contract boundary.

**Evidence:** [viavitae-web/app/api/quote/route.ts:21](https://github.com/Via-Vitae/viavitae-web/blob/ca4656378679bf1b5aaf38bb5ddcd61b2ec6fdde/app/api/quote/route.ts#L21); [viavitae-api/app/schemas/quote.py:9](https://github.com/Via-Vitae/viavitae-api/blob/328eb14830c7bb71c8f02b7eca36cf859308e55a/app/schemas/quote.py#L9); [viavitae-api/app/schemas/assessment.py:29](https://github.com/Via-Vitae/viavitae-api/blob/328eb14830c7bb71c8f02b7eca36cf859308e55a/app/schemas/assessment.py#L29); [viavitae-web/docs/contracts/quotes.openapi.yaml:37](https://github.com/Via-Vitae/viavitae-web/blob/ca4656378679bf1b5aaf38bb5ddcd61b2ec6fdde/docs/contracts/quotes.openapi.yaml#L37)

## VV-2026-10-103 — QA can report success without substantive verification

**Observed:** Seven source assertions unconditionally pass. Local QA reports 7 unit and 5 contract tests passed, including these placeholders. Successful nightly run 40812219232 skipped load, performance, accessibility and E2E jobs after staging was unavailable.

**Why it matters:** Green status does not establish provider signature checking, tenant-schema compatibility, PII screening or end-to-end readiness.

**Repair sequence:** Replace unconditional assertions with real negative and positive cases. Make required staging checks fail or report an explicitly incomplete result when prerequisites are absent; include executed/skipped counts in the aggregate.

**How a student can verify the repair:** Break a contract and confirm CI fails; remove staging configuration and confirm release gates do not pass; run a configured staging suite and preserve results.

**Evidence:** [viavitae-qa/contract/stripe/webhook-contract.spec.ts:5](https://github.com/Via-Vitae/viavitae-qa/blob/2d670eccce2e63592a16bae3e59d90e8dcccd809/contract/stripe/webhook-contract.spec.ts#L5); [viavitae-qa/unit/fixtures-validator.spec.ts:5](https://github.com/Via-Vitae/viavitae-qa/blob/2d670eccce2e63592a16bae3e59d90e8dcccd809/unit/fixtures-validator.spec.ts#L5); [viavitae-qa/.github/workflows/nightly-e2e.yml:63](https://github.com/Via-Vitae/viavitae-qa/blob/2d670eccce2e63592a16bae3e59d90e8dcccd809/.github/workflows/nightly-e2e.yml#L63); [viavitae-qa/.github/workflows/nightly-e2e.yml:283](https://github.com/Via-Vitae/viavitae-qa/blob/2d670eccce2e63592a16bae3e59d90e8dcccd809/.github/workflows/nightly-e2e.yml#L283)

## VV-2026-10-104 — Disaster-recovery script emits Executed without restoring data

**Observed:** A reviewed copy run in a disposable local directory returned exit 0 and generated Status: Executed while the restore body only printed placeholders. No live restore was attempted.

**Why it matters:** Recovery evidence can misrepresent an unperformed restore; actual recoverability remains unverified.

**Repair sequence:** Return a nonzero/incomplete result until real restore and integrity checks are implemented. Record measured duration, restored artifact identifiers and independent verification, clearly separating dry runs from actual drills.

**How a student can verify the repair:** Restore synthetic data into an isolated target, compare integrity checks and measured recovery objectives, test failure paths, and obtain accountable review of the evidence.

**Evidence:** [viavitae-infra/scripts/dr-drill.sh:179](https://github.com/Via-Vitae/viavitae-infra/blob/a4145b540bf544a7300b167d10b6c3ed53674c2d/scripts/dr-drill.sh#L179); [viavitae-infra/scripts/dr-drill.sh:197](https://github.com/Via-Vitae/viavitae-infra/blob/a4145b540bf544a7300b167d10b6c3ed53674c2d/scripts/dr-drill.sh#L197)

## VV-2026-10-105 — Network validation is a placeholder and contradicts the policy CIDR

**Observed:** The dry-run statement says 10.10.40.0/24, but the committed policy permits 10.10.20.0/24. The real probe creates a pod but has no ingress test. No cluster observation established the effective source address.

**Why it matters:** The claimed evidence does not validate the intended ingress boundary; the effective policy may deny intended traffic or permit unintended sources.

**Repair sequence:** Measure source addresses in a controlled environment, reconcile topology and policy, implement positive/negative connectivity assertions and cleanup on failures. Do not change the CIDR solely on the basis of comments.

**How a student can verify the repair:** From explicitly approved and denied sources, prove allowed traffic succeeds and forbidden traffic fails; inspect effective manifests and retain sanitized results.

**Evidence:** [viavitae-infra/scripts/netpol-probe.sh:54](https://github.com/Via-Vitae/viavitae-infra/blob/a4145b540bf544a7300b167d10b6c3ed53674c2d/scripts/netpol-probe.sh#L54); [viavitae-infra/scripts/netpol-probe.sh:77](https://github.com/Via-Vitae/viavitae-infra/blob/a4145b540bf544a7300b167d10b6c3ed53674c2d/scripts/netpol-probe.sh#L77); [viavitae-infra/k8s/policies/network-policies/allow-ingress-from-haproxy.yaml:26](https://github.com/Via-Vitae/viavitae-infra/blob/a4145b540bf544a7300b167d10b6c3ed53674c2d/k8s/policies/network-policies/allow-ingress-from-haproxy.yaml#L26); [viavitae-infra/ansible/inventories/prod/hosts.yml:89](https://github.com/Via-Vitae/viavitae-infra/blob/a4145b540bf544a7300b167d10b6c3ed53674c2d/ansible/inventories/prod/hosts.yml#L89)

## VV-2026-10-106 — Documented deployment reviewer gates are not configured

**Observed:** Live infra has four *-plan environments and no *-apply environments. API has zero environments. Web production/staging have empty protection_rules, no branch policy and admin bypass enabled.

**Why it matters:** The documented independent deployment approval is not established. Missing credentials or runners may stop deployments, but are not an approval mechanism; no deployment bypass was executed.

**Repair sequence:** Provision the intended deployment environments with reviewed branch and reviewer policies using the authorized governance path. Verify service credential scope, and prevent workflows from silently relying on nonexistent protection.

**How a student can verify the repair:** Read live environment rules, test unauthorized branches and approval flow in nonproduction, and preserve results. No real apply is needed to prove configuration exists.

**Evidence:** [viavitae-infra/.github/workflows/deploy.yml:420](https://github.com/Via-Vitae/viavitae-infra/blob/a4145b540bf544a7300b167d10b6c3ed53674c2d/.github/workflows/deploy.yml#L420); [viavitae-api/.github/workflows/deploy.yml:4](https://github.com/Via-Vitae/viavitae-api/blob/328eb14830c7bb71c8f02b7eca36cf859308e55a/.github/workflows/deploy.yml#L4); [viavitae-web/.github/workflows/deploy.yml:97](https://github.com/Via-Vitae/viavitae-web/blob/ca4656378679bf1b5aaf38bb5ddcd61b2ec6fdde/.github/workflows/deploy.yml#L97)

## VV-2026-10-107 — Platform GitOps paths deploy placeholders rather than workloads

**Observed:** All six configured environment/component directories contain placeholder ConfigMaps. They do not reference the workload manifests in the application repositories.

**Why it matters:** Successful synchronization of these paths is not successful application deployment. This is a source readiness finding, not an assertion about every live cluster.

**Repair sequence:** Wire versioned application manifests through an approved release design, reconcile namespace/image conventions, and retain exactly one authoritative deployment path.

**How a student can verify the repair:** Render all six targets, confirm intended Deployment/Service/Ingress resources and immutable image references, then verify application readiness and rollback in staging.

**Evidence:** [viavitae-infra/k8s/argocd/applicationsets/platform.yaml:46](https://github.com/Via-Vitae/viavitae-infra/blob/a4145b540bf544a7300b167d10b6c3ed53674c2d/k8s/argocd/applicationsets/platform.yaml#L46); [viavitae-infra/k8s/platform/prod/api/placeholder.yaml:14](https://github.com/Via-Vitae/viavitae-infra/blob/a4145b540bf544a7300b167d10b6c3ed53674c2d/k8s/platform/prod/api/placeholder.yaml#L14); [viavitae-infra/k8s/platform/prod/web/placeholder.yaml:14](https://github.com/Via-Vitae/viavitae-infra/blob/a4145b540bf544a7300b167d10b6c3ed53674c2d/k8s/platform/prod/web/placeholder.yaml#L14)

## VV-2026-10-108 — Demo reset and seed verification are incomplete

**Observed:** The root verification command fails looking for reset.manifest.json at the repository root. All nine template manifests have zero hash entries. An explicit template verification exits 0 over zero entries. Reset prints wipe/restore messages without performing those operations.

**Why it matters:** A demo can retain stale state; reported integrity checks do not establish seed integrity.

**Repair sequence:** Make verification enumerate each template, require a complete nonempty inventory where seed files exist, reject missing/extra files, and implement transactional reset or explicitly disable it.

**How a student can verify the repair:** Corrupt/add/remove a seed file and require failure. Mutate synthetic demo state, reset it and compare against expected state. Keep provider credentials out of this test.

**Evidence:** [viavitae-demos/package.json:20](https://github.com/Via-Vitae/viavitae-demos/blob/1ec6e2fdb4a0cbcaf9234d53cc74f3170ffbc64f/package.json#L20); [viavitae-demos/reset/seed-hash.ts:108](https://github.com/Via-Vitae/viavitae-demos/blob/1ec6e2fdb4a0cbcaf9234d53cc74f3170ffbc64f/reset/seed-hash.ts#L108); [viavitae-demos/reset/reset.sh:100](https://github.com/Via-Vitae/viavitae-demos/blob/1ec6e2fdb4a0cbcaf9234d53cc74f3170ffbc64f/reset/reset.sh#L100); [viavitae-demos/templates/online-store/reset.manifest.json:1](https://github.com/Via-Vitae/viavitae-demos/blob/1ec6e2fdb4a0cbcaf9234d53cc74f3170ffbc64f/templates/online-store/reset.manifest.json#L1)

## VV-2026-10-109 — Hermes CI is not a mandatory merge barrier

**Observed:** Live branch protection has no required status contexts and the repository has no rulesets; ci and gitleaks can run without being merge prerequisites.

**Why it matters:** A failing invariant test is not a configured required merge check.

**Repair sequence:** Declare the actual emitted Hermes job names as required in the control plane, validate the plan and apply through authorized governance.

**How a student can verify the repair:** Create a harmless failing test on a review branch and verify merge is blocked; restore it and verify checks are required.

**Evidence:** [viavitae-control/repos_data.tf:221](https://github.com/Via-Vitae/viavitae-control/blob/b1b2e32a309f7624d91510510627b77b4c1c7c11/repos_data.tf#L221)

## VV-2026-10-110 — Independent code review is not enforced for active public repositories

**Observed:** 22 repositories report zero required approvals, including every nonempty public repository. Four empty public repositories report one. Five private repository protection queries are unavailable under the current plan. Org plan reports one filled seat.

**Why it matters:** A PR is an audit record, but does not by itself provide independent human review. This is a disclosed staffing/governance exception, not evidence of malicious activity.

**Repair sequence:** Assign an independent reviewer or formally document a time-limited compensating control. Enforce the chosen review policy through governance code when staffing supports it.

**How a student can verify the repair:** Show that author-only approval cannot satisfy the intended control and that emergency exceptions are logged and reviewed.

**Evidence:** [viavitae-control/modules/repo/branch_protection.tf:29](https://github.com/Via-Vitae/viavitae-control/blob/b1b2e32a309f7624d91510510627b77b4c1c7c11/modules/repo/branch_protection.tf#L29)

## VV-2026-10-111 — Private compliance repositories lack verified platform protection

**Observed:** All five private repos return a plan-upgrade error for branch protection and report native secret scanning/push protection disabled. The only nonempty private repository, compliance, contains one tracked file and no workflow/CODEOWNERS.

**Why it matters:** The same protection baseline as public code is not verified for private evidence. This is not proof that evidence has leaked or been altered.

**Repair sequence:** Keep sensitive evidence private. Obtain an appropriate platform capability or implement reviewed compensating access, integrity and scanning controls; do not make private records public to unlock features.

**How a student can verify the repair:** Demonstrate protected change history, access review, secret scanning and recovery for the actual private evidence location.

**Evidence:** see the named JSON/log artifacts and exact commands in findings.json.

## VV-2026-10-112 — Client consent checker accepts existence instead of evidence validity

**Observed:** The checker only checks file existence; it does not parse the referenced record. Both scripts exclude underscore-prefixed example folders. Current tenant validation exits 0 with No tenant.yaml files found.

**Why it matters:** A passing empty-scope run or a file placed at the expected path is not evidence of valid, approved processing decisions.

**Repair sequence:** Validate an explicit record schema and linkage to tenant/purpose/version/approver, expose the checked population count, and keep legal determination with the accountable privacy owner.

**How a student can verify the repair:** Test missing, empty, malformed, mismatched and expired records and report zero-scope checks as incomplete when real tenants are expected.

**Evidence:** [viavitae-clients/scripts/check_consent_dpias.py:45](https://github.com/Via-Vitae/viavitae-clients/blob/3a9db5f8ec835474062814a2d39af584daaddfc6/scripts/check_consent_dpias.py#L45); [viavitae-clients/scripts/validate_tenants.py:25](https://github.com/Via-Vitae/viavitae-clients/blob/3a9db5f8ec835474062814a2d39af584daaddfc6/scripts/validate_tenants.py#L25)

## VV-2026-10-113 — Control-plane state reconciliation could not be verified

**Observed:** The checkout contains no authoritative Terraform state or deployment variable file. Terraform 1.16.1 matches the pin; formatting, validation and guard tests pass. Live settings were checked directly, but a three-way state reconciliation was not possible.

**Why it matters:** No conclusion can be drawn about actual state encryption, locking, off-host backup, state drift or restoration from this checkout.

**Repair sequence:** Have the state custodian provide a sanitized state/resource inventory and witnessed backup/restore/locking evidence. Keep state contents and credentials out of public audit artifacts.

**How a student can verify the repair:** Reconcile declared resources, authoritative state addresses and live IDs; witness a safe nonproduction recovery procedure. Do not infer production drift from an empty local state.

**Evidence:** [viavitae-control/docs/COMPLIANCE.md:36](https://github.com/Via-Vitae/viavitae-control/blob/b1b2e32a309f7624d91510510627b77b4c1c7c11/docs/COMPLIANCE.md#L36)

## VV-2026-10-114 — Organization action pinning has exceptions

**Observed:** Org reports sha_pinning_required=false. Default-branch source sweep finds two tag-referenced actions in control and three in Hermes. Other inspected workflow uses are pinned or local.

**Why it matters:** Mutable upstream tags weaken reproducibility and can change executed code without a repository diff.

**Repair sequence:** Pin the remaining actions to verified commit revisions and align organization/repository enforcement with the intended policy. Review updates through the dependency process.

**How a student can verify the repair:** Static scan rejects a tag reference and passes immutable references; confirm organization action policy and required checks live.

**Evidence:** [viavitae-control/.github/workflows/audit.yml:34](https://github.com/Via-Vitae/viavitae-control/blob/b1b2e32a309f7624d91510510627b77b4c1c7c11/.github/workflows/audit.yml#L34); [viavitae-hermes-agents/.github/workflows/ci.yml:13](https://github.com/Via-Vitae/viavitae-hermes-agents/blob/0e35dccd290ffc81c02ba64902e4dd8d9c0a4847/.github/workflows/ci.yml#L13)

## Earlier API/Hermes findings still relevant to default branches

The earlier targeted audit identified incomplete authentication, persistent approval and retrieval implementations, heuristic classification, a caller-supplied lawful-basis marker rather than evidence verification, and in-memory audit storage. The draft fixes cover blank boundary settings, mismatched audit-chain lengths and a pastoral tenant gate; they do not implement all those missing controls. Five API skips and one expected failure still describe unfinished verification. A hash chain without independent persistence/anchors is not a complete audit system.

See the [earlier detailed report](../AUDIT-REPORT.md) for the tested fixes and remaining release blockers.

## Ordered remediation plan

1. **Make evidence truthful.** Replace unconditional tests; make missing prerequisites produce incomplete/failing release gates; make restore/probe/reset stubs refuse to claim success. Success means an intentionally broken control makes the gate fail.
2. **Triage dependencies.** Map each alert to locked version, execution path and exposure. Prioritize critical reachable cases and build inputs. Success means documented remediation or an approved, expiring exception for each high-priority alert.
3. **Reconcile product contracts.** Agree one assessment/quote specification and test the actual producer against the consumer. Success means a synthetic end-to-end request completes with the documented response.
4. **Finish identity and persistence.** Verify identity, tenant membership and resource permissions before storage/retrieval; implement version-bound approvals and transactionally safe payment replay handling. Success means negative tenant/replay/stale-approval tests fail closed.
5. **Make deployment reproducible.** Replace GitOps placeholders, align image/namespace conventions, configure reviewer gates and require the real CI checks. Success means an authorized staging release can be traced to an exact reviewed artifact and rolled back.
6. **Prove operational controls.** Rehearse restore, network boundaries, incident handling, retention/deletion and secret rotation using synthetic data. Success means actual measured results, not templates or echo statements.
7. **Resolve evidence ownership.** Identify the authoritative policy, vendor, privacy, training and state locations; obtain accountable decisions and access reviews. Do not assume dedicated empty repositories must be populated if an approved system already holds the records.
8. **Reconsider Qoder after these gates.** Begin with approved source/synthetic material and supervised development. Select SDK, cloud, remote access or QoderWake only for a concrete task and approved data flow. The [Qoder guide](../QODER-INTEGRATION-GUIDE.md) remains the proposed adoption path; no runtime was enabled.

## Assurance boundaries

SOC 2 assurance comes through an examination/reporting process; ISO/IEC 27001 concerns an information security management system. Code tests and GitHub check names do not establish either. Special-category processing requires additional privacy conditions; a source-code consent flag is not the determination. Consult the accountable privacy/security owners and independent assessors as appropriate. Sources: [AICPA SOC services](https://www.aicpa-cima.com/resources/landing/system-and-organization-controls-soc-suite-of-services), [ISO/IEC 27001](https://www.iso.org/standard/27001), [EDPB legal basis guidance](https://www.edpb.europa.eu/topics/key-gdpr-concepts/legal-basis_en).

No full-history secret audit, complete organization audit-log review, penetration test, live database isolation proof, contract/legal approval verification or production recovery drill was performed. These limitations must accompany any use of this report as evidence.

