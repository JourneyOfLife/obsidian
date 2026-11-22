




I'll load the brainstorming process skill and pull the repo's existing audit/prompt context in parallel.

Good context. Let me verify the repo counts and locate the hermes-agents/MCP assets before drafting.

Three orgs each have a `hermes-agents` repo, and jol has `jol-mcp-servers`. Let me inspect those assets and the declared repo inventory.

Verified enough to draft. Key corrections before the prompt:

- **Via-Vitae 31** ✓ consistent (`.github` + 30 `viavitae-*` local clones; matches the merged 2026-10-26 audit).
- **jolarca-dev 17** ✓ consistent (`.github` + 16 `jolarca-*`).
- **journeyoflife-org 41** ✗ **unverified** — local tree has `.github` + 41 `jol-*` = 42, plus non-org `obsidian/` and `.wt/`. Your counting rule is undefined.
- **89 is the wrong effort denominator.** 9 Via-Vitae repos are empty and 10 are landing scaffolds. Tiering beats parallelism.
- **Real blocker to "5 in parallel × 3 orgs":** 15 agents sharing one `gh` keyring account and one local Terraform state dir will collide. That must be designed around, not discovered.

Here is the optimized prompt asset, in the house style of `docs/superpowers/prompts/2026-10-19-…`:

````markdown
# Multi-Org Parallel Pre-Deployment Independent Audit — Reusable Prompt (v1)

| Field | Value |
| --- | --- |
| Created | 2026-10-26 |
| Revision | 1 |
| Repo | viavitae-control |
| Type | Prompt asset (not a policy, not a plan — ungated by CI) |
| Scope | `Via-Vitae` (31), `journeyoflife-org` (41 claimed), `jolarca-dev` (17) — 89 claimed total |
| Frameworks | SOC 2 Type II, ISO/IEC 27001:2022, GDPR; PCI-DSS 4.0 (`jolarca-dev` only) |
| Operator model | Single owner, single programmer, no teams, no separation of duties |
| Baseline verified | 2026-10-26, host `jol`: three control-plane trees, local clone census of all three `/opt/*/repos` trees, `viavitae-control/repos_data.tf`, `policies/AUDIT_PROMPT.md`, `docs/security/pre-deployment-independent-audit-2026-10-26.md` |
| Supersedes | Nothing. Composes `policies/AUDIT_PROMPT.md` (Gates 0–5, single-org) and `docs/superpowers/prompts/2026-10-19-multi-org-secrets-and-state-architecture-prompt.md` (credential/state opinion). |

## What this file is

A copy-paste prompt that turns a **one-repo-per-day serial audit** into a **tiered,
parallel, evidence-gated production line** across three legally separate GitHub
organizations, without weakening a single control.

It exists because the naive version of this request — *"run 89 audits, 5 at a time"* —
produces three failure modes at once:

1. **Wrong denominator.** An empty repo and a payments service get the same budget.
2. **Credential and state collision.** Parallel agents share one `gh` keyring account
   and one local Terraform state directory. They corrupt each other's evidence and
   trip GitHub secondary rate limits.
3. **Unmergeable output.** Fifteen prose reports that one human must read serially.
   The bottleneck moves from generation to review, and total throughput does not improve.

This prompt attacks all three: it forces a **tiering verdict**, a **concurrency verdict
with an isolation design**, and a **machine-readable output contract** so results merge
by script instead of by reading.

## What it asks for that the existing assets do not

| # | New demand | Why |
| --- | --- | --- |
| 1 | A **counting rule** (§5) and four numbers per org, never one | `.github`, forks, archived and empty repos are counted inconsistently across the three planes |
| 2 | A **T0–T4 tiering of all 89 repos** with per-tier depth and minutes (§6) | Effort is driven by tier, not by repo count |
| 3 | A **two-phase split**: bulk org sweep once, then parallel per-repo deep audits (§7) | Prior runs re-derived identical org-wide evidence inside every per-repo session |
| 4 | A **concurrency verdict** bounded by human review capacity, not agent count (§8.3) | 15 parallel audits → 15 reports → 1 reviewer. Review is serial. |
| 5 | **Per-org isolation** of worktree, credential, MCP root and evidence dir (§8.4–§8.5) | Three controllers; boundary 1 forbids cross-entity transit |
| 6 | A **concrete MCP + hermes-agents configuration** reconciled to files that exist (§9) | "Configure the MCP server" is not actionable until servers, roots, scopes and write-capability are named |
| 7 | A **tool-self-test gate** before any BLOCKED cell (§10) | A jq path bug once understated 20 critical Dependabot alerts and made a report too lenient |
| 8 | **Resumability** via run-id and manifest (§8.7) | A solo operator's day gets interrupted; restart-from-zero is the real cost |

## How to use

a. Re-verify §4 with the refresh commands below. Facts drift; a stale baseline poisons every verdict.
b. Copy everything between the `~~~~` fences.
c. Send to a top-tier model **with file and shell access**. If the channel has no file access, run the refresh commands yourself and paste the output into §4 in place of the recorded values.
d. Expect the reply to **correct your repo counts**. If it accepts 31/41/17/89 without re-deriving them, it failed §5 and you must re-run it.
e. Land the reply as a plan under `docs/superpowers/plans/` using `TEMPLATE.md`, on a feature branch, with an `ai:top` provenance trailer in both the commit body and the PR body.
f. Do **not** let a cheap tier execute §7–§9. Tier routing is a compliance control (`policies/AI_WORKFLOW_POLICY.md`).

### Refresh commands

All read-only. All write to uniquely-named files under `/tmp` — empty stdout has
previously produced false "no findings" conclusions on this host.

```bash
RUN=$(date -u +%Y%m%dT%H%M%SZ); OUT=/tmp/audit-baseline-$RUN; mkdir -p "$OUT"; echo "$OUT"

# 1. Live repo census per org — the ONLY authoritative count. HTML pages are not evidence.
for org in Via-Vitae journeyoflife-org jolarca-dev; do
  gh api "orgs/$org/repos?per_page=100" --paginate \
    --jq '.[] | [.name,.fork,.archived,.size,.visibility,.is_template,.default_branch] | @tsv' \
    > "$OUT/$org-repos.tsv" 2> "$OUT/$org-repos.err"
  printf '%-20s total=%s fork=%s archived=%s empty=%s private=%s\n' "$org" \
    "$(wc -l < "$OUT/$org-repos.tsv")" \
    "$(awk -F'\t' '$2=="true"' "$OUT/$org-repos.tsv" | wc -l)" \
    "$(awk -F'\t' '$3=="true"' "$OUT/$org-repos.tsv" | wc -l)" \
    "$(awk -F'\t' '$4==0'    "$OUT/$org-repos.tsv" | wc -l)" \
    "$(awk -F'\t' '$5=="private"' "$OUT/$org-repos.tsv" | wc -l)"
done

# 2. Local clone census — reveals repos with no working copy (cannot be deep-audited offline).
for d in /opt/viavitae/repos /opt/jol/repos /opt/jolarca/repos; do
  printf '%s: ' "$d"; ls -1A "$d" | tr '\n' ' '; echo
done

# 3. Declared inventory vs live, per control plane.
for d in /opt/viavitae/repos/viavitae-control /opt/jol/repos/jol-control /opt/jolarca/repos/jolarca-control; do
  printf '%-45s exists=%s state=%s\n' "$d" "$(test -d "$d" && echo Y || echo N)" \
    "$(stat -c%s "$d/terraform.tfstate" 2>/dev/null || echo NA)"
done

# 4. Agent and MCP assets that actually exist.
ls -1 /opt/jol/repos/jol-mcp-servers/servers/     > "$OUT/mcp-servers.txt"      2>&1
ls -1 /opt/jol/repos/jol-hermes-agents/           > "$OUT/jol-hermes.txt"        2>&1
ls -1 /opt/jolarca/repos/jolarca-hermes-agents/agents/ > "$OUT/jolarca-agents.txt" 2>&1
ls -1 /opt/viavitae/repos/viavitae-hermes-agents/config/ > "$OUT/vv-hermes-config.txt" 2>&1

# 5. Credential and identity context (names and scopes only — never values).
gh auth status                                    > "$OUT/gh-auth.txt" 2>&1
command -v git worktree && git -C /opt/viavitae/repos/viavitae-control worktree list > "$OUT/worktrees.txt" 2>&1

# 6. Prior audit artifacts to compose with, not duplicate.
ls -1 /opt/viavitae/repos/viavitae-control/docs/security/ > "$OUT/prior-audits.txt" 2>&1
```

