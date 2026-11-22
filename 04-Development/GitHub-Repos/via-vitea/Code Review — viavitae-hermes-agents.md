# Code Review — `viavitae-hermes-agents`

**Org:** Via-Vitae (`https://github.com/Via-Vitae`)
**Local root:** `/opt/projects/viavitae/repos/viavitae-hermes-agents`
**Branch reviewed:** `main` @ `0e35dcc` — working tree clean
**Audit type:** code-level review (source inspection, read-only) · **Date:** 2026-11-22
**Stack:** Python (src-layout package `viavitae_hermes`), Pydantic v2 / pydantic-settings, uv, ~1 619 LOC across 27 source files + 6 test files.

---

## Summary

This is a real, runnable in-process orchestrator (not just declarative config). The
security model is genuinely well engineered: **deny-by-default egress**, **fail-closed
configuration**, **mandatory human review**, and **safeguarding escalations that cannot
be suppressed by an approval**. Code style is modern and consistent
(`from __future__ import annotations`, full type hints, ruff-style `msg = f"..."`).

**Verdict: 🟢 GO for continued development; 🟡 NOT-YET-GO for production reliance** — the
control plane is sound but the pipeline is intentionally inert (see F-2), so functional
assurance is still pending.

---

## Strengths (verified in code)

- **Boundary guard** — [`security/boundary.py`](file:///opt/projects/viavitae/repos/viavitae-hermes-agents/src/viavitae_hermes/security/boundary.py): `assert_no_external_egress()` always raises; egress is structurally impossible, not merely unused.
- **Fail-closed config** — [`config.py`](file:///opt/projects/viavitae/repos/viavitae-hermes-agents/src/viavitae_hermes/config.py): `frozen=True`, `extra="forbid"`, secrets are `SecretStr` with **no defaults**, and a `model_validator` rejects any control referencing another estate. Loading incomplete config raises instead of degrading.
- **Orchestrator gates** — [`agents/orchestrator.py`](file:///opt/projects/viavitae/repos/viavitae-hermes-agents/src/viavitae_hermes/agents/orchestrator.py): halts when there is no verified lawful basis, never auto-approves/sends, records an `AuditEvent` at every step; `_resolve_status()` guarantees a safeguarding escalation outranks a human approval.
- **Hygiene** — `.env` is git-ignored and untracked (no committed secrets); ships `.env.example`, `.gitleaks.toml`, `gitleaks.yml` + `ci.yml` workflows, `.pre-commit-config.yaml`, `CODEOWNERS`.

---

## Findings

| ID | Sev | Finding | Evidence | Recommendation |
|----|-----|---------|----------|----------------|
| **F-1** | S3 | Estate-isolation check is a naive substring test on `casefold()`. `_FORBIDDEN_ESTATE_MARKER = "jol"` matches any value containing the letters "jol", producing **false positives** (legit IDs/words) while still not being a true boundary — real isolation is the deployment (separate DSN/KMS/namespace), which the check only *asserts*, not verifies. | [`config.py:15,61`](file:///opt/projects/viavitae/repos/viavitae-hermes-agents/src/viavitae_hermes/config.py#L60-L66) · [`boundary.py:19`](file:///opt/projects/viavitae/repos/viavitae-hermes-agents/src/viavitae_hermes/security/boundary.py#L38-L46) | Treat the marker as defense-in-depth only; move to explicit allow-listed estate identifiers rather than substring match. Document that true isolation is enforced infra-side. |
| **F-2** | S2 | The pipeline is **non-functional by design in this release**: `NullRetrievalSource` is the default and the boundary guard always raises, so no real retrieval/egress occurs. Fine for a scaffold, but means end-to-end behavior is unproven. | [`orchestrator.py:210-212`](file:///opt/projects/viavitae/repos/viavitae-hermes-agents/src/viavitae_hermes/agents/orchestrator.py#L210-L213) · [`retrieval.py:23`](file:///opt/projects/viavitae/repos/viavitae-hermes-agents/src/viavitae_hermes/agents/retrieval.py) | Before any production reliance, introduce a real `RetrievalSource` + an allow-list-driven egress path behind a config flag, with its own tests. |
| **F-3** | S3 | The `evaluations/` **framework is scaffolded but empty**: it defines the right categories (`functional`, `privacy`, `prompt-injection`, `regression`, `security`) and rules ("synthetic only, reproducible, gated"), but each directory holds only a 3-line README — **no actual case files yet**. The sibling `jolarca-hermes-agents` already ships real `cases.yaml`. | [`evaluations/*/README.md`](file:///opt/projects/viavitae/repos/viavitae-hermes-agents/evaluations) | Populate each category with concrete cases (esp. prompt-injection + consent-forgery) and wire the evals into CI as a release gate; otherwise "gated" is aspirational. |

---

## Recommended sequence
1. F-2 — land a real retrieval/egress path behind a gate, with tests → 2. F-3 — add adversarial eval corpus → 3. F-1 — refine the isolation marker into an explicit identifier.

> Related: [[Code Review — jol-hermes-agents]] · [[Code Review — jolarca-hermes-agents]] (three independent implementations of the same "hermes-agents" concept).
