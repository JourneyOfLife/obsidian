# Code Review — `jolarca-hermes-agents`

**Org:** JolArca Marketplace / jolarca-dev (`https://github.com/jolarca-dev`)
**Local root:** `/opt/projects/jolarca/repos/jolarca-hermes-agents`
*(the path in the request — `.../repos/viavitae-hermes-agents` — does not exist; the real JolArca repo is `jolarca-hermes-agents`.)*
**Branch reviewed:** `ci/declare-node-engines` @ `b6cf8ab` — **not trunk**
**Audit type:** code-level review (source inspection, read-only) · **Date:** 2026-11-22
**Stack:** Python + Node (mixed), declarative agent fleet (`agents/<name>/{agent,policy}.yaml`), JSON Schema Draft 2020-12, bandit, qodana, dependabot. ~3 328 LOC, 45 py files, **27 test files**.

---

## Summary

The richest of the three repos in breadth: **12 agents** each with `agent.yaml` +
`policy.yaml` + own tests, an **adversarial evaluation corpus**
(`evaluations/{privacy,prompt-injection,security}/cases.yaml`), formal **JSON Schemas**, and
a **16-script policy-gate suite** (932 LOC of `check_*.py`). Test coverage is the strongest
of the three (27 files, incl. meta-tests `test_ci_gates_are_real.py`,
`test_ci_hardening.py`).

**Verdict: 🟡 CONDITIONAL** — excellent structure and eval intent, but the headline
security gates are **static YAML-string presence checks**, so "gate passed" ≠ "control
enforced" (the exact *configured ≠ effective* trap flagged in the org fleet audits).

---

## Strengths (verified in code)

- **Adversarial evals** — `evaluations/prompt-injection/cases.yaml`, `privacy/`, `security/` give a real red-team corpus the other two repos lack.
- **Formal schemas** — `schemas/agent.schema.json`, `policy.schema.json`, `provenance.schema.json`, `eval-case.schema.json` (Draft 2020-12), validated in `tests/test_schemas.py`.
- **Honest validators** — [`scripts/validate_agents.py`](file:///opt/projects/jolarca/repos/jolarca-hermes-agents/scripts/validate_agents.py) documents that its structural checks are *complementary, not equivalent* to schema conformance; enforces `identity_tag` pattern `agent:jolarca:<module-id>` + uniqueness.
- **Self-aware CI** — `ci.yml` runs a **full-history** gitleaks scan (catches secrets committed then removed); `.gitleaksignore` is an empty template (no secrets excused yet); bandit present as a dependency; dependabot configured.
- **Test discipline** — tests assert the gates themselves are wired and real (`test_ci_gates_are_real.py`, `test_ci_control_wiring.py`).

---

## Findings

| ID | Sev | Finding | Evidence | Recommendation |
|----|-----|---------|----------|----------------|
| **F-1** | S2 | **Policy gates are presence-only, not behavioral.** `check_kill_switch.py` passes iff `"kill_switch"` appears in `orchestrator.tool_grants`; `check_tenant_isolation.py` passes iff `"cross_tenant_index_access"` is in `rag.deny.actions`. These assert the string is declared — they cannot detect a runtime that ignores the grant, nor a kill-switch that doesn't work. | [`scripts/check_kill_switch.py:26-29`](file:///opt/projects/jolarca/repos/jolarca-hermes-agents/scripts/check_kill_switch.py#L26-L29) · [`scripts/check_tenant_isolation.py:28-31`](file:///opt/projects/jolarca/repos/jolarca-hermes-agents/scripts/check_tenant_isolation.py#L28-L31) | Keep the declaration checks, but pair each with a **behavioral test** (exercise the kill-switch / attempt a cross-tenant read against a stub runtime and assert it is denied). The `evaluations/` corpora are the right vehicle — wire them to a runtime harness. |
| **F-2** | S2 | **Not on trunk.** HEAD is a feature branch `ci/declare-node-engines`; reviewed state isn't what `main`/protection governs. | `git rev-parse --abbrev-ref HEAD` | Re-review on `main`; land the branch via PR. |
| **F-3** | S3 | **Mixed Python + Node runtime surface** (`package.json` + `pyproject.toml`, `node_modules` on disk). Larger supply chain and two lockfiles to keep current; confirm both are scanned by dependabot and the Node side has an equivalent `engines`/SBOM gate (the branch name suggests this is in progress). | repo root | Verify dependabot covers `npm` + `pip`; add SBOM generation. |
| **F-4** | S3 | 16 hand-rolled `check_*.py` scripts risk divergence/duplication vs the JSON Schemas; some logic is intentionally re-implemented ("hand-rolled rather than schema-driven"). | [`validate_agents.py:6-16`](file:///opt/projects/jolarca/repos/jolarca-hermes-agents/scripts/validate_agents.py#L6-L17) | Consider collapsing structural checks into schema validation to reduce drift; keep bespoke checks only where they add signal (identity, budget, coverage). |

---

## Recommended sequence
1. F-1 — back each declared gate with a behavioral/eval-driven test → 2. F-2 — rebase onto trunk → 3. F-3 — confirm dual-ecosystem dependency scanning → 4. F-4 — reduce validator/schema duplication.

> Related: [[Code Review — viavitae-hermes-agents]] · [[Code Review — jol-hermes-agents]]. Org-level fleet audit lives in [[Audit  jolarca-dev]].