---

## THE PROMPT

~~~~
# 1. ROLE AND STANCE

You are a **paranoid, compliance-driven security and infrastructure architect** with
30+ years of operational experience, retained to give a **binding professional opinion**
— not a survey of options. You have personally cleaned up after credential leaks, lost
Terraform state, and single-operator bus-factor incidents.

You are wearing **two hats at once**, and you must not drop either:

- **Hat A — Independent auditor.** You re-derive every fact from Terraform state and the
  live GitHub API. **Documentation is a claim, not evidence.** You do not trust
  `README.md`, `docs/COMPLIANCE.md`, a prior audit report, or the operator's own numbers.
- **Hat B — Throughput architect.** You are separately accountable for making 89
  repository audits finish in days rather than months, **without weakening a control to
  get the speed**. Speed obtained by skipping evidence is not speed; it is deferred
  incident cost. Say so plainly if the operator's proposed parallelism is unsafe.

**Teaching register (mandatory).** The reader is a competent student: fluent with Linux,
git and Terraform, but has never run a multi-org audit or configured an MCP server.
Every phase you emit must use this **8-element block**, in this order:

| # | Element | What it must contain |
| --- | --- | --- |
| 1 | **Purpose** | One sentence: what this phase establishes. |
| 2 | **Why it matters** | The framework clause it satisfies, cited by name (e.g. SOC 2 CC7.2, ISO A.8.4, GDPR Art. 32). |
| 3 | **Preconditions** | What must already be true; the command that proves it. |
| 4 | **Commands** | Copy-pasteable, each prefixed with the directory it runs from. |
| 5 | **Expected output** | The shape of correct output, so the student can recognise wrong output. |
| 6 | **Decision rule** | If output = X → conclusion Y. No ambiguity, no "it depends". |
| 7 | **Failure modes** | The specific ways this step silently lies. Named, not generic. |
| 8 | **Artifact + finding** | The file written, and the finding emitted if the gate fails. |

Define every acronym and technical term at first use. Never skip a "why". Prefer a table
over a paragraph wherever three or more items are comparable. **Length is not a virtue;
completeness and correctness are.**

# 2. COMPLIANCE TARGETS

Map every recommendation onto these frameworks, citing the clause by name each time:

- **SOC 2 Type II** — CC2.1/CC2.2 (information and communication), CC3.2 (risk assessment),
  CC4.1 (monitoring), CC5.2/CC5.3 (control deployment and segregation), CC6.1 (logical
  access), CC6.6 (external threats), CC7.1/CC7.2 (vulnerability detection and monitoring),
  CC8.1 (change management), CC9.2 (vendor risk), A1.2/A1.3 (backup and recovery).
- **ISO/IEC 27001:2022 Annex A** — A.5.15–A.5.18 (access control, identity, authentication
  information, access rights), A.5.24/A.5.25 (incident management planning), A.5.30 (ICT
  continuity), A.5.34 (privacy/PII), A.5.36 (compliance with IS policies), A.8.2 (privileged
  access), A.8.4 (source code), A.8.8 (technical vulnerabilities), A.8.15 (logging),
  A.8.24 (cryptography), A.8.32 (change management).
- **GDPR** — three controllers, one per legal entity. Art. 5(1)(f) integrity and
  confidentiality, Art. 5(1)(c) minimisation, Art. 24 responsibility, Art. 28 processor
  contracts, Art. 30 records of processing, Art. 32 security of processing, Art. 33/34
  72-hour breach notification, Art. 44–50 international transfer.
- **PCI-DSS 4.0** — `jolarca-dev` only. Req. 3 (stored account data), Req. 6.3/6.4 (vulnerability
  and patch management), Req. 7/8 (access control and authentication), Req. 10 (logging),
  Req. 11.3 (scanning), Req. 12.10 (incident response).

# 3. HARD CONSTRAINTS

1. **One human being operates all three planes.** No teams, no second approver, no on-call,
   no separation of duties. Every control that normally needs two people must be replaced
   by a **named compensating control one person can execute**, or declared unachievable
   with the residual risk recorded. Do not weaken a control because there is no team —
   add the compensating control instead.
2. **Three legal entities are three controllers.** `Via-Vitae` is a commercial IT company;
   `journeyoflife-org` is a Catholic charity whose stated principle is "built and operated
   in-house on hardware it owns"; `jolarca-dev` is a marketplace platform inside PCI-DSS
   scope. Data, credentials, tokens, evidence files, agent context and MCP filesystem
   roots from one **must not** transit or rest in infrastructure controlled by another.
   Treat cross-entity leakage as a **reportable incident**, not a tidy-up task.
   `jolarca-control/docs/security/isolation-model.md` defines this as boundary 1; it
   currently covers only the `jolarca-dev` ↔ `journeyoflife-org` pair, which is an open gap.
3. **Read-only / plan-only execution authority.** You may run `terraform plan`,
   `terraform validate`, `terraform fmt -check`, `terraform providers schema -json`,
   `gh api` **GET only**, and read-only git inspection. You may **NOT** run
   `terraform apply`, `destroy`, `import`, `init -migrate-state`, any `gh api`
   PATCH/PUT/POST/DELETE, `git push` to `main`, any visibility change, membership change,
   2FA change, or history rewrite. **Every mutating step you emit must carry the printed
   line:** `STOP — human approval required before this step`. A runbook with ungated
   mutating steps is a contract violation.
4. **Halt on destroy or replace.** Any step whose plan output would destroy or replace a
   resource stops at the plan and escalates, with the blast radius stated.
5. **No hedging.** Never write "consider using", "you might want to", "one option is",
   "it depends". Every question ends in **one verdict**, a rationale, and an explicit
   **rejection record** for each alternative you discarded.
6. **Never invent an identifier.** No fabricated repo name, count, PAT id, workspace id,
   ruleset id, alert number, file path or clause. If you did not read it, it goes in
   §12 "What I could not determine".
7. **Never write secret material or personal data into an artifact.** Reference by
   location and redacted placeholder, never by value. This includes email local-parts,
   which have previously leaked into a public repo's immutable history.
8. **Provenance and plan-as-contract.** Any commit produced from your advice carries the
   tier-provenance trailer required by `policies/AI_WORKFLOW_POLICY.md` (`ai:top`,
   `ai:mid`, `ai:cheap`) as a **standalone line in the commit body** — the commit-msg hook
   rejects tiers inside the subject — and the same trailer appears in the PR body for the
   server-side gate. The written plan is a contract: if execution deviates, work **halts
   for review** instead of improvising, including when a "better" approach is found.
9. **Cheap tiers must not reason about compliance adequacy.** Visibility changes,
   `manage_files` flips, ruleset edits, tier assignment of a repo, and GO/NO-GO verdicts
   are reserved for a human or a top tier. When a cheap tier fails the same step twice,
   work stops and escalates — it does not loop.
10. **Verify by command output, never by agent claim.** An agent asserting "all repos are
    protected" is not evidence. A file containing the API response is.
11. **Rate-limit discipline is a safety control, not a nicety.** Parallel sweeps against
    one org with one token trip GitHub's **secondary** rate limits, which return `403`
    with a body that `gh api` prints to **stdout** — silently corrupting evidence columns.
    Your design must state a concurrency ceiling, honour `Retry-After` /
    `X-RateLimit-Reset`, and fail loudly rather than record an empty value.
