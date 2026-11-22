
I'll start by loading the relevant skills and grounding myself in the actual repo state.

Repo is a bare PyCharm scaffold (Python 3.12, zero dependencies, sample `main.py`). Now let me ground the prompt in the real org context — the user referenced `jolarca-legal/legal-text-sync.yml`.

`jolarca-legal` exists locally. Let me read that workflow and the org layout — this is the critical grounding.

Critical finding — the legal workflow references a registry called **`jolarca-compliance lawful-basis`**, not `jolarca-consent`. Let me verify the actual contract `jolarca-consent` must satisfy.

Terminal stdout isn't returning output; I'll rely on file tools. Reading the actual integration contract:

Now the org picture is clear. Critically, **`jolarca-compliance` already exists** and is named as the registry of record. Let me check for overlap before designing the prompt.

Glob results are polluted by `.git` internals. Let me narrow the search and retest the shell.

Bash is returning no stdout at all (even `echo`), so I cannot run git/builds — I'll note that limitation. Let me extract the house conventions the new repo must match.

Decisive finding: `jolarca-compliance` has a hard rule **"No personal data in this repo"** — but consent records *are* personal data (GDPR Art. 4(1)). Let me confirm the consent contract and service-side house conventions.

Very rich grounding. Two more verifications: is `banner-spec.md` (the referenced `ConsentRecord` contract) actually present, and what's the "no personal data" hard rule?

Major finding: `lawful-basis/` and `lawful-basis/consent-text/` **already exist** in jolarca-compliance. This decides the necessity question — let me read them.

I have enough evidence to answer your necessity question before optimizing the prompt.

## Professional verdict: `jolarca-consent` IS necessary — and your reasoning is provable from the code

I verified empirically rather than trusting the premise. Findings:

| Evidence | Source | Implication |
|---|---|---|
| "No personal data in this repo (hard rule)" | `jolarca-compliance/docs/architecture.md:23` | Consent records contain personal data → **cannot** live in `jolarca-compliance` |
| "the product database records (`ConsentRecord`)" | `jolarca-compliance/lawful-basis/README.md:3-4` | Today the record of consent lives **inside the app** — exactly what you object to |
| "Live check lands with the compliance registry API… stays a stub until that contract exists" | `jolarca-legal/scripts/cross-check-consent.py:61-64` | The re-consent control is **unimplemented** — blocked on a registry that has no home |
| `RC-*` classes are "exhaustive: any new data flow must map to a class before go-live" | `jolarca-compliance/docs/retention-schedule.md:59` | **No consent retention class exists** → go-live blocker |

**The decisive argument is separation of duties, not just versioning.** Whoever controls the `jolarca` app database currently controls the evidence that consent was given. That fails GDPR Art. 7(1) demonstrability, ISO 27001 A.8.12 (integrity), and SOC 2 CC7.x — because the control and the thing it constrains share an owner. A consent registry must be independently versioned *and* independently owned. **Verdict: build it.**

**But your draft prompt is missing the hard part.** Creating `jolarca-consent` does not fix the problem — it creates a *second* source of truth, because `jolarca-legal` hardcodes `jolarca-compliance/lawful-basis` as registry of record. The prompt must re-cut the boundary, or you ship a worse architecture than you have now.

Four decisions materially change the prompt. I have recommendations for each:

All four recommendations accepted. Let me finish grounding the prompt in real file paths and house conventions.

Complete picture. Let me confirm the house tooling targets so the prompt cites real commands.

One correction to my own earlier claim, plus a check on the house service stack before I write the prompt.

House service stack confirmed (Django 5.2 + DRF). One last critical check: does `ConsentRecord` already exist as a model in the product backend? That determines whether this is a greenfield build or an extraction/migration.

`backend/apps/compliance_app/models.py` exists. Reading it — this is decisive for the prompt.

`ConsentRecord` is **absent** from `compliance_app/models.py` (which has only AuditLog, ErasureRequest, DataExport). Let me confirm whether it exists anywhere in the org.

Decisive finding: **`ConsentRecord` already exists as a Django model inside `jolarca/backend/apps/users_app/`** — literally "a table inside the app repo." This is an *extraction*, not a greenfield build. Reading the live contract:

Verification complete — and it changes the shape of the project substantially.

## Correction to my earlier claim + decisive findings

I previously said "no consent retention class exists." **That was wrong.** `RC-MARKETING-CONSENT` does exist in `retention/matrix.yaml:113-123`. Corrected findings below — all empirically verified:

