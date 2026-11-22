





## Repository inventory — 28 total

### `/opt/jol/repos` — **22 repositories** (Church Platform tree)

| # | Repository | Branch | # | Repository | Branch |
|---|---|---|---|---|---|
| 1 | jol-analytics-ai | main | 12 | jol-hub | feat/pages-step6 |
| 2 | jol-auth | main | 13 | jol-infrastructure | docs/db-pilot-tenant-isolation-delta |
| 3 | jol-backend-platform | main | 14 | jol-link-registry | main |
| 4 | jol-bitrix24-integration | main | 15 | jol-llm | main |
| 5 | jol-compliance | feature/initial-setup | 16 | jol-mcp-servers | fix/ci-pipeline |
| 6 | jol-core | main | 17 | **jol-qoder-history** | main |
| 7 | jol-devops | main | 18 | jol-rag-server | main |
| 8 | jol-domain-taxonomy | main | 19 | jol-repo-template | main |
| 9 | jol-ecommerce-engine | main | 20 | jol-scripts | main |
| 10 | jol-frontend-platform | main | 21 | jol-security | main |
| 11 | jol-hermes-agents | main | 22 | obsidian | main |

### `/opt/jol-m/repos` — **6 repositories** (Marketplace tree)

| # | Repository | Branch |
|---|---|---|
| 1 | jol-m-compliance | eng/step30-retention-code |
| 2 | jol-m-data | main |
| 3 | jol-m-infrastructure | main |
| 4 | jol-m-legal | compliance/step26-vmi-isaf-record |
| 5 | jol-m-marketplace | main |
| 6 | **jol-m-qoder-history** | main |

Notes: all 28 are git repositories (no bare directories). Both bolded history repos are the ones we created today. The marketplace one needed a scoped `safe.directory` exception after your ownership alignment to `jolm-app:jolm-dev` — verified healthy now (`main`, commit `52701da`).