12. **Evidence immutability.** Each run gets a unique `run-id`. Evidence files are written
    once and never edited in place. A corrected finding is a **new** finding that
    retracts the old one, with the retraction logged.

# 4. VERIFIED ENVIRONMENT BASELINE (2026-10-26, host `jol`, Ubuntu 24.04)

**Correct any line below that the live tree contradicts, and say so loudly.** Do not
silently accept a stale fact, and do not silently correct one either — report the
contradiction, then continue on the corrected value.

## 4.1 The three planes

| GitHub org | Legal entity | Frameworks | Control-plane repo | Local path | State backend |
| --- | --- | --- | --- | --- | --- |
| `Via-Vitae` | Commercial IT company | SOC 2, ISO 27001, GDPR | `viavitae-control` | `/opt/viavitae/repos/viavitae-control` | **local** `terraform.tfstate`, mode 0600 |
| `journeyoflife-org` | Catholic charity, owns its hardware | SOC 2, ISO 27001, GDPR | `jol-control` | `/opt/jol/repos/jol-control` | **local** (re-verify; previously 195,121 bytes, mode 0600) |
| `jolarca-dev` | Marketplace platform | + PCI-DSS 4.0 | `jolarca-control` | `/opt/jolarca/repos/jolarca-control` | **HCP Terraform** (org `jolarca-dev`, ws `jolarca-control`, free tier <500 resources; ADR-0006 rejected "S3 + DynamoDB") |

## 4.2 Repo counts — the operator's claim, and its problems

| Org | Operator's claim | Local clone census 2026-10-26 | Status |
| --- | --- | --- | --- |
| `Via-Vitae` | 31 | `.github` + 30 `viavitae-*` = **31** | **CONSISTENT** — matches `repos_data.tf` (31 declared, zero drift both directions) and the merged 2026-10-26 audit |
| `journeyoflife-org` | 41 | `.github` + 41 `jol-*` = **42**, plus non-org `obsidian/` and `.wt/` | **INCONSISTENT — off by one or more. Counting rule undefined.** |
| `jolarca-dev` | 17 | `.github` + 16 `jolarca-*` = **17** | **CONSISTENT** |
| Total | 89 | — | **Only holds if `.github` counts in every org and no fork/archived repo exists. Unproven.** |

The operator also supplied six **HTML URLs** (org profile pages and `/repositories`
listing pages). Give your professional opinion on whether those belong in an audit input
at all. Note: an org *profile* page is not the org *settings* surface, and an HTML repo
listing is paginated, cached, and omits private repos the viewer cannot see.

## 4.3 Composition of the Via-Vitae 31 (from the merged 2026-10-26 audit)

- **9 of 31 repos are empty** — including every compliance-artifact repo
  (`viavitae-data-governance`, `viavitae-vendor-register`, `viavitae-policies`,
  `viavitae-threat-model`, `viavitae-compliance`) that application code **cites**
  (e.g. `DPIA-001`). A cited-but-empty artifact repo is a higher-severity finding than an
  unprotected branch, because the citation makes the control *look* satisfied.
- **10 `viavitae-landing-*` repos are template scaffolds** (~19 files each, a PyCharm
  `main.py` stub). The operator states they are not fully completed.
- **5 repos are private** (`viavitae-compliance`, `-data-governance`, `-policies`,
  `-vendor-register`, `-threat-model`) — staged privatization of 2026-09-24. On **GitHub
  Free**, private repos lose secret scanning, and branch protection requires Team+.
  These 5 returned `403` on several security endpoints in the prior run.
- **Protection coverage is 21 of 31**, not 31 of 31.
- **`viavitae-api` CI has been red on `main` since 2026-10-15** ("Build and push image").
- **All production k8s manifests in `viavitae-infra` are `placeholder.yaml`.**
- **Dependency posture (corrected after a jq-path retraction):** 20 critical / 63 high
  open Dependabot alerts on default branches (`api` 8/23, `brand` 5/23, `web` 5/9,
  `docs` 2/3).
- Org-level: 2FA enforced; Actions allow-list present; `sha_pinning_required = false`.
- **Prior verdict: NO-GO**, findings `VV-2026-10-001..010+`, merged as PR #39
  (squash commit `905d0e52`, 2026-10-26). 15 repos were left **NOT-RUN** for deep audit.

## 4.4 Agent and MCP assets that exist today (verify, do not assume)

| Asset | Path | Observed contents |
| --- | --- | --- |
| `viavitae-hermes-agents` | `/opt/viavitae/repos/viavitae-hermes-agents` | `uv`/`pyproject.toml`, `config/` (6), `src/`, `tests/`, `evaluations/`, `docs/`, `QODER.md`, `.gitleaks.toml`, pre-commit |
| `jol-hermes-agents` | `/opt/jol/repos/jol-hermes-agents` | 40 entries |
| `jolarca-hermes-agents` | `/opt/jolarca/repos/jolarca-hermes-agents` | `agents/` (12), `schemas/`, `policies/`, `evaluations/`, `scripts/` (17), `tests/` (17), `Makefile`, `qodana.yaml`, `QODER.md` |
| `jol-mcp-servers` | `/opt/jol/repos/jol-mcp-servers` | `servers/` (6), `shared/` (7), `inventory/`, `Makefile`, `qodana.yaml`, `.secrets.baseline`, `.bandit-report.json` |
| `jol-rag-server`, `jol-llm` | `/opt/jol/repos/…` | retrieval and model-serving components |

**Read these trees before proposing configuration.** Do not invent MCP servers, agent
names or tool names that are not in them.

## 4.5 Existing audit assets you must COMPOSE with, not duplicate

| Asset | Path | What it already gives you |
| --- | --- | --- |
| Evidence-gated audit prompt | `viavitae-control/policies/AUDIT_PROMPT.md` | Gates 0–5, findings JSON schema (§6), P0/P1/P2 severity rubric (§7), anti-patterns (§8) |
| AI workflow policy | `viavitae-control/policies/AI_WORKFLOW_POLICY.md` | 15 levers, tier routing, mandatory §5 review checklist |
| Plan template | `viavitae-control/docs/superpowers/plans/TEMPLATE.md` | STOP conditions, compliance-rationale table, task blocks |
| Landed audit report | `viavitae-control/docs/security/pre-deployment-independent-audit-2026-10-26.md` | 29 sections: PART I method (13 phases), PART II findings/evidence registers, PART III checklist + glossary |
| Per-repo audit plan | `viavitae-control/docs/superpowers/plans/2026-09-27-…-per-repo-audit.md` | D1–D9 matrix, 12 dimensions, 5 strata, 12-gate go/no-go, validated bash harnesses |
| Control register | `viavitae-control/docs/COMPLIANCE.md` | five-value status vocabulary: `enforced-verified` / `declared-not-applied` / `deferred-empty-repo` / `gap-plan-tier` / `out-of-band` |
| Prior prompt exemplar | `docs/superpowers/prompts/2026-10-19-multi-org-secrets-and-state-architecture-prompt.md` | credential surfaces A/B/C, state backend opinion, expiry cluster |

Reusing an existing schema, ID scheme or status vocabulary is **mandatory**. Inventing a
parallel one is a finding against you.

## 4.6 Known tool traps on this host (each one has already caused a wrong conclusion)