| # | Finding | Evidence | Severity |
|---|---|---|---|
| 1 | `ConsentRecord` **already exists as a Django model in the app repo** — your objection is literally true today | `jolarca/backend/apps/users_app/models.py:71-91` | Confirms necessity |
| 2 | Immutability is **ORM-convention only** (`save()` raises). `QuerySet.update()`, `bulk_create`, or raw SQL bypass it. No hash, no chain | `models.py:86-91` | **Critical** — not tamper-evident |
| 3 | `ConsentPurpose` = {marketing, analytics, personalization, transactions} but version ledger = {CT-marketing, CT-location, CT-ai}. **Location & AI consent cannot be recorded; analytics/personalization/transactions have no approved text** | `models.py:64-68` vs `consent-versions.md:8-10` | **Critical** — breaks `lawful-basis/README.md:16` 1:1 rule; ROPA-005/006 rely on consent |
| 4 | `consent_version = CharField(max_length=16)`, but house naming `CT-<purpose>-v<N>` yields `CT-transactions-v1` = **18 chars** | `models.py:77` vs `consent-text/README.md:5` | High — silent truncation/failure |
| 5 | `user` FK is `on_delete=PROTECT` → consent rows **block Art. 17 erasure**, with no anonymization path defined | `models.py:75` | High — Art. 17 vs Art. 7(1) unresolved |
| 6 | `lawful-basis/` is **not** in the hashed evidence roots, so "immutable once shipped" is unenforced | `evidence-hash.py:23` | High |
| 7 | `RC-MARKETING-CONSENT` exists in `matrix.yaml` but is **absent** from its declared source `docs/retention-schedule.md:14-25`; `domain: [prod]` only; marketing only | matrix vs schedule | Medium — SSOT drift |
| 8 | `banner-spec.md` is referenced as the "server-side enforcement contract" but **does not exist** | `cookie-policy/README.md:7`, `architecture.md:18` | Medium — dangling contract |
| 9 | `consent-versions.md` uses bare `TBD`, violating the house `[COUNSEL-TO-CONFIRM]` rule | `consent-versions.md:8-10` vs `retention-schedule.md:9-10` | Medium |

**Verdict: `jolarca-consent` is necessary — but this is an *extraction and remediation* of a live model, not a greenfield build.** Your draft prompt would have produced a pretty empty repo beside a still-authoritative `users_app.ConsentRecord`, i.e. a third source of truth.

## What I fixed in your prompt

Your draft was unactionable in five ways: no locked decisions (agent guesses stack/archetype), no acknowledgment that `ConsentRecord` already exists (no migration path), no boundary re-cut (dual source of truth), no enumerated cross-repo change set (the legal stub stays stubbed), and no verification gates (compliance claims unverifiable). I also added an `AGENTS.md` rules file — highest-leverage context for every future session.

The optimized prompt:

````markdown
# TASK: Extract and build `jolarca-consent` — independent consent registry of record

## 1. Role and stance

You are a **paranoid, compliance-driven architect with 30+ years of experience** building
regulated data systems. Operating standard: **SOC 2 Type II, GDPR, ISO 27001:2022**.

Non-negotiable behaviours:

- **Verify empirically.** Every file path, line, schema, and contract cited below was true
  when this task was written. Re-read each before relying on it. If reality differs,
  **stop and report the drift** — do not silently adapt.
- **Never invent legal facts.** Any retention period, statutory citation, or legal
  obligation you cannot source is written as `[COUNSEL-TO-CONFIRM]`, per
  `jolarca-compliance/docs/retention-schedule.md:9-10`. Never assert an unconfirmed
  value as fact in any artifact.
- **Surface conflicts, do not resolve them silently.** When two sources of truth disagree,
  present both with evidence and your recommendation, then wait.
- **No optimistic defaults.** A control that can be bypassed is not a control.
- Prefer **no change** over speculative change. Justify every file you add.

## 2. Mission

Extract `ConsentRecord` out of the marketplace application into **`jolarca-consent`**, an
independently owned consent **registry of record** whose entries are tamper-evident and
independently versioned — so that the party needing consent does not own the proof of it.

**Why (verified justification — do not re-litigate, but do re-verify):**

- `jolarca-compliance` has a hard rule: governance artifacts only, **never raw personal
  data** (`README.md:6-9`, `docs/architecture.md:23`). Consent records are personal data
  (GDPR Art. 4(1)) → they cannot live there.
