
### Overview

Below are ready‑to‑use artifacts your DevOps team can drop into GitHub and ArgoCD. They are **compliance‑first**, auditable, and designed for a RAG pipeline that enforces PII redaction, immutable index snapshots, and human sign‑off before publishing factual pages.

---

### GitHub Actions CI workflow YAML snippet

**Purpose** Build and test ingestion, create embedding snapshot, run automated checks, generate draft content, open PR for human review.

```yaml
name: rag-ci-pipeline

on:
  push:
    branches:
      - main
  workflow_dispatch: {}

permissions:
  contents: read
  pull-requests: write

jobs:
  ci:
    runs-on: ubuntu-latest
    env:
      INDEX_SNAPSHOT_BUCKET: rag-index-snapshots
      EMBEDDING_MODEL: pinned-embedding-v1
      LLM_ROUTER_ENDPOINT: https://llm-router.internal/api/generate
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r infra/requirements.txt

      - name: Lint ingestion code
        run: |
          flake8 ingestion || true

      - name: Run ingestion unit tests
        run: |
          pytest tests/ingestion -q

      - name: Run PII redaction unit tests
        run: |
          pytest tests/redaction -q

      - name: Build embeddings (staging)
        run: |
          python ingestion/build_embeddings.py --model $EMBEDDING_MODEL --out staging/embeddings.jsonl

      - name: Create index snapshot
        run: |
          python infra/create_index_snapshot.py --input staging/embeddings.jsonl --out snapshots/index-$(date +%Y%m%d%H%M%S).tar.gz
        env:
          SNAPSHOT_BUCKET: ${{ env.INDEX_SNAPSHOT_BUCKET }}

      - name: Run automated checks on embeddings
        run: |
          python infra/checks/verify_provenance.py staging/embeddings.jsonl

      - name: Generate draft pages via LLM router
        run: |
          python generation/generate_drafts.py --index staging/embeddings.jsonl --router $LLM_ROUTER_ENDPOINT --out drafts/
        env:
          LLM_API_KEY: ${{ secrets.LLM_API_KEY }}

      - name: Run automated content checks
        run: |
          python infra/checks/verify_citations.py drafts/ || exit 1
          python infra/checks/verify_no_pii.py drafts/ || exit 1

      - name: Create Pull Request for human review
        uses: peter-evans/create-pull-request@v5
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          commit-message: "RAG: new index snapshot and generated drafts"
          branch: rag/generated-$(date +%s)
          title: "RAG Generated Drafts for Review"
          body: |
            Automated RAG generation completed. This PR includes:
            - index snapshot metadata
            - generated drafts (evidence attached)
            - automated check results
```

---

### ArgoCD Application YAML snippet

**Purpose** Deploy the indexer and retriever services and ensure GitOps sync. Replace `repoURL` and `path` with your repo values.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: rag-indexer
  namespace: argocd
spec:
  project: default
  source:
    repoURL: 'git@github.com:your-org/rag-infra.git'
    targetRevision: main
    path: k8s/indexer
  destination:
    server: 'https://kubernetes.default.svc'
    namespace: rag
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: rag-retriever
  namespace: argocd
spec:
  project: default
  source:
    repoURL: 'git@github.com:your-org/rag-infra.git'
    targetRevision: main
    path: k8s/retriever
  destination:
    server: 'https://kubernetes.default.svc'
    namespace: rag
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

---

### PII Redaction Rules and Pipeline Step

**Purpose** Detect and redact sensitive data before indexing. Rules are conservative and logged for audit.

#### Core redaction regex rules

- **Email**: `(?i)[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}`
- **Phone**: `(?:(?:\+?\d{1,3})?[\s\-\.]?)?(?:\(?\d{2,4}\)?[\s\-\.]?)?\d{3,4}[\s\-\.]?\d{3,4}`
- **National ID** (generic EU style): `\b\d{6,10}[A-Z0-9]?\b`
- **Credit Card**: `\b(?:\d[ -]*?){13,19}\b`
- **IBAN**: `\b[A-Z]{2}[0-9]{2}[A-Z0-9]{1,30}\b`
- **Postal Address heuristics**: street keywords `\b(street|st|road|rd|avenue|ave|g\.|gatvė|ulica)\b` combined with number token
- **Date of birth**: `\b(19|20)\d{2}[-\/\.]\d{1,2}[-\/\.]\d{1,2}\b` and `\b\d{1,2}[-\/\.]\d{1,2}[-\/\.](19|20)\d{2}\b`