| # | Trap | Consequence if ignored |
| --- | --- | --- |
| T1 | `gh api` prints HTTP **error bodies to stdout** | `var=$(cmd \|\| echo FALLBACK)` concatenates JSON garbage into evidence columns. Use if-capture. |
| T2 | Dependabot severity lives at `.security_advisory.severity`, **not** `.severity` | A jq path bug once read as "severities unreadable / access limited", understating 20 critical alerts and making the report **too lenient**. |
| T3 | Code scanning endpoint is `/code-scanning/alerts`, **not** `/code-scanning/sarifs` | False "code scanning absent" finding. |
| T4 | Unborn HEAD: `git rev-parse --abbrev-ref HEAD` prints `HEAD` **and** exits non-zero | `\|\| echo UNKNOWN` yields a two-line value that corrupts a TSV row. Probe with `rev-parse --verify -q HEAD` first. |
| T5 | Terminal stdout is frequently **empty** on this host | Negative results get read as "no findings". Always redirect to a uniquely-named file and read the file back. |
| T6 | `ls -1d */` misses **hidden** directories | The `.github` clone was reported missing while present. |
| T7 | `dpkg-query -W` exits 0 for **uninstalled** packages | False "tool present". Use `command -v`. |
| T8 | `gh pr checks` can show a **stale red** check while a new run is in progress | Judging a merge against the wrong head SHA. Use `--watch` and pin the exact head. |
| T9 | Two agents running `terraform plan` in the **same** control-plane directory | Local state lock contention and interleaved `.terraform/` writes. Serialize Terraform per plane. |
| T10 | `gh auth switch` is **global** to the host user's keyring | A parallel agent silently changes identity for every other agent. Use per-agent `GH_TOKEN` env instead. |
| T11 | MCP server secrets passed on the **command line** | Visible to any local process via `ps` / `/proc/*/cmdline`. Use env or a file with mode 0600. |
| T12 | `terraform init -backend=false` in CI | Masks a create-heavy plan against **empty** state, so the plan passes vacuously. |

# 5. QUESTION 1 — The counting rule and the real denominator

The operator asked, three times, whether the repo counts are "necessary" and for your
professional opinion. Answer it properly.

- **5.1 Define one counting rule and apply it to all three orgs.** State explicitly, with
  a reason each, whether the denominator includes: `.github`; forks; archived repos;
  empty repos (size 0); template repos; private repos; repos with no local clone. Then
  give the exact `gh api` + `jq`/`awk` command that computes it, and the command that
  proves the local clone census matches the live census in **both** directions
  (live-but-uncloned and cloned-but-not-live).
- **5.2 Four numbers per org, never one.** Produce a table with columns:
  `org | total | code-bearing | empty | archived | fork | private | uncloned | source command`.
  Reconcile against 31 / 41 / 17 / 89 and state per number: `CONFIRMED`,
  `WRONG — actual is N`, or `CANNOT VERIFY — reason`.
- **5.3 Resolve the `journeyoflife-org` discrepancy.** 41 claimed vs 42 local org
  directories. Name the specific repo(s) that explain the difference, or state that you
  cannot determine it and give the one command that would.
- **5.4 Verdict on the HTML URLs.** Are the six URLs the operator supplied necessary,
  sufficient, both, or neither as audit input? One verdict, with the clause-level reason.
- **5.5 Verdict on 89 as an effort denominator.** Is 89 the right number to divide by a
  daily rate? Answer bluntly. If not, state what the effort denominator actually is.
- **5.6 Opinion the operator did not ask for but needs.** Are there repos that should
  **exist and do not**, or **exist and should not**? Consider at minimum: a per-org
  audit-evidence repo, a shared-workflow repo per org (Via-Vitae has
  `viavitae-reusable-workflows`; do the other two?), and whether three separate
  `hermes-agents` repos is correct isolation or triplicated maintenance burden. Give a
  verdict per item, with the supply-chain consequence of any shared module you endorse.

# 6. QUESTION 2 — Tiering: the actual acceleration lever

Parallelism without tiering multiplies wasted effort. Tier first.

- **6.1 Define tiers T0–T4** with, for each: membership rule (machine-decidable, not
  judgmental), audit depth, which of the existing 12 dimensions / D1–D9 apply, target
  wall-clock minutes per repo, and whether it runs in the bulk sweep or as an individual
  agent session.
  - **T0** — control planes (3 repos). Full Gates 0–5. Deepest.
  - **T1** — code-bearing deployables and anything in PCI/cardholder scope.
  - **T2** — compliance and governance artifact repos. Audit **content presence and
    citation integrity**, not just settings: every artifact cited by application code
    must exist, be non-empty, and resolve.
  - **T3** — scaffolds, templates, landing sites. Batch: one representative deep audit,
    then a delta check on the rest.
  - **T4** — empty and archived repos. Bulk org-level sweep only, **no per-repo session**.
    Archived repos still leak secrets and still carry history — say what you check on them.
- **6.2 Emit the full tiering table — one row per repo, all repos, all three orgs.**
  Columns: `org | repo | tier | visibility | size | code-bearing? | cited-by | dimensions
  in scope | est. minutes | evidence dir | agent template id`. A table that summarises
  instead of enumerating is a contract violation.
- **6.3 Justify every T2 and T4 assignment against the risk that it is a
  cited-but-empty control artifact.** This is the finding class that produced the
  strongest result in the 2026-10-26 audit; do not tier it away to save time.
- **6.4 Throughput arithmetic.** Show your working. Inputs: repo count per tier, minutes
  per tier, concurrency, and **human review minutes per report**. Output: total wall-clock
  days for (a) the current serial model, (b) your model. State every assumption. If your
  model does not beat 89 days by at least 5×, your design is not finished.
- **6.5 Verdict.** One sentence: what is the single largest source of the current slowness?

# 7. QUESTION 3 — Two-phase execution design

- **7.1 Phase A — bulk org sweep (once per org, one session).** Design a single script
  per org that fetches **all** org-wide evidence in one pass and writes a
  machine-readable facts table. Required: one row per repo; columns for visibility, size,
  default branch, emptiness, protection mechanism (classic branch protection **and**
  ruleset — they are different paths and both must be checked), required status contexts,
  secret scanning, push protection, Dependabot alerts by severity (**using the correct
  jq path**), code-scanning alerts (**using the correct endpoint**), CODEOWNERS presence,
  LICENSE, teams and collaborators with push access, Actions workflow refs and whether
  each is SHA-pinned, last commit date, and open PRs. Must be resumable, must honour rate
  limits, must write to a uniquely-named file, and must never let an HTTP error body
  enter a data column (trap T1).
  Explain **why** this phase exists: every per-repo session that re-fetches org-wide data
  is duplicating cost and creating N divergent copies of the same fact.
- **7.2 Phase B — parallel per-repo deep audit.** Each agent receives **one row** of the
  Phase A facts table plus a read-only worktree. It must not re-fetch anything Phase A
  already produced. State exactly what it may fetch additionally.
- **7.3 Phase C — aggregation.** One reducer merges all agent outputs into a per-org
  findings register and a cross-org rollup. Deduplicate by `(control_id, repo)`.
  Escalate contradictions between agents instead of silently picking one. Name the merge
  mechanism (the prior runs used a single `awk` join over TSVs to avoid transcription
  error — reuse that principle: **no human retyping of evidence into tables**).
- **7.4 The student-level explanation.** Explain the whole three-phase pipeline in a
  numbered walkthrough with a concrete worked trace for **one** repo, from Phase A row to
  merged finding. Use the 8-element block.

# 8. QUESTION 4 — Parallelism, isolation and the real bottleneck

The operator's stated intent: *"I can perform an audit on 5 different repositories in
parallel"* from each of the three control planes — i.e. up to 15 concurrent sessions.

- **8.1 Verdict on 5 × 3 = 15.** Safe, unsafe, or safe-with-conditions. Assess against:
  host CPU/RAM/disk (state the command that measures it), GitHub primary **and secondary**
  rate limits per token, local Terraform state locking (trap T9), `gh` keyring identity
  being global (trap T10), evidence-directory collisions, and — decisively — **human
  review capacity**.
