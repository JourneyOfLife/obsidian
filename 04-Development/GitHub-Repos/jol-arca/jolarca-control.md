
## 2026-10-07 — A-01 visibility audit (read-only)

**Method:** `gh api orgs/jolarca-dev/repos --paginate` + per-repo unauthenticated
`curl` fetch + declaration read from `repos/*.yml`. Evidence:
`[evidence]/{live-repos.jsonl,noauth-probe.txt,validate.out,compliance.out}`.
Verified by (operator, before paste): re-run commands in the Verification section.

| Repository | Declared visibility (`repos/*.yml`) | Declared `data_classification` | Live visibility | Live `forks_count` | Contradicts declared classification? | Finding |
|---|---|---|---|---|---|---|
| .github                       | (not in registry, HEALTH_REPO) | n/a | public  | 0 | No (out of registry by design) | D-43 correction |
| jolarca                       | public                         | public       | public  | 0 | No                               | — |
| jolarca-compliance            | public                         | internal     | public  | 0 | **Masked — content is `confidential`** | D-31 (OPEN/S1), D-52 |
| jolarca-consent               | public                         | internal     | public  | 0 | No                               | — |
| jolarca-control               | public                         | internal     | public  | 0 | **README says "private and internal"** — doc drift, YAML+live agree | A-01 |
| jolarca-data                  | public                         | internal     | public  | 0 | **Masked — content is `confidential`** | D-31 (OPEN/S1), D-52 |
| jolarca-docs                  | public                         | public       | public  | 0 | No                               | — |
| jolarca-dr                    | public                         | internal     | public  | 0 | No                               | — |
| jolarca-hermes-agents         | public                         | internal     | public  | 0 | No                               | — |
| jolarca-identity              | public                         | internal     | public  | 0 | No                               | D-34 (identity re-privating reverted) |
| jolarca-infrastructure        | public                         | internal     | public  | 0 | **Masked — content is `confidential`** | D-31 (OPEN/S1), D-52 |
| jolarca-legal                 | public                         | internal     | public  | 0 | **Masked — content is `confidential`** | D-31 (OPEN/S1), D-52 |
| jolarca-observability         | public                         | internal     | public  | 0 | No                               | — |
| jolarca-payments              | public                         | internal     | public  | 0 | No                               | — |
| jolarca-runbooks              | public                         | internal     | public  | 0 | No                               | — |
| jolarca-security              | public                         | internal     | public  | 0 | No                               | — |
| jolarca-vendor                | public                         | internal     | public  | 0 | No                               | — |

### A-01 · README of `jolarca-control` claims "private and internal" — **NEW, S3 (doc accuracy)**
| | |
|---|---|
| Evidence | `README.md:219` reads "This repository is **private and internal**"; `repos/jolarca-control.yml:21` declares `visibility: public`; live API returns `visibility: public` and unauthenticated `GET https://github.com/jolarca-dev/jolarca-control` returns HTTP 200. The YAML and the live object agree — the README is the outlier. |
| Impact | Breaches AGENTS.md §6 ("documentation must state actual implementation status"). Anyone reading the README assumes server-side confidentiality that does not exist. Compounds D-31's story (regulated-adjacent content publicly readable) — an auditor who trusts the README concludes the exposure was accidental; the YAML and D-37 record show it was a forced, accepted decision. |
| Required | Edit `README.md` §License to state the current declared+live posture (`public`, `data_classification: internal`, PCI-DSS risk acceptance in `policy/compliance-gates.yml`), with a pointer to ADR-0007 and D-37/D-52. Not applied here — this step is read-only. |
| Related | D-01 (resolved by D-37), D-31/D-52 (masked classification), ADR-0007 (public marketplace repo). |

### Cross-reference — declared-classification vs content (existing findings, unchanged)
D-31 remains **S1 BLOCKING OPEN** per its 2026-10-06 correction. D-52 records that
`confidential → internal` in D-37 was made *to stop the validator flagging the
exposure*; the four masked rows above are exactly those repos. `compliance_check.py`
still reports `public_but_sensitive: []` because it enforces only
`CLASSIFICATIONS_REQUIRING_PRIVATE = {confidential, restricted}` — a green validator
result is therefore NOT evidence the classifications are honest.

### Verification commands (reproduce every row)
```bash
# 1. Live visibility, fork count, updated_at
gh api orgs/jolarca-dev/repos --paginate \
  --jq '.[] | {name, visibility, private, fork_count: .forks_count, updated_at}'

# 2. Unauthenticated confirmation (public = no token required)
for r in jolarca-control jolarca-compliance jolarca-data jolarca-infrastructure \
         jolarca-legal jolarca-identity jolarca-security; do
  printf '%-24s api=%s html=%s\n' "$r" \
    "$(curl -sS -o /dev/null -w '%{http_code}' https://api.github.com/repos/jolarca-dev/$r)" \
    "$(curl -sS -o /dev/null -w '%{http_code}' https://github.com/jolarca-dev/$r)"
done

# 3. Declared side
for f in repos/*.yml; do
  printf '%-28s visibility=%-8s data_classification=%s\n' \
    "$(basename "$f" .yml)" \
    "$(awk '/^visibility:/{print $2;exit}' "$f")" \
    "$(awk '/^  data_classification:/{print $2;exit}' "$f")"
done

# 4. Validators (both must exit 0 for the registry, exit 1 for real drift)
.venv/bin/python scripts/validate_repos.py
.venv/bin/python scripts/compliance_check.py | grep -A1 public_but_sensitive