#### Redaction pipeline pseudocode

```python
def redact_text(text, source_id):
    redaction_log = []
    for rule_name, pattern in REDACTION_RULES.items():
        for match in re.finditer(pattern, text, flags=re.IGNORECASE):
            span = match.span()
            snippet = text[span[0]:span[1]]
            text = text[:span[0]] + f"[REDACTED:{rule_name}]" + text[span[1]:]
            redaction_log.append({
                "source_id": source_id,
                "rule": rule_name,
                "original": snippet,
                "timestamp": now_iso()
            })
    store_redaction_log(redaction_log)
    return text
```

#### Operational controls

- **Redaction policy**: redact by default for PII classes above. Allow encrypted storage of original only when explicit legal justification exists and with strict access controls.
- **Audit**: every redaction event writes to an append‑only audit store with `source_id`, `rule`, `operator`, and `timestamp`. Retention governed by policy.
- **Human review**: flagged redactions create a ticket for regional compliance officer if content appears to be critical record (e.g., burial record) so a controlled release process can be followed.

---

### Prompt Template Bundle

**Purpose** Standardized prompts to enforce evidence use, citation, and insufficient evidence behavior.

#### Evidence Injection Template

```
SYSTEM:
You are a factual content generator. Use ONLY the evidence blocks provided below. For every factual claim include a citation token [SOURCE:id]. If evidence is insufficient to answer, respond exactly with: INSUFFICIENT_EVIDENCE.

EVIDENCE:
{{#each evidence_chunks}}
--- CHUNK ID: {{id}} SOURCE: {{source_url}} DATE: {{date}} LANG: {{lang}}
{{text}}
--- END CHUNK
{{/each}}

USER INSTRUCTION:
Produce a factual page for: {{entity_name}}.
Language: {{language}}
Required sections: Summary; Address; Service Times; Historical Notes; Sources.
Constraints: Do not invent dates, names, or addresses. Cite each factual sentence with [SOURCE:id].
```

#### Constrained Generation Template for LLM Router

```
TASK:
Generate a draft page using the evidence provided. Strict rules:
1. Use only the evidence chunks.
2. Prepend each factual sentence with its citation token [SOURCE:id].
3. If a claim cannot be supported by evidence, do not assert it.
4. Output JSON with fields: title, summary, sections[], citations[].

INPUT:
{{evidence_payload_json}}

OUTPUT FORMAT:
{
  "title": "...",
  "summary": "... [SOURCE:id]",
  "sections": [
    {"heading":"Address","content":"... [SOURCE:id]"},
    {"heading":"Service Times","content":"... [SOURCE:id]"}
  ],
  "citations": [
    {"id":"id","url":"...","date":"..."}
  ]
}
```

#### Verification and Human Review PR Template

```
PR TITLE:
[RAG REVIEW] Draft for {{entity_name}} - evidence snapshot {{snapshot_id}}

PR BODY:
Automated RAG generation created this draft. Review checklist:
- [ ] Verify each [SOURCE:id] citation matches the claim.
- [ ] Confirm no PII was improperly redacted or removed.
- [ ] Approve historical text for liturgical accuracy.
- [ ] If legal or burial records are present, escalate to compliance.

Attached artifacts:
- index snapshot: {{snapshot_url}}
- retrieved chunks: {{retrieved_chunks_url}}
- generated draft: {{draft_url}}
- automated check report: {{report_url}}

If approved, merge to trigger deployment pipeline.
```

---

### Deployment Notes and Quick Checklist

- **Pin model versions** for embeddings and LLMs and record them in every index snapshot.
- **Immutable snapshots**: store index snapshots as immutable artifacts and reference them in PRs.
- **Audit everything**: retrieval queries, top_k results, prompt used, model response, and reviewer decision must be logged.
- **DR plan**: daily encrypted snapshots, weekly full exports, and tested restore runbook.
- **SLA**: define retrieval latency SLO (e.g., 200ms p95) and monitor.
- **Access control**: enforce least privilege for index write operations; require approval for any unredaction.

---

If you want, I will now:

- produce **GitHub Actions YAML files** for the full pipeline (multiple workflows) and **ArgoCD Application manifests** for each microservice, or
- generate **sample Python scripts** for the redaction step and the verification checks referenced above.

State which artifact you want next and I will produce it immediately.