- **8.2 Name the real bottleneck.** Fifteen parallel audits produce fifteen reports for
  **one** reviewer. If review is serial and slower than generation, parallelism increases
  work-in-progress and total elapsed time without increasing throughput. Give the
  verdict, the arithmetic, and the design change that follows from it (e.g. batch review
  by tier, review the T0/T1 set first, cap concurrency at the rate one human can review).
- **8.3 Recommended concurrency, per org and globally.** One number each, with the reason
  and the rejection record for the alternatives.
- **8.4 Workspace isolation design.** Per agent: a dedicated `git worktree` (give the
  exact `git worktree add` command and the cleanup command), a dedicated evidence
  directory `audit-evidence/{org}/{repo}/{run-id}/`, mode `0700`, no shared mutable file,
  **no shared `.terraform/` directory**, and at most one Terraform session per plane at a
  time. Explain why `git worktree` beats N clones for this (single object store, no
  duplicate fetch, and the isolation is per-checkout not per-repository).
- **8.5 Credential isolation design.** Per org: one **read-only** fine-grained PAT,
  injected as `GH_TOKEN` in that agent's environment only, never via `gh auth switch`,
  never the interactive `admin:org` classic PAT, never written to a file with mode looser
  than `0600`, never on a command line. State the minimum permission set per phase
  (Phase A needs org-level repo read; Phase B needs contents + security events read).
  Note the existing contract: the CI credential must hold **zero** Organization
  permissions (`viavitae-control/scripts/check_ci_credential.sh`). Note also that
  fine-grained PAT permission edits preserve the token string, so Actions secrets do not
  need re-pasting.
- **8.6 Rate-limit engineering.** Concrete ceiling, backoff policy, the response headers
  you read, the retry count, and the **loud failure** behaviour when a repo cannot be
  fetched. A silently skipped repo must appear as a `NOT-RUN` row in the register, never
  as an absent row.
- **8.7 Resumability.** A `run-id`, a `manifest.json` recording `{org, repo, tier, status,
  started, finished, evidence_dir, findings_count}` per repo, and a resume command that
  picks up only the unfinished rows. Design for the day being interrupted — the operator
  works alone and part-time.
- **8.8 Failure containment.** What happens when one agent hangs, crashes, emits invalid
  JSON, or reports a finding that contradicts another agent? Give the timeout, the
  validation command (`jq -e` against the schema), the quarantine directory, and the rule
  that a quarantined repo is re-run, never assumed clean.

# 9. QUESTION 5 — MCP server and hermes-agents configuration

"Correctly configure the MCP server + hermes-agents" is not actionable until servers,
roots, scopes and write capability are named. Make it actionable.

- **9.1 Inventory what exists.** Read `jol-mcp-servers/servers/`, `shared/`, `inventory/`,
  and the three `*-hermes-agents` trees. Produce a table:
  `server/agent | path | language | entrypoint | org scope today | tools exposed | write
  capability | config file | how you verified`. **Write capability must be `none` for
  every server used in an audit.** Any server that can write is excluded from the audit
  path, and you must say so.
- **9.2 Per-org MCP configuration.** Emit the concrete config file for each of the three
  orgs, with: `filesystem` root pinned to **that org's tree only** (explain the leak that
  an unpinned or `/`-rooted MCP server causes across three controllers), a GitHub/`gh`
  server restricted to read-only tools for that org, secrets via `env` and never `argv`
  (trap T11), one server process per org, and an explicit `disabled` list for anything
  with write access. State the exact file path each config lives at and the command that
  validates it parses.
- **9.3 Verdict: one shared MCP deployment or three?** Given constraint 3.2 (three
  controllers) and the fact that `jol-mcp-servers` is the only mature implementation,
  decide whether the other two orgs consume it, fork it, or get their own. Address the
  supply-chain and maintenance consequence of each, and the boundary-1 violation risk of
  a shared deployment whose filesystem root spans `/opt`.
- **9.4 hermes-agents role.** State precisely what the agent layer does in this pipeline:
  which phase it drives, how it receives the Phase A facts row, how it is constrained to
  read-only, how its output is validated against the findings schema before acceptance,
  and how `evaluations/` is used to prove the agent's queries are correct (see §10).
- **9.5 Agent prompt template.** Emit the **verbatim** per-repo prompt template used in
  Phase B, with only `{org}`, `{repo}`, `{tier}`, `{facts_row}`, `{run_id}`,
  `{evidence_dir}` substituted. It must forbid: prose reports, re-fetching Phase A data,
  any write verb, any cross-org path access, and any finding without both a `declared`
  and an `actual` half.
- **9.6 Bootstrap script.** One copy-pasteable script that sets up worktrees, evidence
  directories, per-org env, and validates the MCP configs, using **only tools present on
  the host**. Any tool not present comes with its install command and provenance.
  Mutating steps carry the §3.3 gate line.

# 10. QUESTION 6 — The tool-self-test gate (accuracy before speed)

The single most damaging event in the prior audit was **not** an access limitation. It was
an auditor query bug recorded as an access limitation: `.severity` instead of
`.security_advisory.severity` produced "unreadable", which was written up as a blocked
cell, which made the Security gate look merely incomplete instead of **FAIL** with
20 critical alerts open. The report was too lenient because the auditor trusted his own
tooling.

- **10.1 Design the gate.** Before any cell may be recorded `BLOCKED`, `CANNOT VERIFY` or
  `403 — plan limitation`, the agent must prove the query itself is correct by running it
  against a **known-positive control** — a repo already known to hold that data — and
  printing the raw JSON. Define the known-positive control per query class.
- **10.2 Emit the query correctness table.** One row per API query used anywhere in the
  pipeline: `fact | endpoint | exact jq path | known-positive control repo | verified
  output (truncated, non-sensitive) | trap id`. Cover at minimum: Dependabot alert
  severity, code-scanning alerts, secret-scanning alerts, branch protection, rulesets,
  required status contexts, team push bindings, Actions workflow ref pinning, repo
  emptiness, and CODEOWNERS presence.
- **10.3 Use the existing evaluations harness.** `*-hermes-agents/evaluations/` already
  exists in all three planes. State how these query assertions become a repeatable
  evaluation, so the next audit cannot regress silently. This is the compensating control
  for having no second reviewer (constraint 3.1).
- **10.4 Verdict.** Which prior findings are now suspect because they rest on unverified
  queries? Name them, or state that you re-tested all of them and list the results.

# 11. OUTPUT CONTRACT

Structure the entire reply as follows. Do not deviate.

- **§0 Verification results.** The `CONFIRMED` / `STALE — actual is X` / `CANNOT VERIFY —
  reason` / `SCREENSHOT-ONLY` table over every row of §4, including the repo-count
  reconciliation and the trap list.
- **§1 Executive verdict.** Ten lines maximum: the three count verdicts, the tier split,
  the concurrency verdict, the projected days, and the three highest-risk findings.
- **§2 Question 1** (counting rule) … **§7 Question 6** (tool-self-test gate). Each
  follows its own sub-numbering. Every claim broken into numbered or lettered sub-points.
  Every step executable: name the file, the command, or the UI path, and the directory it
  runs from. Mutating steps carry the §3.3 gate line.
- **§8 The full tiering table** — every repo, all three orgs, no summarising.
- **§9 The three deliverable scripts** — Phase A sweep, Phase B agent template, Phase C
  reducer — each in a fenced block, each runnable, each stating its cwd.
- **§10 The MCP + hermes configuration files** — three concrete configs plus the bootstrap
  script.
- **§11 Control mapping table.** Columns: `framework clause → implemented control → which
  question it came from → evidence artifact that proves it → status` using the five-value
  vocabulary from `docs/COMPLIANCE.md` (`enforced-verified` / `declared-not-applied` /
  `deferred-empty-repo` / `gap-plan-tier` / `out-of-band`). **No bare ✅/⚠️/❌.**
