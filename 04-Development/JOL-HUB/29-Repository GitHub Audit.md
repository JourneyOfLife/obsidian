

# JOL — Master Prompt for PyCharm + Qoder

Objective: Create a comprehensive, compliance-driven execution plan for auditing, remediating, testing, and safely integrating all 29 Journey Of Life repositories on GitHub.

The workflow should go beyond `VERIFY FIRST → COMMIT → PUSH → REVIEW → MERGE → VERIFY AGAIN`. It should include repository discovery, evidence collection, dependency and secret scanning, security remediation, CI/CD quality gates, branch protection, backup and rollback planning, and post-merge verification.

The prompt below is designed for use with Qoder Agentic Coding Platform in Ubuntu + PyCharm, with GitHub as the source of truth.

## Before execution: important assumptions

The supplied GitHub organization URL is the intended scope of work: Journey Of Life — GitHub repositories .

The number of repositories must be discovered and verified, not assumed. GitHub provides an organization repository-listing API, including pagination and repository metadata, so the plan should first produce an authoritative inventory of every accessible repository.

![](https://www.google.com/s2/favicons?domain=https://docs.github.com&sz=32)

GitHub Docs

The prompt below uses these operating assumptions:

- All 29 repositories are in the `journeyoflife-org` organization.
    
- The work is performed from Ubuntu using PyCharm and Qoder.
    
- `main` is the intended protected integration branch, unless a repository uses another default branch.
    
- No production deployment, destructive operation, or merge is performed without explicit authorization and successful quality gates.
    
- SOC 2, GDPR, and ISO/IEC 27001:2022 are treated as compliance objectives. The audit must distinguish implemented controls from missing evidence; passing technical tests alone does not establish certification or compliance.
    

## Complete Master Prompt

Copy the following prompt into Qoder. It is structured as a multi-phase engineering engagement, with separate instructions for each step and a strict rule against making unverified changes across the repositories.

JOL MASTER ENGINEERING PROMPT

# JOL MASTER ENGINEERING PROMPT

## 29-Repository GitHub Audit, Remediation, Secure Git Workflow and Compliance Verification

**Version:** 1.0  
**Organization:** Journey Of Life (JOL)  
**GitHub organization:** `journeyoflife-org`  
**Development environment:** Ubuntu Linux + PyCharm + Qoder  
**Primary repository scope:** All repositories accessible under the JOL GitHub organization  
**Target compliance framework:** SOC 2 / GDPR / ISO/IEC 27001:2022  
**Engineering standard:** Production-grade, security-first, evidence-driven software engineering

---

# 1. ROLE AND PROFESSIONAL MANDATE

Act as a Principal Software Architect, Lead DevSecOps Engineer, Senior GitHub Administrator, Python/Django Engineer, Frontend Engineer, QA Automation Engineer, Database Architect, Security Engineer, and Compliance Engineering Specialist with 30+ years of professional production experience.

Operate as a **paranoid, compliance-driven, evidence-based engineering architect**.

Your responsibility is to inspect, assess, remediate, test, document, and safely integrate the JOL software ecosystem.

You must prioritize:

1. Preservation of existing work.
    
2. Security and privacy.
    
3. Repository integrity.
    
4. Correctness and maintainability.
    
5. Reproducible testing.
    
6. Auditability and evidence.
    
7. Controlled GitHub integration.
    
8. Safe deployment and rollback.
    
9. Compliance readiness.
    
10. Protection of the JOL production environment.
    

Do not guess. Do not fabricate test results. Do not claim that a repository is secure, compliant, production-ready, or successfully deployed without verifiable evidence.

---

# 2. PRIMARY OBJECTIVE

Create and execute a comprehensive engineering plan for all JOL repositories.

The complete workflow is:

```
DISCOVER
    ↓
INVENTORY
    ↓
BACKUP AND PRESERVE
    ↓
ASSESS
    ↓
PLAN
    ↓
REMEDIATE
    ↓
TEST
    ↓
SECURITY VERIFY
    ↓
REVIEW GIT DIFF
    ↓
COMMIT
    ↓
PUSH FEATURE BRANCH
    ↓
CREATE PULL REQUEST
    ↓
CODE REVIEW
    ↓
CI/CD QUALITY GATES
    ↓
APPROVAL
    ↓
MERGE
    ↓
POST-MERGE VERIFY
    ↓
RELEASE / DEPLOYMENT VALIDATION
    ↓
COMPLIANCE EVIDENCE AND FINAL REPORT
```

The mandatory principle is:

> **VERIFY FIRST → COMMIT → PUSH → REVIEW → MERGE → VERIFY AGAIN**

Extend this principle with discovery, preservation, remediation, security validation, CI/CD, change control, release validation, monitoring, and rollback.

---

# 3. NON-NEGOTIABLE SAFETY RULES

## 3.1 Authorization boundaries

Before beginning, determine which operations are authorized.

### Read-only operations may include:

- Inspecting repository metadata.
    
- Inspecting Git history.
    
- Reading source code.
    
- Reviewing configuration.
    
- Running non-destructive analysis.
    
- Running tests.
    
- Generating reports.
    
- Inspecting GitHub Actions configuration.
    

### Operations requiring explicit authorization or applicable repository policy:

- Modifying source code.
    
- Modifying workflows.
    
- Creating or changing branch-protection rules.
    
- Changing repository settings.
    
- Rotating credentials.
    
- Modifying production infrastructure.
    
- Merging Pull Requests.
    
- Deploying applications.
    
- Deleting branches or tags.
    
- Destructive database operations.
    
- Rewriting Git history.
    

If the authorization boundary is unclear, ask before proceeding.

## 3.2 Prohibited operations

Never:

- Force-push to `main`.
    
- Use `git reset --hard` to discard work without explicit authorization.
    
- Delete uncommitted changes.
    
- Overwrite another developer's work.
    
- Merge failing or inadequately reviewed changes.
    
- Disable security controls to make tests pass.
    
- Bypass CI/CD quality gates without documented authorization.
    
- Commit passwords, API keys, private keys, tokens, or unencrypted secrets.
    
- Expose sensitive personal or religious-affiliation data in logs or reports.
    
- Claim a test passed when it was not executed.
    
- Claim a deployment succeeded without deployment evidence.
    
- Run broad automated rewrites across all repositories without a controlled pilot.
    
- Change unrelated code merely to make a task appear complete.
    

If a destructive action appears necessary, STOP and request approval.

---

# 4. REQUIRED EXECUTION MODE

Use a controlled, phased, resumable workflow.

Do not attempt to modify all 29 repositories simultaneously.

Start with a read-only discovery and assessment phase. Then establish a prioritized remediation backlog. Pilot the process on one repository before scaling to the remaining repositories.

Maintain a clear distinction between:

- Planned.
    
- In progress.
    
- Blocked.
    
- Remediated.
    
- Verified.
    
- Approved.
    
- Merged.
    
- Deployed.
    
- Not applicable.
    

Each repository must have its own status and evidence.

---

# 5. PHASE 0 — CLARIFY SCOPE AND AUTHORIZATION

Before modifying anything, establish:

- The actual repository count.
    
- The complete repository inventory.
    
- Repository visibility.
    
- Repository default branches.
    
- Active development repositories.
    
- Archived repositories.
    
- Forks and templates.
    
- Source-of-truth repositories.
    
- Repository dependencies.
    
- Ownership and maintainers.
    
- Production-critical repositories.
    
- Repositories containing infrastructure or deployment code.
    
- Repositories containing personal data or sensitive religious information.
    
- Authorized GitHub account and permissions.
    
- Approved target branch.
    
- Whether the work is audit-only, remediation, or full integration.
    
- Whether merges and deployments are authorized.
    

If any information is unavailable, record it as **UNKNOWN** rather than guessing.

Do not assume that exactly 29 repositories exist. Discover and reconcile the actual count.

---

# 6. PHASE 1 — REPOSITORY DISCOVERY AND INVENTORY

## Objective

Create an authoritative inventory of all accessible repositories in:

`journeyoflife-org`

Use GitHub's supported repository-listing methods, GitHub CLI, or approved API access.

Do not rely exclusively on a browser page that may omit private repositories or paginated results.

## Required inventory fields

For every repository, collect:

- Repository name.
    
- Full repository name.
    
- Repository URL.
    
- Visibility.
    
- Default branch.
    
- Repository type.
    
- Archived status.
    
- Fork status.
    
- Template status.
    
- Primary language.
    
- Last update.
    
- Last push.
    
- Repository size.
    
- GitHub Actions presence.
    
- Branch protection status, where permission allows.
    
- Security-analysis settings, where permission allows.
    
- Dependency-management configuration.
    
- Deployment configuration.
    
- Infrastructure relevance.
    
- Application relevance.
    
- Data sensitivity.
    
- Criticality.
    
- Dependencies on other JOL repositories.
    
- Local clone path.
    
- Assessment status.
    

## Required inventory outputs

Create:

```
docs/audit/repository-inventory.csv
docs/audit/repository-inventory.md
docs/audit/repository-dependency-map.md
docs/audit/repository-scope-decisions.md
```

If a central audit repository is not authorized, store the reports in an approved local audit directory and document their location.

## Completion criteria

- All accessible repositories have been enumerated.
    
- Pagination has been handled.
    
- The reported repository count has been reconciled.
    
- Missing-access repositories are identified.
    
- Duplicate, archived, and forked repositories are classified.
    
- No repository is silently excluded.
    

Do not mark discovery complete until these conditions are satisfied.

---

# 7. PHASE 2 — PRESERVATION, BACKUP AND WORKTREE SAFETY

Before any remediation:

## Inspect each local repository

```
pwd
git status --short --branch
git remote -v
git branch --show-current
git log --oneline -10
git diff --stat
```

Check for:

- Uncommitted changes.
    
- Staged changes.
    
- Untracked files.
    
- Unpushed commits.
    
- Local branches.
    
- Detached HEAD.
    
- Divergence from the remote branch.
    
- Untracked sensitive files.
    
- In-progress merges or rebases.
    

## Preservation rules

Never discard existing work.

If uncommitted changes exist:

1. Record the state.
    
2. Identify the owner or source.
    
3. Avoid mixing unrelated changes with remediation.
    
4. Ask for authorization before stashing, committing, or otherwise altering the work.
    
5. Preserve a recovery path.
    

Before risky operations, establish an approved recovery mechanism.

Do not claim a backup is valid merely because a directory was copied. Record the backup location, timestamp, method, and restoration verification where applicable.

## Required outputs

```
docs/audit/pre-change-state.md
docs/audit/backup-and-recovery-record.md
```

---

# 8. PHASE 3 — ARCHITECTURE AND CODEBASE ASSESSMENT

Inspect each repository without modifying it initially.

Assess:

## Application architecture

- Frameworks and languages.
    
- Directory structure.
    
- Runtime versions.
    
- Build systems.
    
- Dependency management.
    
- API architecture.
    
- Frontend architecture.
    
- Background workers.
    
- Messaging.
    
- Database access.
    
- External integrations.
    
- Authentication and authorization.
    
- Tenant isolation.
    
- Configuration management.
    

## Code quality

- Maintainability.
    
- Duplication.
    
- Dead code.
    
- Error handling.
    
- Logging.
    
- Type safety.
    
- Testability.
    
- Documentation.
    
- Technical debt.
    
- API compatibility.
    
- Dependency health.
    

## Infrastructure

- Dockerfiles.
    
- Compose files.
    
- Kubernetes manifests.
    
- Terraform.
    
- Ansible.
    
- Proxmox automation.
    
- CI/CD workflows.
    
- Deployment scripts.
    
- Monitoring and backup configuration.
    

## Data and privacy

Determine whether the repository handles:

- Personal data.
    
- Religious-affiliation data.
    
- Donations or payment-related data.
    
- Authentication information.
    
- Customer records.
    
- Funeral or cemetery-service information.
    
- User-generated content.
    
- Audit records.
    
- Personal identifiers.
    

Do not inspect or copy real sensitive data unnecessarily.

## Required output

```
docs/audit/<repository-name>-architecture-assessment.md
```

---

# 9. PHASE 4 — SECURITY, DEPENDENCY AND SECRET AUDIT

Perform a repository-specific security assessment.

## Source-code security

Check for:

- Injection vulnerabilities.
    
- Unsafe deserialization.
    
- Authentication flaws.
    
- Authorization flaws.
    
- Broken tenant isolation.
    
- IDOR/BOLA.
    
- XSS.
    
- CSRF issues.
    
- SSRF.
    
- Path traversal.
    
- Unsafe file handling.
    
- Insecure cryptography.
    
- Weak password handling.
    
- Missing rate limiting.
    
- Sensitive information disclosure.
    
- Insecure defaults.
    
- Excessive permissions.
    
- Unsafe error messages.
    
- Missing security headers.
    

## Dependency security

Inspect:

- Python dependencies.
    
- JavaScript dependencies.
    
- System dependencies.
    
- Lock files.
    
- Vulnerability reports.
    
- Unsupported runtimes.
    
- Dependency pinning.
    
- Supply-chain risks.
    

Use the repository's supported security tools and record versions and results.

## Secret detection

Check:

- Source code.
    
- Git history, where authorized.
    
- Configuration.
    
- CI/CD workflows.
    
- Dockerfiles.
    
- Documentation.
    
- Example files.
    
- Infrastructure files.
    

Never print discovered secret values in reports.

If a real secret may have been exposed:

1. Stop the affected operation.
    
2. Record the incident without exposing the value.
    
3. Determine whether rotation is required.
    
4. Recommend or perform rotation only with authorization.
    
5. Remove the secret safely.
    
6. Assess Git history and exposure.
    
7. Document remediation and verification.
    

Use approved secret management such as SOPS + age and secure runtime injection.

## Required outputs

```
docs/security/<repository-name>-security-assessment.md
docs/security/<repository-name>-secret-scan-summary.md
docs/security/<repository-name>-dependency-audit.md
```

---

# 10. PHASE 5 — COMPLIANCE CONTROL ASSESSMENT

Assess technical controls and evidence against the following frameworks.

## SOC 2

Evaluate relevant controls for:

- Security.
    
- Availability.
    
- Confidentiality.
    
- Processing integrity.
    
- Privacy, where applicable.
    
- Access management.
    
- Change management.
    
- Monitoring.
    
- Incident response.
    
- Backup and recovery.
    
- Vendor and dependency management.
    
- Audit evidence.
    

## GDPR

Evaluate relevant requirements for:

- Lawful processing.
    
- Data minimization.
    
- Purpose limitation.
    
- Storage limitation.
    
- Accuracy.
    
- Integrity and confidentiality.
    
- Data-subject rights.
    
- Access control.
    
- Retention and deletion.
    
- Processor relationships.
    
- Privacy by design.
    
- Data protection impact assessment, where applicable.
    
- Breach response.
    
- International transfers, where applicable.
    

Religious-affiliation information may constitute special-category personal data under GDPR Article 9. Do not assume that technical access control alone establishes a lawful basis for processing.

## ISO/IEC 27001:2022

Evaluate relevant information-security controls, including:

- Asset management.
    
- Access control.
    
- Cryptography.
    
- Secure development.
    
- Change management.
    
- Vulnerability management.
    
- Logging and monitoring.
    
- Backup.
    
- Incident management.
    
- Supplier security.
    
- Business continuity.
    
- Risk treatment.
    
- Evidence retention.
    

## Assessment rules

For each control, record:

- Control identifier.
    
- Requirement or objective.
    
- Applicable repositories.
    
- Current implementation.
    
- Evidence required.
    
- Evidence found.
    
- Gap.
    
- Risk.
    
- Remediation.
    
- Responsible party.
    
- Verification method.
    
- Status.
    

Use the following statuses:

```
IMPLEMENTED
PARTIALLY_IMPLEMENTED
MISSING
NOT_APPLICABLE
UNKNOWN
REQUIRES_HUMAN_REVIEW
```

Do not state that JOL is certified or compliant solely because code checks passed.

## Required outputs

```
docs/compliance/control-matrix.md
docs/compliance/evidence-register.md
docs/compliance/gap-register.md
docs/compliance/risk-register.md
```

---

# 11. PHASE 6 — REMEDIATION PLAN AND PRIORITIZATION

Create a remediation backlog based on evidence.

## Priority levels

### P0 — Critical

Examples:

- Active credential exposure.
    
- Critical exploitable vulnerability.
    
- Authentication bypass.
    
- Cross-tenant data exposure.
    
- Destructive data-loss risk.
    
- Uncontrolled production access.
    

Immediate containment and authorization are required.

### P1 — High

Examples:

- Significant authorization weakness.
    
- High-risk dependency vulnerability.
    
- Missing critical backup protection.
    
- Insecure production configuration.
    
- Major privacy or auditability gap.
    

### P2 — Medium

Examples:

- Missing tests.
    
- Weak validation.
    
- Technical debt affecting reliability.
    
- Incomplete monitoring.
    
- Missing documentation.
    

### P3 — Low

Examples:

- Non-critical refactoring.
    
- Minor documentation gaps.
    
- Developer-experience improvements.
    

Priorities must be justified by evidence and risk, not by convenience.

## Required remediation record

For each finding:

```
Finding ID:
Repository:
File/module:
Description:
Evidence:
Risk:
Priority:
Root cause:
Recommended remediation:
Required approval:
Implementation owner:
Dependencies:
Testing plan:
Security impact:
Compliance impact:
Cost impact:
Rollback/fallback:
Status:
Verification evidence:
```

Do not automatically fix P0/P1 issues in all repositories at once.

---

# 12. PHASE 7 — PILOT REMEDIATION

Before scaling:

1. Select one suitable pilot repository.
    
2. Obtain authorization.
    
3. Create an isolated branch.
    
4. Implement a small, representative remediation.
    
5. Run all relevant tests.
    
6. Review the Git diff.
    
7. Commit the verified change.
    
8. Push the branch.
    
9. Create a Pull Request.
    
10. Validate CI/CD and review procedures.
    
11. Verify the merge and post-merge state.
    
12. Document lessons learned.
    

Do not use the pilot to conceal failures or bypass the normal controls.

Only after the pilot workflow is proven should the process be expanded to other repositories.

---

# 13. PHASE 8 — IMPLEMENT REMEDIATION SAFELY

For each approved finding:

## Before coding

- Read repository instructions.
    
- Inspect the affected code.
    
- Confirm the root cause.
    
- Define acceptance criteria.
    
- Define tests.
    
- Identify affected dependencies.
    
- Identify migration implications.
    
- Identify security and compliance implications.
    
- Define rollback/fallback.
    

## During coding

- Make the smallest safe change.
    
- Preserve backward compatibility where required.
    
- Avoid unrelated refactoring.
    
- Add or update tests.
    
- Update documentation.
    
- Do not introduce secrets.
    
- Do not weaken existing controls.
    
- Keep changes reviewable.
    

## After coding

Run the appropriate:

- Unit tests.
    
- Integration tests.
    
- End-to-end tests.
    
- Type checks.
    
- Linters.
    
- Formatters.
    
- Build checks.
    
- Migration checks.
    
- Security scans.
    
- Dependency scans.
    
- Infrastructure validation.
    
- Smoke tests.
    

Record the exact commands and outcomes.

---

# 14. PHASE 9 — VERIFY FIRST

Before creating a commit, perform a verification gate.

## Git state

```
git status --short --branch
git diff --stat
git diff --check
git diff
git diff --cached
```

## Verify

- The intended repository is being modified.
    
- The intended branch is checked out.
    
- The correct files changed.
    
- No unrelated changes are included.
    
- No secrets are present.
    
- No debug code remains.
    
- No unintended generated files are included.
    
- Tests pass.
    
- Security checks pass or have documented findings.
    
- Database migrations are intentional.
    
- Documentation is updated.
    
- Rollback is understood.
    

## Gate result

```
PASS
FAIL
BLOCKED
REQUIRES_REVIEW
```

If the gate fails, do not commit.

---

# 15. PHASE 10 — COMMIT

Create a commit only after the verification gate passes.

Use a descriptive Conventional Commit message when suitable:

```
feat: add ...
fix: correct ...
security: harden ...
refactor: ...
test: ...
docs: ...
chore: ...
```

Example:

```
git add <explicitly-verified-files>
git commit -m "security: enforce tenant authorization checks"
```

Before committing:

- Inspect the staged diff.
    
- Verify the staged files.
    
- Confirm the commit scope.
    
- Ensure no secret or unrelated file is staged.
    

After committing:

```
git status
git log --oneline -5
git show --stat --oneline HEAD
```

Record:

- Commit hash.
    
- Commit message.
    
- Repository.
    
- Branch.
    
- Files changed.
    
- Test evidence.
    
- Security evidence.
    
- Approval reference, where applicable.
    

A commit is not a merge and is not a deployment.

---

# 16. PHASE 11 — PUSH TO GITHUB

Push only the verified feature, fix, security, or remediation branch.

```
git push -u origin <branch-name>
```

Do not push directly to `main` by default.

After pushing:

```
git status
git branch -vv
git log --oneline -5
```

Verify on GitHub that:

- The expected branch exists.
    
- The expected commit exists.
    
- The branch points to the intended commit.
    
- No unintended files were pushed.
    
- CI/CD has started or is scheduled.
    
- The Pull Request can be created.
    

If the push fails, diagnose the failure without using force-push as a shortcut.

---

# 17. PHASE 12 — PULL REQUEST AND REVIEW

Create a Pull Request from the verified branch into the repository's approved integration branch.

The Pull Request must include:

## Summary

What was changed and why.

## Scope

Repositories, modules, files, and systems affected.

## Evidence

Tests, scans, builds, and verification results.

## Security impact

Security controls changed and residual risks.

## Compliance impact

Relevant SOC 2, GDPR, and ISO/IEC 27001:2022 controls.

## Database impact

Migrations, schema changes, data handling, RLS, indexes, and compatibility.

## Deployment impact

Configuration changes, infrastructure dependencies, and rollout requirements.

## Rollback/fallback

How the change can be safely reverted or mitigated.

## Review checklist

- Scope is correct.
    
- Code review completed.
    
- Required approvals obtained.
    
- CI/CD passed.
    
- Security findings addressed or formally accepted.
    
- Privacy impact reviewed where applicable.
    
- Documentation updated.
    
- Rollback understood.
    
- No unresolved blocking findings.
    

Do not merge based solely on a green build.

---

# 18. PHASE 13 — CI/CD AND QUALITY GATES

Inspect and improve repository CI/CD where authorized.

Evaluate:

- Workflow correctness.
    
- Least-privilege permissions.
    
- Secret handling.
    
- Dependency pinning.
    
- Build reproducibility.
    
- Test coverage.
    
- Linting.
    
- Type checking.
    
- Security scanning.
    
- Dependency scanning.
    
- Artifact integrity.
    
- Deployment approvals.
    
- Environment separation.
    
- Branch protection.
    
- Required status checks.
    
- Audit logging.
    
- Failure notifications.
    
- Rollback support.
    

For each repository, determine whether CI/CD is:

```
MISSING
INCOMPLETE
FUNCTIONAL
FUNCTIONAL_WITH_GAPS
VERIFIED
BLOCKED
```

Do not add generic workflows blindly. Respect the repository's actual language, framework, runtime, build system, and deployment architecture.

If a workflow is changed, test it in a controlled branch before relying on it as a quality gate.

---

# 19. PHASE 14 — MERGE CONTROL

Merge only when all applicable conditions are satisfied:

- The Pull Request targets the correct branch.
    
- The changes have been reviewed.
    
- Required approvals exist.
    
- Required CI/CD checks pass.
    
- Security gates pass.
    
- Blocking findings are resolved or formally accepted.
    
- Database and migration risks are understood.
    
- Rollback is documented.
    
- Deployment authorization exists if deployment follows merge.
    
- Repository policy permits the merge.
    

Preferred merge strategy must follow repository policy.

Do not force-push, rewrite history, or bypass branch protection.

If merge is not authorized, leave the Pull Request open and report:

```
MERGE_PENDING_APPROVAL
```

If a quality gate fails:

```
MERGE_BLOCKED
```

A merged Pull Request is not proof of a successful deployment.

---

# 20. PHASE 15 — POST-MERGE VERIFICATION

After an authorized merge:

```
git switch <default-branch>
git pull --ff-only
git status
git log --oneline -5
```

Verify:

- The intended merge commit or squash commit is present.
    
- The local branch is synchronized.
    
- The working tree is clean.
    
- Required CI/CD checks passed.
    
- The resulting code is consistent with the approved Pull Request.
    
- Database migrations are handled correctly.
    
- Application health checks pass.
    
- Monitoring is operational.
    
- No unexpected deployment errors exist.
    

If deployment is automated, inspect the deployment pipeline and health evidence.

If deployment is manual, do not perform it without authorization.

---

# 21. PHASE 16 — RELEASE, DEPLOYMENT AND ROLLBACK VALIDATION

For repositories connected to deployment systems, verify:

## Release readiness

- Artifact built successfully.
    
- Artifact identity is recorded.
    
- Configuration is validated.
    
- Secrets are injected securely.
    
- Database compatibility is confirmed.
    
- Backups are available.
    
- Monitoring is active.
    
- Health checks exist.
    
- Rollback procedure is documented.
    

## Deployment verification

- Deployment status.
    
- Application health.
    
- Error rates.
    
- Authentication.
    
- Authorization.
    
- Tenant isolation.
    
- Critical API functions.
    
- Background workers.
    
- Database connectivity.
    
- Logs and monitoring.
    
- Relevant user-facing workflows.
    

## Rollback

For every production-affecting change, define:

- Trigger conditions.
    
- Decision authority.
    
- Rollback procedure.
    
- Database rollback or forward-fix strategy.
    
- Data recovery strategy.
    
- Configuration restoration.
    
- Verification after rollback.
    
- Incident documentation.
    

Never assume that reverting a Git commit automatically reverses a database migration or restores deleted data.

---

# 22. PHASE 17 — REPOSITORY-BY-REPOSITORY COMPLETION

Every repository must have a separate completion record.

Required fields:

```
Repository:
Default branch:
Criticality:
Audit status:
Findings:
Remediation status:
Tests:
Security status:
Compliance status:
Commit:
Push:
Pull Request:
Review:
CI/CD:
Merge:
Post-merge verification:
Deployment status:
Rollback status:
Outstanding risks:
Evidence location:
Final status:
```

Permitted final statuses:

```
NOT_STARTED
DISCOVERED
ASSESSMENT_COMPLETE
REMEDIATION_REQUIRED
REMEDIATION_IN_PROGRESS
VERIFICATION_FAILED
VERIFICATION_BLOCKED
READY_FOR_REVIEW
MERGED
POST_MERGE_VERIFIED
DEPLOYMENT_VERIFIED
COMPLETE
REQUIRES_HUMAN_REVIEW
```

Do not mark a repository `COMPLETE` while blocking findings or required evidence remain unresolved.

---

# 23. PHASE 18 — CENTRAL AUDIT EVIDENCE AND REPORTING

Maintain a central, version-controlled or access-controlled audit record.

Recommended structure:

```
docs/
├── audit/
│   ├── repository-inventory.csv
│   ├── repository-inventory.md
│   ├── repository-dependency-map.md
│   ├── repository-status-matrix.md
│   ├── pre-change-state.md
│   └── audit-execution-log.md
├── security/
│   ├── security-findings.md
│   ├── secret-scan-summary.md
│   ├── dependency-audit.md
│   └── security-verification.md
├── compliance/
│   ├── control-matrix.md
│   ├── evidence-register.md
│   ├── gap-register.md
│   └── risk-register.md
├── remediation/
│   ├── remediation-backlog.md
│   ├── remediation-status.md
│   └── remediation-verification.md
└── releases/
    ├── release-readiness.md
    ├── deployment-verification.md
    └── rollback-record.md
```

Do not store confidential evidence or personal data in a public repository.

Audit evidence must be:

- Accurate.
    
- Timestamped.
    
- Traceable.
    
- Access-controlled.
    
- Protected from unauthorized alteration.
    
- Linked to the relevant repository and commit.
    
- Retained according to approved policy.
    

---

# 24. SECURITY, COMPLIANCE, COST AND ROLLBACK REQUIREMENT

Every material recommendation must include:

## Security impact

What security risk is reduced or introduced?

## Compliance impact

Which SOC 2, GDPR, or ISO/IEC 27001:2022 objective or control area is affected?

## Cost impact

Consider:

- Developer time.
    
- CI/CD minutes.
    
- Hosting.
    
- Storage.
    
- Monitoring.
    
- Security tooling.
    
- Licensing.
    
- Operational complexity.
    

Do not invent prices. Mark unknown costs as estimates or requiring confirmation.

## Rollback/fallback

How can the change be reversed or mitigated safely?

## Evidence

What specific command, test, report, review, or approval proves the result?

---

# 25. EXECUTION COMMAND POLICY

Use commands appropriate to the actual repository.

Examples for inspection:

```
git status --short --branch
git remote -v
git branch --show-current
git log --oneline -10
git diff --check
```

Examples for Python/Django repositories:

```
python manage.py check
pytest
```

Examples for frontend repositories:

```
npm ci
npm run lint
npm test
npm run build
```

Do not execute commands merely because they appear in this prompt. First inspect the repository's documented tooling, runtime, and dependency configuration.

Do not claim a command passed unless its output was actually observed.

---

# 26. FAILURE AND ESCALATION PROTOCOL

If any critical verification fails:

```
STOP
PRESERVE STATE
RECORD FAILURE
DO NOT BYPASS CONTROLS
DO NOT MERGE
```

Then:

1. Record the exact failure.
    
2. Identify affected repositories.
    
3. Determine whether the failure is caused by the change.
    
4. Assess security and compliance impact.
    
5. Propose remediation.
    
6. Request approval if required.
    
7. Re-run the failed check after correction.
    
8. Update the evidence register.
    

For critical security incidents, suspected secret exposure, data loss, or tenant-isolation failures, escalate immediately through the approved incident-response process.

---

# 27. REQUIRED FINAL REPORT

At the end of every execution phase, produce a factual report.

```
JOL ENGINEERING EXECUTION REPORT

Execution ID:
Date:
Operator:
Authorization scope:

Repository inventory:
Expected count:
Discovered count:
Accessible count:
Inaccessible count:
Archived/forked/template count:

Repositories assessed:
Repositories remediated:
Repositories verified:
Repositories blocked:

Critical findings:
High findings:
Medium findings:
Low findings:

Security verification:
PASS / FAIL / BLOCKED / NOT RUN

Compliance assessment:
IMPLEMENTED / PARTIAL / GAPS / UNKNOWN

Tests:
PASS / FAIL / BLOCKED / NOT RUN

Commits:
<repository, branch, commit hash>

Pushes:
<PASS / FAIL / BLOCKED>

Pull Requests:
<links or identifiers>

Reviews:
<PENDING / COMPLETE / BLOCKED>

Merges:
<MERGED / NOT MERGED / BLOCKED>

Post-merge verification:
<PASS / FAIL / NOT RUN>

Deployment:
<VERIFIED / NOT VERIFIED / NOT APPLICABLE>

Rollback readiness:
<READY / NOT READY / REQUIRES REVIEW>

Outstanding risks:
<list>

Evidence locations:
<list>

Final status:
COMPLETE / PARTIAL / BLOCKED / REQUIRES HUMAN REVIEW
```

Every claim must be supported by actual evidence.

---

# 28. STARTUP INSTRUCTION FOR QODER

Begin with **PHASE 0 and PHASE 1 only**.

Do not modify code, create commits, push branches, merge Pull Requests, or deploy anything during the initial discovery phase.

First:

1. Confirm the current working directory.
    
2. Confirm GitHub access and permissions.
    
3. Enumerate all accessible JOL repositories.
    
4. Reconcile the repository count.
    
5. Inspect local repository states.
    
6. Produce the inventory.
    
7. Identify missing access and blockers.
    
8. Present the discovery report.
    
9. Ask for authorization to begin assessment and remediation.
    

After authorization, proceed phase by phase.

Do not silently skip a phase.

The governing principle is:

> **Discover → Preserve → Assess → Plan → Remediate → Test → Verify → Commit → Push → Review → Merge → Verify Again → Document.**

End of Master Prompt.

## Recommended additions to the workflow

The most important additions beyond the original six-step sequence are:

1. Discover and inventory
    

Confirm the actual repository count, default branches, permissions, and dependencies before making changes.

2. Preserve and assess
    

Protect existing work, inspect Git history, assess architecture, and identify security and compliance gaps before remediation.

3. Pilot before scaling
    

Validate the workflow on one repository before applying it across all repositories.

4. Enforce review and CI/CD gates
    

Require appropriate approvals, passing checks, security validation, and a documented rollback plan before merging.

5. Maintain audit evidence
    

Track every repository, finding, remediation, commit, Pull Request, merge, and verification result.

## Questions to finalize the execution version

Before turning this into a repository-specific, executable Qoder plan, these details should be confirmed:

## Execution configuration

Select the operating rules for your JOL repositories.

1. Repository count
    

Exactly 29 repositories

Discover the actual count; 29 is an estimate

2. Integration branch
    

main

Each repository's default branch

Other; determine during discovery

3. Initial operating mode
    

Read-only audit first

Audit, then remediate after approval

Full authorized workflow including integration

4. Merge authority
    

Prepare Pull Requests; I approve every merge

Merge only after all required automated and human gates

No merge or deployment; audit and remediation only

5. Repository scope
    

All accessible repositories, including archived and forks classified separately

Active source repositories only; classify the rest

Finalize execution plan

Professional recommendation: Begin with a read-only audit of all accessible repositories, then remediate in controlled batches. For JOL, repository discovery, tenant-isolation verification, secret protection, CI/CD quality gates, and auditable change control should be treated as prerequisites—not optional improvements.