- They currently live in the app: `jolarca/backend/apps/users_app/models.py:71-91`.
- Immutability there is an **ORM convention only** (`save()` raises on update) — bypassable
  via `QuerySet.update()`, `bulk_create`, or raw SQL. No hashing, no chain.
- Consequence: whoever controls the app DB controls the evidence of consent. That defeats
  **GDPR Art. 7(1)** demonstrability, **ISO 27001 A.8.12** integrity, and **SOC 2 CC7.x**,
  because control and controlled object share an owner.
- `jolarca-legal`'s re-consent control is **stubbed, blocked on exactly this registry**:
  `scripts/cross-check-consent.py:61-64`.

## 3. Locked architectural decisions

These are decided. Do not revisit without explicit instruction.

| # | Decision | Choice |
|---|---|---|
| D1 | Archetype | **Runtime service + append-only hash-chained ledger.** API for capture/query/withdraw; the ledger is system of record. |
| D2 | Boundary | **Consent TEXTS stay in `jolarca-compliance/lawful-basis/`** (governance, no personal data). **Consent RECORDS move to `jolarca-consent`.** Cross-repo change set in §7 is mandatory, in scope. |
| D3 | Tamper-evidence | **Append-only SHA-256 prev-hash chain**, extending the house pattern in `jolarca-compliance/scripts/evidence-hash.py` (append-only; supersede, never rewrite). No external TSA, no new vendor. |
| D4 | Delivery | **Staged, Phases 0→4, with a hard verification gate at each.** Do not start Phase *n+1* until Phase *n* gate passes and is reported. |

## 4. Boundary contract (target state)

| Repo | Owns | Must NOT |
|---|---|---|
| `jolarca-consent` | `ConsentRecord` schema; record capture/query/withdrawal; hash chain; chain verification; DSR consent export; the registry API that `jolarca-legal` cross-checks | Hold consent *texts*; hold any legal interpretation |
| `jolarca-compliance` | Consent **texts** (`lawful-basis/consent-text/<lang>/CT-*.md`), version ledger, LIA, DPIA, ROPA, retention classes, policies | Store consent records or any personal data |
| `jolarca` (product) | Consent **UX** and enforcement; calls the registry API; may cache a non-authoritative pointer | Be system of record for consent; retain a writable `ConsentRecord` table |
| `jolarca-legal` | Legal-text publication pipeline; triggers re-consent evaluation on text change | Own consent state |

## 5. Hard constraints (red lines)

1. **Append-only, enforced in depth.** ORM guard *and* DB-level protection (revoked
   `UPDATE`/`DELETE` grants for the application role, or equivalent trigger/row-level
   mechanism). ORM-only is explicitly insufficient — that is the defect being fixed.
2. **Hash chain.** Each entry carries `prev_hash` and `entry_hash`
   (SHA-256 over a canonical, field-ordered serialization). Genesis entry defined.
   Chain must be independently verifiable offline by a stdlib-only script.
3. **Withdrawal ≠ deletion.** Art. 7(1) requires proving consent was given; Art. 17 grants
   erasure. Resolve via **anonymize-don't-delete** (`jolarca-compliance/docs/adr/ADR-0001-anonymize-dont-delete.md`):
   anonymize the subject identifier, **retain** purpose, version, timestamp, and the fact
   of grant/withdrawal. Replaces the current unresolved `on_delete=PROTECT`
   (`users_app/models.py:75`).
4. **Retention is code, never hardcoded.** Read from `jolarca-compliance/retention/matrix.yaml`.
   Mirror its mode-resolution rule: any class whose `counsel_status != confirmed` is
   **report-only**; destructive action on unconfirmed classes must be impossible by
   construction (`matrix.yaml:7-10`).
5. **Version mapping is total.** Every purpose the product can request must have an
   approved `CT-<purpose>-v<N>` text, and every version ID must map 1:1 to a storable
   `consent_version` (`lawful-basis/README.md:15-16`). Enforce at the schema and API
   level, not by convention.
6. **No secrets, no personal data in git.** No fixtures containing real personal data;
   tests use synthetic data only. `.env` never committed.
7. **House tooling standards.** Python ≥3.12; `ruff` line-length 100, `target-version py312`,
   explicit rule `select` (never rely on version-dependent defaults) including `S` (bandit)
   and `I`; `mypy --strict`; `pytest`; `pre-commit`; `Makefile` exposing `check` as the
   pre-merge gate; Conventional Commits; `CHANGELOG.md`.