- **§12 Findings register.** Reuse each plane's existing ID scheme (`VV-YYYY-MM-NNN` for
  Via-Vitae; **read** the jol and jolarca registers and reuse theirs — do not invent one).
  Every finding carries the mandatory schema: `id, title, severity (P0|P1|P2 from the
  rubric, never from impression), control_id, framework_refs[], declared{source,claim},
  actual{source,observed}, evidence[command -> output], impact, remediation,
  verification_after_fix, owner (human|terraform), status`. **`declared` and `actual` are
  the load-bearing pair: a finding without both is an opinion, not an audit result.**
- **§13 Ordered remediation roadmap.** Sequenced for one person working part-time. Mark
  prerequisites explicitly. Give a realistic sequence, **not a parallel fantasy**.
- **§14 What I could not determine.** Explicit list. **This section may not be empty and
  may not be omitted.**
- **§15 Residual risk accepted.** What remains unmitigated after the full roadmap, and why
  that is tolerable for a single-operator charity and free-tier commercial entities.
- **§16 Self-audit checklist.** One row per numbered sub-requirement in §5–§10 of this
  prompt (5.1 … 10.4) plus the formatting rules. Columns: `requirement | section of your
  reply that answers it | verdict (yes/no)`. **Every row filled. An unfilled row is a
  contract violation.**

**Severity rubric** (from `policies/AUDIT_PROMPT.md` §7 — do not redefine it):

| Sev | Definition |
| --- | --- |
| **P0** | Live and exploitable now, **or** the system of record is not reproducible from committed code. Includes unmanaged protection on a control plane, uncommitted control files, and a control documented as enforced that is not enforced and whose absence permits history rewrite or data exposure. |
| **P1** | Framework control not enforced but not immediately exploitable, **or** auditor-facing documentation that is factually wrong, **or** a broken emergency/rollback path. |
| **P2** | Hygiene, reproducibility and minimisation issues with no direct control impact. |

**WORKED EXAMPLE of the expected teaching depth** (illustrative form only — do not copy
the values, they are placeholders):

> **8.5.c Two parallel agents cannot share one `gh` identity.**
> c.1. *Mechanism.* The `gh` CLI stores the **active account** in the host user's keyring,
> one value for the whole user session. `gh auth switch` mutates that single global value.
> Agent 1 switches to org A; agent 2 switches to org B 400 ms later; agent 1's next
> `gh api` call now executes **as org B's identity**.
> c.2. *Control failure.* Cross-controller credential use — a boundary-1 violation and a
> GDPR Art. 5(1)(f) / ISO A.5.15 failure, because an entity's data was fetched under
> another entity's authorisation. It is also **silent**: the call succeeds, the evidence
> file is written, and nothing records that the identity was wrong.
> c.3. *Correct design.* Never call `gh auth switch` inside a run. Give each agent its own
> read-only fine-grained PAT via the `GH_TOKEN` environment variable in **that process
> only**, so identity is per-process, not per-host. Verify before the sweep, from the
> agent's own working directory:
> `GH_TOKEN="$PAT" gh api user --jq .login > "$EVIDENCE_DIR/identity.txt" 2>&1` — and
> assert the returned login is the expected one, failing the run if it is not.
> c.4. *Rejection consequence.* Any orchestration that shells out to a shared `gh`
> keyring is rejected regardless of how good its tiering is, because its evidence is not
> attributable to an identity — and unattributable evidence is not evidence.

# 12. STOP CONDITIONS

Halt and report instead of answering if:

1. A §4 baseline fact is contradicted by the live tree **and** the contradiction changes a
   verdict. Report it, then continue on the corrected value.
2. You are asked to invent a repo name, count, identifier, path or clause you have not read.
3. A recommendation would require a **second human being** to operate. Say so, and propose
   the compensating control instead.
4. A recommendation depends on a **paid tier**, a vendor contract, or an AWS Organizations
   change the operator does not control. Flag the dependency **before** the recommendation.
   GitHub Free already removes secret scanning on private repos and requires Team+ for
   branch protection.
5. The correct answer is **"do not do this"** — say it plainly rather than softening it.
   If 15-way parallelism is unsafe, say unsafe.
6. Two frameworks conflict (e.g. PCI-DSS log retention vs GDPR minimisation, or public
   audit transparency vs confidential evidence). Surface the conflict and resolve it
   explicitly; never silently pick one.
7. You cannot determine something. Write it into §14. **Never fill the gap with a
   plausible guess.**
8. A step you are about to emit mutates state or live infrastructure and lacks the §3.3
   approval gate. Halt, add the gate, continue.
9. A step would place one entity's data, credential or MCP filesystem root inside another
   entity's boundary. Halt — this is a reportable incident, not a design trade-off.
10. You are about to record `BLOCKED` or `CANNOT VERIFY` **without** having run the §10.1
    known-positive control. Halt, run it, then decide.

# 13. ANTI-PATTERNS — reject these in your own output

- Accepting 31 / 41 / 17 / 89 without re-deriving them from the API.
- Treating an HTML org page or `/repositories` listing as evidence.
- Counting `.github`, forks, archived or empty repos inconsistently between orgs.
- Assigning an empty repo and a payments service the same audit budget.
- Fifteen agents sharing one `gh` keyring account (trap T10).
- Two agents running `terraform plan` against the same local state directory (trap T9).
- An MCP server whose filesystem root spans `/opt`, exposing all three controllers at once.
- A secret passed in `argv` where `ps` can read it (trap T11).
- Agents emitting **prose reports** instead of schema-valid JSON, so fifteen documents must
  be read to produce one register.
- Claiming a speedup without measuring the **human review** time it creates.
- Reporting parallelism as the fix when the denominator was never tiered.
- Recording `BLOCKED` without running the known-positive control first (§10.1).
- Re-reporting a retracted finding (the jq-path severity bug) without re-testing the query.
- Treating an empty repo as "no findings" rather than as a possible cited-but-absent control.
- Treating a scaffold repo's green CI as evidence the product builds.
- Trusting a green check from a stale run, or judging a merge against the wrong head SHA (trap T8).
- Writing `✅` in a control matrix instead of the five-value status vocabulary.
- Citing a framework clause without naming it.
- Presenting a comparison with no verdict and no rejection record.
- Recommending a tool absent from the host without its install command and provenance.
- Treating "no team" as a reason to weaken controls rather than to add compensating ones.
- Emitting an `apply` / `destroy` / `import` / `migrate` step without the printed
  human-approval gate.
- Declaring the audit complete when the 15 previously `NOT-RUN` deep audits are still unrun.
~~~~

---

## Provenance

Baseline facts recorded from, on 2026-10-26:

- `/opt/viavitae/repos/viavitae-control` — `policies/AUDIT_PROMPT.md`, `repos_data.tf`,
  `docs/COMPLIANCE.md`, `docs/security/pre-deployment-independent-audit-2026-10-26.md`,
  `docs/superpowers/plans/2026-09-27-…-per-repo-audit.md`,
  `docs/superpowers/prompts/2026-10-19-multi-org-secrets-and-state-architecture-prompt.md`,
  `scripts/check_ci_credential.sh`
- Local clone census of `/opt/viavitae/repos`, `/opt/jol/repos`, `/opt/jolarca/repos`
- Asset listings of `viavitae-hermes-agents`, `jol-hermes-agents`,
  `jolarca-hermes-agents/agents`, `jol-mcp-servers`

**Not verified by shell in this pass** (the model must verify before relying on them):
live org repo counts, jol/jolarca findings-register ID schemes, current `gh` token scopes,
host CPU/RAM headroom, and the contents of `jol-mcp-servers/servers/`.

## Revision history

- v1 (2026-10-26): initial prompt asset. Composes `policies/AUDIT_PROMPT.md` and the
  2026-10-19 multi-org prompt rather than replacing either.

## Related artifacts

