# Code Review — `jol-hermes-agents`

**Org:** Journey of Life / journeyoflife-org (`https://github.com/journeyoflife-org`)
**Local root:** `/opt/projects/journeyoflife/repos/jol-hermes-agents`
*(the path in the request — `.../repos/viavitae-hermes-agents` — does not exist; the real JOL repo is `jol-hermes-agents`.)*
**Branch reviewed:** `scratch/post-merge-prep` @ `072c115` — **not trunk**
**Audit type:** code-level review (source inspection, read-only) · **Date:** 2026-11-22
**Stack:** Python + declarative YAML (config-first), SQLite memory store, SOPS, Docker/systemd, uv. Only 4 tracked Python source files (`main.py` + 3 scripts); ~1 659 LOC incl. tests.

---

## Summary

This is a **config-first validator + operational tooling repo, not the agent runtime** —
[`main.py`](file:///opt/projects/journeyoflife/repos/jol-hermes-agents/main.py) explicitly states
"Runtime orchestration lives elsewhere; this is the single entry point for CI and local
checks." The committed code is small and high quality: a repo validator plus a genuinely
well-reasoned GDPR Art. 17 erasure tool. Secret handling is the strongest of the three
(org-level SOPS encryption at rest).

**Verdict: 🟡 CONDITIONAL** — the reviewed code is sound, but the repo is on a scratch
branch and its security guarantees live almost entirely in YAML that this code only
*checks for presence*, not enforces at runtime.

---

## Strengths (verified in code)

- **Env-contract enforcement** — [`main.py:184-197`](file:///opt/projects/journeyoflife/repos/jol-hermes-agents/main.py): every `${ENV_VAR}` (and `*_env:` key) referenced under `config/` must exist in `config/example.env`, else CI fails. Comments are stripped so doc examples aren't mistaken for live refs.
- **EU residency + model pinning** — [`main.py:104-138`](file:///opt/projects/journeyoflife/repos/jol-hermes-agents/main.py): rejects non-EU provider regions, unpinned `*-latest` aliases, and any provider with `context_length < 64_000`.
- **Erasure tooling** — [`scripts/memory_erase.py`](file:///opt/projects/journeyoflife/repos/jol-hermes-agents/scripts/memory_erase.py): fails closed (rolls back if an audit table is missing), writes the `audit.approvals` row **before** deletion and the event in the same txn, refuses `audit.*` and `purge_method: anonymise` namespaces, requires a second factor from an allow-list, and is candid that Art. 17 discovery is not mechanically solvable here.
- **Secrets** — SOPS (`.sops.yaml` + `secrets/encrypted/`), `.env` git-ignored/untracked, config references secrets only via `${ENV_VAR}`. No committed secrets found.
- **CI breadth** — `ci.yaml`, `codeql.yml`, `compliance-check.yml`, `dependabot.yml`, `.pre-commit-config.yaml`, `.yamllint`, Dockerfile + docker-compose + systemd/logrotate.

---

## Findings

| ID | Sev | Finding | Evidence | Recommendation |
|----|-----|---------|----------|----------------|
| **F-1** | S2 | **Not on trunk.** HEAD is `scratch/post-merge-prep`; the reviewed state may not reflect what CI/main protects. The name ("post-merge-prep") suggests an in-flight integration. | `git rev-parse --abbrev-ref HEAD` | Re-run this review against `main`; if scratch work is intended, open a PR so branch protection + CodeQL apply. |
| **F-2** | S2 | The policy controls (e.g. `agent-policy.yaml` `secrets_in_prompts: deny`, `single_use_token`, telegram redaction regexes) are **declarative**. `main.py` validates file presence/schema, not runtime behavior — enforcement depends on the external runtime, which is out of this repo. | [`config/agent-policy.yaml:59,86`](file:///opt/projects/journeyoflife/repos/jol-hermes-agents/config/agent-policy.yaml) · [`config/gateway/telegram.yaml:25-29`](file:///opt/projects/journeyoflife/repos/jol-hermes-agents/config/gateway/telegram.yaml) | Track/point to the runtime repo that honors these; add contract tests proving the runtime reads and enforces them. |
| **F-3** | S2 | **GDPR Art. 17 erasure is not mechanically discoverable here.** `memory/schema.yaml` declares no subject-reference field in any namespace, so "delete everything about data subject X" cannot be automated — `memory_erase.py` deletes only exactly the predicate it is given, and the code itself flags this as an open gate. Real DSAR compliance therefore depends on operator judgement + the external runtime. | [`scripts/memory_erase.py:28-32`](file:///opt/projects/journeyoflife/repos/jol-hermes-agents/scripts/memory_erase.py#L28-L32) | Add a subject-reference column convention to `memory/schema.yaml` (or a per-namespace `subject_key`) so erasure can enumerate all rows for a subject; close `docs/audit/CHECKLIST.md` once done. |
| **F-4** | S3 | Erasure predicate uses f-string-quoted identifiers (`DELETE FROM "{namespace}" WHERE "{column}" = ?`). Values are parameterized (safe), and identifiers are constrained to declared schema names — but the safety rests entirely on the schema-membership check upstream. | [`memory_erase.py:167,174`](file:///opt/projects/journeyoflife/repos/jol-hermes-agents/scripts/memory_erase.py#L163-L175) | Keep the schema-allowlist invariant; add an explicit identifier-format assertion as defense-in-depth. |
| **F-5** | S3 | Working tree carries non-source noise (PyCharm `.iml`, `.idea/*.xml`, `.venv`, `*.egg-info`, `pytest_cache`). None are git-tracked, but they risk accidental adds. | disk inspection | Confirm `.gitignore` covers `.idea/`, `*.iml`, `*.egg-info`, `.pytest_cache/`. |

---

## Recommended sequence
1. F-1 — move to / PR onto trunk → 2. F-2 — add runtime contract tests for the declared policy → 3. F-3 — make Art. 17 erasure discoverable via a subject-reference schema field → 4. F-4/F-5 — hardening.

> Related: [[Code Review — viavitae-hermes-agents]] · [[Code Review — jolarca-hermes-agents]]. Org-level fleet audit lives in [[Audit — journeyoflife-org]].