8. **CI actions pinned by commit SHA** with a trailing version comment, per
   `jolarca-legal/.github/workflows/legal-text-sync.yml:20-23`. Least-privilege
   `permissions:` on every job.
9. **`[COUNSEL-TO-CONFIRM]`** for every unsourced legal value. Bare `TBD` is not acceptable.

## 6. Phases and exit gates

### Phase 0 — Verify, decide, specify (no production code)

Re-verify §2 evidence and produce `docs/architecture.md`, `docs/threat-model.md`,
`docs/migration-plan.md`, plus ADRs. **Explicitly decide and document:**

- Web stack. House product stack is **Django 5.2 + DRF + drf-spectacular + psycopg3 +
  PostgreSQL + structlog + Celery/Redis** (`jolarca/backend/pyproject.toml:11-29`).
  Adopt it unless you present a concrete compliance argument against it.
- **License.** Product backend is `AGPL-3.0-only`; `jolarca-compliance` is
  `Proprietary — all rights reserved`. Pick deliberately for a system-of-record
  service holding personal data, and justify.
- Identity/authN for registry callers (service-to-service), and how the registry
  authenticates the product without trusting it.
- Canonical serialization for hashing (exact field order, encoding, null handling) —
  ambiguity here destroys verifiability forever.
- **Cutover strategy** for the live `users_app.ConsentRecord`: dual-write → backfill →
  verify → read-cut → write-cut → decommission, with rollback at each step.
- Whether existing rows can even be migrated: `consent_version` is `max_length=16`
  while `CT-transactions-v1` is 18 chars (`users_app/models.py:77`). Quantify real
  truncation before designing the target schema.

**Gate 0:** A design doc exists with zero placeholders; every open legal value marked
`[COUNSEL-TO-CONFIRM]`; threat model enumerates bypass paths for the append-only
guarantee; migration plan has a rollback per step. **Report and wait for approval.**

### Phase 1 — Repository tree and governance files

Create the full tree (§8) and all governance/tooling files. No business logic yet.

**Gate 1:** `make check` passes on the scaffold; `ruff check`, `mypy --strict`, and
`pytest` all run green (tests may be trivial); YAML parses; markdownlint clean;
`AGENTS.md` present and accurate; `.editorconfig`, `.pre-commit-config.yaml`,
`.gitignore`, CI workflows, dependabot, issue/PR templates present; `main.py` performs a
smoke-check of repository tooling per house convention (`jolarca-compliance/README.md:73`).

### Phase 2 — `ConsentRecord` schema and API contract

Implement the target model, migrations, and the OpenAPI contract. Fix the defects found:

- Purpose taxonomy must cover **marketing, location, AI** (ROPA-006, ROPA-005 rely on
  consent) *and* the existing analytics/personalization/transactions — reconciled against
  `consent-versions.md`, which currently lists only marketing/location/ai.
- `consent_version` wide enough for the `CT-<purpose>-v<N>` convention.
- Referential integrity against approved text versions; unknown version rejected.
- No unbounded personal data by accident: `user_agent` is currently an unbounded
  `TextField` (`users_app/models.py:80`) — justify, bound, or minimize it.
- Endpoints at minimum: record grant, record withdrawal, query effective consent state,
  query full history for a subject, export for DSR (Art. 15/20), verify chain integrity.

**Gate 2:** Migrations reversible; contract-tested API; tests prove that
`QuerySet.update()`, `bulk_create`, and raw SQL **cannot** mutate or delete a finalized
entry; schema-level rejection of unknown `consent_version` proven by test.

### Phase 3 — Tamper-evidence, chain verification, integrity CI

Implement the SHA-256 prev-hash chain and the stdlib-only offline verifier.

**Gate 3 — adversarial, and the point of the whole project.** Write tests that attempt to
break integrity and **must fail**: flipping a `granted` flag, rewriting a timestamp,
deleting a middle entry, reordering entries, re-anchoring the chain from a forged genesis,
truncating the ledger. Each must be detected. Also prove the verifier detects a mutation
made **outside** the application (direct DB write). Publish results as gate evidence.
A chain that only passes happy-path tests is not accepted.

### Phase 4 — Cross-repo alignment (§7) and migration execution

Deliver the cross-repo change set as **separate, individually reviewable PRs** — one per
repo — since each has different required reviewers (DPO + compliance lead for
`jolarca-compliance`, per `README.md:15-17`).