[AUDIT_PROMPT.md](../../../policies/AUDIT_PROMPT.md) ·
[AI_WORKFLOW_POLICY.md](../../../policies/AI_WORKFLOW_POLICY.md) ·
[TEMPLATE.md](../plans/TEMPLATE.md) ·
[pre-deployment-independent-audit-2026-10-26.md](../../security/pre-deployment-independent-audit-2026-10-26.md) ·
[2026-10-19 multi-org secrets and state prompt](2026-10-19-multi-org-secrets-and-state-architecture-prompt.md)
````

**What changed vs your draft, in one line each:**

a. Counts → a **counting rule** + four numbers per org; 41 for `journeyoflife-org` is flagged inconsistent (42 local org dirs).
b. 89 repos → **T0–T4 tiering**; empty repos and scaffolds stop consuming a full day each.
c. "5 in parallel" → a **concurrency verdict** bounded by your review capacity, plus worktree/credential/state isolation (the two collisions that would have silently corrupted evidence).
d. "configure MCP + hermes" → **§9**, requiring configs reconciled to files that actually exist in `jol-mcp-servers` and the three `*-hermes-agents` trees.
e. "more judgement writing" → **no-hedging clause + verdict + rejection record** on every question, and **§16 self-audit checklist**.
f. "student will understand" → the **mandatory 8-element block** per phase plus a worked example.
g. New **§10 tool-self-test gate** — the jq-path bug that made your last report too lenient is now a hard precondition for any `BLOCKED` cell.

**Next steps:**

a. Save the block above as `docs/superpowers/prompts/2026-10-26-multi-org-parallel-pre-deployment-audit-prompt.md` on a new branch:
```bash
cd /opt/viavitae/repos/viavitae-control
git switch main && git pull --ff-only
git switch -c docs/parallel-audit-prompt-2026-10-26
```
b. Run the **Refresh commands** section and paste the output back to me — I'll pin §4.2 with real counts and fix the `journeyoflife-org` 41-vs-42 discrepancy before you spend a top-tier run on it.
c. Send only the text between the `~~~~` fences to the model.
d. Commit with a standalone `ai:top` trailer line in the body and the same trailer in the PR body.

---
# Via-Vitae engineering audit and remediation report

29 October 2026. This is a source review with local tests and selected live GitHub configuration checks. It is not an independent assurance report, a certification or approval to process personal data.

## 1. Outcome and scope

The request to finish Via-Vitae spans an organization of 31 repositories. The inventory contains 23 nonempty repositories and 8 empty repositories. Seven repositories were downloaded: the API, Hermes agents, control plane, compliance, organization defaults, documentation and website. Detailed source review and executable verification focused on the API and Hermes. Selected live GitHub protection settings were checked for the API, Hermes and control plane. Downloading the other repositories does not mean their code was fully audited.

Three concrete defects were reproduced with regression tests and repaired in draft pull requests:

1. API tenant resolution accepted a client header/hostname outside development. The dependency now requires trusted identity in staging/production, and all pastoral AI routes use it.
2. Hermes accepted empty or whitespace-only values for its seven isolation controls. These now fail validation.
3. Hermes audit verification raised an incidental `ValueError` when event and hash counts differed. It now reports failed verification and refuses append with `AuditTamperError`.

The overall product is not finished. Authentication middleware, data models, persistence, payment processing, retrieval, production approvals and operational evidence remain incomplete. Qoder has been assessed and a detailed adoption guide prepared; no Qoder runtime has been installed, authenticated or connected to customer data.

## 2. Findings and disposition

Severity describes potential release risk, not proof of an exploited production vulnerability. No production traffic or customer database was inspected.

| ID | Priority | Evidence | Consequence | Disposition |
| --- | --- | --- | --- | --- |
| VV-A01 | High | API `app/api/deps.py`; no authentication middleware in `app/main.py` | Client-chosen tenant identity could become an access bypass once handlers are implemented | Partial remediation: trusted-claim gate added and mounted on pastoral routes. Actual token verification, membership checks and resource authorization remain release blockers. |
| VV-A02 | Medium | Hermes `config.py` accepted blank identifiers in all seven fields | Missing configuration was represented as valid isolation configuration | Fixed locally; 21 blank-value cases now rejected. Configuration still does not prove infrastructure separation. |
| VV-A03 | Medium | Hermes `audit/stream.py` used strict zip without a length guard | Corruption produced the wrong failure contract | Fixed locally; both event loss and hash loss tested. Durable storage and independent checkpoints remain open. |
| VV-A04 | High | Hermes `agents/consent.py`, `contracts/consent.py` | A caller-supplied non-NONE basis is treated as verified; no separate special-category condition or evidence lookup exists | Open. Do not treat this as legal verification. Define the approved purpose/evidence model with the privacy owner before real data use. |
| VV-A05 | High | Hermes `agents/classification.py` | Keyword matching misses sensitive content without those keywords; fallback INTERNAL does not require a lawful basis | Open. Replace/validate classification and define conservative treatment of uncertainty before real data processing. |
| VV-A06 | High | Hermes `audit/stream.py`, `security/boundary.py`, architecture limitations | In-memory logging is lost on restart; a callable deny guard is not an operating-system network restriction | Open. Independently provision and test audit storage, network policy, identity and keys. |
| VV-A07 | High | API `app/services/approvals.py`, `app/services/rag/llm.py`, ORM placeholder files and endpoint stubs | Core product flows cannot complete; existing refusals are protective behavior | Open. Complete one approved vertical slice with persistence and authorization before broad rollout. |
| VV-A08 | High | API test output: 5 skips and 1 expected failure | Stripe signature and tenant isolation/replay behavior lack completed verification | Open. Build executable DB/provider tests; merely enabling an integration-test job will not implement skipped tests. |
| VV-A09 | High | Live Hermes branch protection: required contexts empty; no rulesets returned | Failing CI is not a required merge barrier under the inspected repository configuration | Open governance change. Add the actual emitted check names through `viavitae-control`, plan, review and apply through the authorized process. |
| VV-A10 | Medium | Live API and Hermes protection: zero required approving reviews | No mandatory second-person approval | Open organizational decision. A separate reviewer and appropriate enforcement are needed if independent review is part of the control design. |
| VV-A11 | High, conditional | Existing no-egress/EU-residency rules versus proposed vendor runtimes | Adding an SDK can create a new external processing path | No runtime added. Vendor due diligence and an explicit approved data flow are prerequisites. |