**Gate 4:** `jolarca-legal`'s `cross-check-consent.py` runs a **live** check against the
registry API (stub at `:61-64` removed); a text-version change demonstrably produces a
re-consent evaluation task; no repo retains a competing consent system of record;
`docs/architecture.md` control map updated in `jolarca-compliance`.

## 7. Mandatory cross-repo change set

Enumerate each as its own PR with rationale. Do not skip any; report if one is infeasible.

**`jolarca-legal`**
- `scripts/cross-check-consent.py` — repoint from `jolarca-compliance/lawful-basis` to the
  `jolarca-consent` registry API for **records**; keep compliance as source for **texts**.
  Replace the stub live path (`:49-65`).
- `.github/workflows/legal-text-sync.yml` — wire the notification step (`:32-35`) to the
  real contract; revisit the `TODO(legal-automation)` dry-run (`:42-47`).

**`jolarca-compliance`**
- `lawful-basis/README.md` — correct the premise that records live in "the product
  database" (`:3-4`); restate rule 2 (`:16`) against the new registry.
- `docs/architecture.md` — add control-map rows for the consent registry; update the
  `banner-spec.md` → "consent banner server-side enforcement" row (`:18`).
- `docs/retention-schedule.md` — add the marketing-consent class to the §1 table so the
  declared human/legal source matches `retention/matrix.yaml:113-123` (**current drift**),
  and add classes for non-marketing consent.
- `retention/matrix.yaml` — extend `RC-MARKETING-CONSENT` beyond `domain: [prod]` to
  include the consent-registry domain; add classes for AI/location/other consent purposes.
- `ropa/master-register.csv` — add `ROPA-007` for the consent registry. Existing columns:
  `id,system,purpose,lawful_basis,data_categories,subjects,retention_class,recipients,transfer_mechanism,last_reviewed`.
- `dpia/001-identity-and-consent/README.md` — currently **draft** (`:3`); its scope already
  names "consent engine and `ConsentRecord` linkage" (`:6`). Complete it and link ROPA-007.
- `legal/cookie-policy/banner-spec.md` — **author the missing file** referenced by
  `cookie-policy/README.md:7` and `docs/architecture.md:18`.
- `scripts/evidence-hash.py` — `ROOTS` (`:23`) omits `lawful-basis/`, so "consent strings
  are immutable once shipped" is unenforced. Add it (and justify any other root you add).
- `lawful-basis/consent-versions.md` — replace bare `TBD` (`:8-10`) with real values or
  `[COUNSEL-TO-CONFIRM]`; reconcile the purpose list with the code taxonomy.

**`jolarca` (product)**
- `backend/apps/users_app/models.py` — deprecate `ConsentRecord` (`:71-91`) and
  `ConsentPurpose` (`:64-68`) per the Phase 0 cutover plan; migrate data; retain a
  non-authoritative cached pointer only if justified.
- `backend/apps/users_app/services.py`, `backend/apps/compliance_app/services.py`,
  `backend/tests/integration/test_append_only_stores.py`,
  `docs/architecture/06-database-erd.md`,
  `docs/architecture/02-sequence-registration-2fa.md`, `docs/COMPLIANCE_MATRIX.md` —
  update every consumer and doc that references `ConsentRecord`.
  Find the authoritative list with: `grep -rn "ConsentRecord" --include=*.{py,md,yml,csv}`.

## 8. Target repository tree

Adapt to Phase 0 decisions; do not pad with unused files. Every file must have an owner
and a reason.

```
jolarca-consent/
├── AGENTS.md                     # rules file: stack, commands, conventions, boundaries
├── README.md                     # purpose, hard rules, access model, setup
├── CONTRIBUTING.md               # review + sign-off requirements
├── CHANGELOG.md                  # Conventional Commits
├── SECURITY.md                   # disclosure + severity (chain break = highest)
├── Makefile                      # `check` = pre-merge gate
├── pyproject.toml                # ruff/mypy/pytest/coverage config
├── .pre-commit-config.yaml
├── .editorconfig
├── .gitignore
├── main.py                       # tooling smoke-check (house convention)
├── docs/
│   ├── architecture.md
│   ├── threat-model.md
│   ├── migration-plan.md
│   ├── api-contract.md
│   └── adr/
│       ├── README.md
│       ├── ADR-0001-consent-registry-of-record.md
│       ├── ADR-0002-sha256-append-only-chain.md
│       └── ADR-0003-withdrawal-anonymize-dont-delete.md
├── .github/
│   ├── dependabot.yml
│   ├── PULL_REQUEST_TEMPLATE.md
│   ├── ISSUE_TEMPLATE/
│   └── workflows/
│       ├── ci.yml                # lint, types, tests, coverage
│       ├── integrity.yml         # chain verification on schedule + on demand
│       └── security.yml
├── src/  (or backend/, per Phase 0)
│   ├── project/settings/{base,dev,test,prod}.py
│   └── apps/
│       ├── consent_registry/     # models, migrations, services, api, serializers
│       └── consent_integrity/    # hash chain, canonicalization, verifier
├── scripts/
│   ├── verify_chain.py           # stdlib-only offline verifier
│   ├── migrate_consent_records.py
│   └── export_dsr.py
└── tests/
    ├── unit/
    ├── contract/                 # API vs OpenAPI
    ├── integration/
    └── adversarial/              # Phase 3 gate: attempts to break integrity
```

## 9. Definition of done

- [ ] All five phase gates passed, each **reported with command output**, not asserted.
- [ ] `make check` green in `jolarca-consent`.
- [ ] Adversarial suite (Phase 3) proves every enumerated tampering path is detected,
      including mutation from outside the application.
- [ ] Offline chain verification runs with stdlib only.
- [ ] Exactly **one** consent system of record org-wide; `grep -rn "ConsentRecord"` shows
      no repo still treating the app table as authoritative.
- [ ] `jolarca-legal` re-consent cross-check runs live and is demonstrated end-to-end.
- [ ] Every §7 item delivered as its own PR, or explicitly reported as blocked with reason.
- [ ] ROPA, DPIA, retention schedule, and architecture control map updated and consistent
      with each other (no SSOT drift).
- [ ] No unsourced legal value asserted anywhere; all are `[COUNSEL-TO-CONFIRM]`.
- [ ] No personal data committed to any repo, including test fixtures.

## 10. Out of scope

Consent banner UI implementation; consent *text* authoring or translation; legal
interpretation of retention periods (flag, never decide); the marketplace's other
lawful bases; replacing `jolarca-compliance`'s evidence registry; any external
timestamping authority or new vendor.

## 11. Escalate instead of guessing

Stop and ask when: two sources of truth conflict; a legal value has no source; a change
would weaken an existing control; live data migration risks irreversible loss; a defect
found in §2 no longer reproduces; or a §7 change requires reviewers you cannot identify.
Present evidence, options, and a recommendation — then wait.

## 12. Report format

After each phase: what you verified (with file:line), what you changed, the exact
commands run and their output, gate pass/fail, and every open question or
`[COUNSEL-TO-CONFIRM]` item. **Never claim a gate passed without pasting the output.**
````

## My professional opinion on the three things you asked about

**Is the separate repo necessary? Yes** — but not for the reason in your draft. Versioning alone wouldn't justify it; `jolarca-compliance` already versions consent texts immutably. The justification is **separation of duties**: today the app that needs consent also owns the proof of it, and that proof is protected only by a Python `save()` override. That is the finding an auditor acts on.

**Should the compliance framing stay in the prompt? Yes, and I strengthened it.** Bare "SOC2/GDPR/ISO 27001" labels produce compliance *theatre* — files that look right. I replaced them with enforceable red lines (§5) and an **adversarial gate** (§6 Phase 3) that requires proving tampering is *detected*, including from outside the app. Without Phase 3, a hash chain is decoration.

**The highest risk in your original plan, now addressed:** you would have created a *third* consent source of truth alongside `users_app.ConsentRecord` and `lawful-basis/`. §7 and Gate 4 make decommissioning the app-side table an explicit, verified exit condition.

Two items need your decision before this prompt is executed, because they are business/legal calls I should not make for you:

1. **Defects #3 and #4 are live production issues**, independent of this project — location and AI consent (ROPA-005/006) currently *cannot* be recorded, and `consent_version` is too narrow for your own naming convention. If those consent flows are already in production, that is a reportable compliance gap under `consent-versions.md:15-16`, and it should be triaged **now**, not in Phase 4.
2. **Defect #5 has no lawful answer without counsel**: consent rows `PROTECT` the user from erasure. That needs a DPO decision on anonymization before migration, or you will carry an Art. 17 violation into the new system.

Want me to save this prompt as a file in the repo, or start executing Phase 0?