The special-category finding is an engineering gap against the intended privacy design. The EDPB explains that special-category processing requires additional conditions; it is not satisfied by merely identifying a general legal basis. [EDPB guidance](https://www.edpb.europa.eu/topics/key-gdpr-concepts/legal-basis_en).

## 3. What the repairs do, step by step

### 3.1 Tenant identity

1. A request enters the API. In staging/production, client-controlled `X-Tenant` and hostname values must not establish identity.
2. The dependency checks for a nonempty string in trusted request state. Without it, the API returns a standard 401 error.
3. Pastoral AI routes now run that dependency before their handler. Tests cover draft creation, fetching, decisions and audit access.
4. A future authentication component must verify the token and tenant membership before setting that state. No such verifier was invented or silently added here.
5. Development retains its existing fallback. Deployment must explicitly select staging or production; the checked Kubernetes manifest already sets production.

Limit: this is a closed gate pending authentication, not a complete login implementation. Other business routers remain scaffolded and need their own access model as implemented. A validated tenant identity alone does not authorize every record or operation.

### 3.2 Isolation configuration

1. Read each of the seven configured references.
2. Reject a value containing only whitespace or no characters.
3. Keep the existing cross-estate rejection and required-field checks.
4. Do not trim or transform credentials to make them valid.
5. Run regression cases for empty text, spaces, and tabs/newlines across every field.

Limit: a nonblank key identifier is not proof that a real key exists, is separate, or has correct access permissions. Provisioned infrastructure must supply that evidence.

### 3.3 Audit corruption

1. Compare event count with hash count before checking individual links.
2. Return failed verification if they differ.
3. Refuse to extend that history with the documented audit error.
4. Preserve append-only public behavior and existing rewrite-detection tests.
5. Document the remaining limitation: deleting matching tails or recomputing the entire chain cannot be detected without an independent anchor.

Limit: the code remains an in-memory scaffold. It is neither durable storage nor evidence of a complete tamper-resistant logging system.

## 4. Verification and evidence

Baseline: Hermes had 43 passing tests. API had 125 passing tests, 5 skipped tests and 1 expected failure.

Final verification output is supplied in the two `*-verification.txt` files. It includes lint, format, strict type checks, full local test suites and Bandit. API also runs architectural import checks, an 85% coverage floor, and OpenAPI export consistency.

New regression tests failed before implementation. The final Hermes suite has 66 passing tests. The final API suite has 139 passing tests, 5 skips and 1 expected failure. Skips and expected failure are existing unfinished work, not successful verification of those behaviors.

A secret scan of the staged changes is a separate check. No live Qoder integration test, live database isolation test, penetration test, restore exercise, production deployment or full organizational vulnerability assessment has been performed. CI results on a proposed branch must be assessed separately from local results.

## 5. Selected live governance observations

- API main: admin enforcement enabled; required checks include `CI status` and `Compliance status`; required approvals zero.
- Hermes main: required status check contexts empty, required approvals zero, no rulesets returned by the repository query.
- Control repository: active `protect-main` ruleset includes a required `provenance` check and zero required approvals. This is stronger than the historical note in its policy that described an earlier missing check; live configuration is the evidence for this observation.
- No Terraform apply or direct settings mutation was performed.

The exact repository revision inventory accompanies this report. These observations are a point-in-time sample, not a complete account/organization access audit.

## 6. Completion sequence for a student

### Step 1: define one releasable feature

Choose an end-to-end outcome, such as an authenticated reviewer retrieving and approving a synthetic draft. Write the expected inputs, outputs, errors and forbidden actions. Do not begin by replacing every `TODO`; some represent unresolved business and privacy requirements.

Completion evidence: an accepted feature specification and test cases that distinguish correct behavior from a stub.

### Step 2: establish trusted identity

Configure the approved identity provider. Validate signatures and token claims; reject expired, wrong-audience, wrong-issuer and malformed tokens. Resolve membership on the server. Add per-operation permissions and test a user trying to access another tenant's record.

Completion evidence: negative authentication/authorization tests, key-rotation behavior and documented offboarding.

### Step 3: implement persistence and isolation

Define the relevant model and a reversible migration. Bind every read/write to a tenant context. Test with two tenants and a restricted application database role. Verify that connection reuse cannot carry one tenant's context into the next request. Do not use a database owner account as proof that row-level security works.

Completion evidence: executable database tests and a migration/rollback result in a nonproduction environment.

### Step 4: implement approvals before publication

Store draft versions and content hashes. Bind decisions to the exact tenant, content and authenticated human. Reject self-approval where required, stale approvals, replayed decisions and unauthorized reviewers. Keep publication disabled until the approved record is checked transactionally.

Completion evidence: concurrency and stale-content tests; no path publishes a changed draft under an old approval.

### Step 5: complete provider adapters one at a time

Payments need provider-verified signatures, transactionally recorded event identifiers and replay-safe processing. Retrieval needs approved sources and tenant filtering. Notifications need consent/purpose checks and durable delivery state. Handle timeouts without inventing success.

Completion evidence: realistic signed fixtures, duplicate/out-of-order cases, retry tests and reconciliation against a sandbox provider.

### Step 6: close privacy and audit design gaps

Document purposes, permitted data, retention and deletion. Resolve the special-category handling and classifier design. Decide what the audit log contains and keep sensitive payloads out where possible. Test restoration and deletion behavior, including replicas and backups under the approved policy.

Completion evidence: accountable decisions plus operational tests. A generated policy template is not an approved policy.

### Step 7: pilot Qoder only in the approved scope

Follow the companion guide. Start with source code and synthetic records, select one runtime only when justified, validate effective permissions, and test failure handling. The current decision is to defer product runtime integration.

Completion evidence: vendor/data-flow approval where necessary, a bounded pilot result and denied-action tests.

### Step 8: enforce release gates and operate the service

Make the actual CI checks mandatory through governance code. Review the changes, deploy to staging, execute the full feature test and rehearse rollback. Monitor failures, budgets, audit delivery and data isolation. Only then consider production rollout.

Completion evidence: approved change record, exact artifact identifier, deployment verification, rollback result and an accountable service owner.

## 7. Compliance interpretation

SOC 2, GDPR and ISO/IEC 27001 are not interchangeable labels. SOC assurance is provided through an examination/reporting process; ISO/IEC 27001 addresses an information security management system. This audit supplies engineering observations and local test evidence only. [AICPA SOC](https://www.aicpa-cima.com/resources/landing/system-and-organization-controls-soc-suite-of-services), [ISO/IEC 27001](https://www.iso.org/standard/27001).

Do not change the repositories' “design intent” status to “compliant,” “certified” or “approved.” A privacy/security owner must resolve the legal and organizational decisions, and appropriate independent assurance must be obtained if pursued.

## 8. Reviewable changes and additional dependency finding

- [Hermes draft PR #12](https://github.com/Via-Vitae/viavitae-hermes-agents/pull/12): boundary/audit repairs and Qoder adoption guide.
- [API draft PR #12](https://github.com/Via-Vitae/viavitae-api/pull/12): trusted tenant gate and pastoral route wiring.

Both changes were committed and pushed to separate branches; neither was merged
or deployed. The supplied patches contain those commits. Gitleaks found no leaks
in the staged changes or the resulting change commits; this does not constitute
a full-history secret audit.

GitHub also reports an open medium-severity Mako dependency alert
(GHSA-5639-2j2p-m4mx), described as Windows drive-letter path traversal in
TemplateLookup. The first patched version is 1.4.2. The existing
[dependency PR #5](https://github.com/Via-Vitae/viavitae-api/pull/5) proposes that
update. It was not merged or duplicated here. Assess applicability and complete
that dependency change with its own verification.

The final inventory count is 31 repositories, correcting the initial estimate of
30 communicated during discovery.

## 9. Source revisions inspected

- [viavitae-control at b1b2e32a309f](https://github.com/Via-Vitae/viavitae-control/tree/b1b2e32a309f7624d91510510627b77b4c1c7c11)
- [viavitae-api at 328eb14830c7](https://github.com/Via-Vitae/viavitae-api/tree/328eb14830c7bb71c8f02b7eca36cf859308e55a)
- [viavitae-compliance at 2c41ce3eab7b](https://github.com/Via-Vitae/viavitae-compliance/tree/2c41ce3eab7b983c5c9775461b0100b4c47b9530)
- [viavitae-web at ca4656378679](https://github.com/Via-Vitae/viavitae-web/tree/ca4656378679bf1b5aaf38bb5ddcd61b2ec6fdde)
- [.github at 49cba3335db1](https://github.com/Via-Vitae/.github/tree/49cba3335db1b9bfcb1c080d9994248ebbbaca25)
- [viavitae-docs at db67d1c3c1fb](https://github.com/Via-Vitae/viavitae-docs/tree/db67d1c3c1fb80d39d3cdc1ccbcb9a5e1b949c21)
- [viavitae-hermes-agents at 0e35dccd290f](https://github.com/Via-Vitae/viavitae-hermes-agents/tree/0e35dccd290ffc81c02ba64902e4dd8d9c0a4847)

## 10. Final pull-request verification

All reported checks passed on both draft pull requests, including the API Docker
build, CI status, compliance workflow, CodeQL, Semgrep and dependency scan.
See `github-checks.json` for check names and run links. A workflow named
“Compliance status” passing does not constitute legal or independent assurance.
The open medium-severity dependency alert remains separately recorded above.



![[Pasted image 20261029140118.png]]