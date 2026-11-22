
Qoder jol-infra
---
---
---

```complete. Final summary: what I independently verified, what I recorded, disposition, conditions. Note the honest discrepancies I discovered (couldn't just agree with "everything's fixed" — F1/F2/F4 are not code-fixed on HEAD; I classified them as containment/deferred and recorded them together with the preconditions). Also mention the backup file DECISION-LOG.md.bak.<timestamp> and the O-007 correction. Keep it concise.
```
## Done — professional opinion recorded, with independent re-verification

I did not simply endorse "fixed and verified" — I re-verified on HEAD `bcb00f2f` (`feat/pages-step6`) and recorded exactly what the evidence supports.

### Independent verification evidence (executed today)
- **vitest**: 114/114 PASS (7 files) in `template-renderer`
- **tsc --noEmit**: exit 0, clean
- **F6 trace**: every HTTP surface (`sitemap.ts`, tenant `layout.tsx`, `route-dispatch.tsx`) uses the **closed** `loadTenantFixture`; the fallback variant has zero HTTP call sites — cross-tenant leakage structurally impossible
- **F3 count**: [redacted] pilot tenants = ratified 31-site hierarchy + VIP cathedral (matches DECISION-LOG O-002, CLOSED)

### What I recorded
1. **[AUDIT-CLOSURE-PROFESSIONAL-OPINION-20260826.md]([repo]/jol-hub/docs/decisions/AUDIT-CLOSURE-PROFESSIONAL-OPINION-20260826.md)** — the certification statement: the audit is remediated sufficiently for **Wave 0 stakeholder showcase and Wave 1 onboarding to proceed**, with per-finding dispositions (F3/F6 CLOSED, F1 CLOSED-AS-DEFERRED, F2 CONTAINED, F4/F5 OPEN-LOW) and four **conditions precedent to public go-live** on `[pilot-domain]`.
2. **[DECISION-LOG.md]([repo]/jol-hub/docs/decisions/DECISION-LOG.md)** — appended the audit-closure reference and new row **O-007**, a correction I had to make: O-005 claimed the `parish-template` directory was removed, but it **still exists** on disk (the referenced legacy directory is still on disk; the log entry was inaccurate). Backup taken first: `DECISION-LOG.md.bak.<timestamp>` per §0.3.

### Honest residuals (the opinion is conditional, not unconditional)
- **F1**: a legacy livestream component with a client-side auth defect survives in a deprecated app; no consent-gated livestream module exists yet in the renderer. Live-streaming must not be sold to funeral tenants until that module ships. Component name tracked privately.
- **F2**: legacy apps are contained by D-004 (single public renderer) but still launchable — delete or `DEPRECATED`-banner before public exposure.
- **F4**: an env-template default for the tenant base-domain config remains; one-line fix open. Variable name tracked privately.

Change control: documentation-only mutations; rollback = revert the recording commit. Next steps when you're ready: the F4 one-liner + legacy-app deprecation, and the O-003 `jol-db-pilot-lt01` spec delta in `jol-infrastructure` (your current branch suggests that's already in flight).







---

Organization: "Journey Of Life" (JOL) - Roman Catholic Digital Mission Platform ~400000 websites developing platform 27 European Union https://github.com/orgs/journeyoflife-org/repositories https://github.com/JourneyOfLife?tab=repositories Role> You must formulate it professionally and correctly. Optimize it to make it better PROMPT. Will you create a better version of this PROMPT that is more comprehensive, structured, and actionable; Ask me any questions you have before proceeding; You are a professional SEO, UI/UX designer, GitHub, DevOps specialist engineer with 30 years of experience.
Lithuania pilot Front-Ends for:
 through which ordered new web link us integration Django Bitrix24 CRM and more:
     Roman Catholic Basilicas (three templates cheap, normal and VIP) - please consider it in your professional opinion.
	 Roman Catholic Cathedrals (three templates cheap, normal and VIP) - please consider it in your professional opinion.
     Roman Catholic Dioceses + commercial (three templates cheap, normal and VIP) - please consider it in your professional opinion.
     Roman Catholic Diaconate + commercial 20% (three templates cheap, normal and VIP) - please consider it in your professional opinion.
     Roman Catholic Churches (three templates cheap, normal and VIP) - please consider it in your professional opinion.
     Protestant   Churches (three templates cheap, normal and VIP) - please consider it in your professional opinion.
     Russian Orthodox  Churches + commercial 20% (three templates cheap, normal and VIP) - please consider it in your professional opinion.
     Other Churches  - please consider it in your professional opinion.
     Funeral Homes + commercial + commercial (three templates cheap, normal and VIP) - please consider it in your professional opinion.
     Cemetery Graves and Monuments Cleaning Services + commercial (three templates cheap, normal and VIP) - please consider it in your professional opinion.
1. Three packages
Please professionally redefine the package boundaries, rather than simply accepting 10/15/20 pages;
For example:
Package
Indicative size
CRM
Commercial
Automation
CHEAP
~10 pages
Bitrix24
No/limited
Basic
NORMAL
~15 pages
Bitrix24
Optional
Standard
VIP
~20+ pages
Bitrix24
Yes
Advanced
AI to determine the optimal number of pages, features, and price for each package based on customer type.
 
UI/UX - please consider it in your professional opinion. examples: [pilot-tenant-url]/ [pilot-tenant-url]/[parish-a]/ [pilot-tenant-url]/[funeral-tenant-a]/ [pilot-tenant-url]/[cemetery-tenant-a]/
 
2. Website categories
You currently have these target verticals:
Roman Catholic Basilicas
Roman Catholic Cathedrals
Roman Catholic Dioceses
Roman Catholic Diaconate
Roman Catholic Churches
Protestant Churches
Russian Orthodox Churches
Other Churches
Funeral Homes
Cemetery Graves & Monument Cleaning Services
One JOL Design System + reusable component library + vertical-specific templates/content models
each website owner can control 10% of the content of the new images and 90% control JOL each website has a separate database. separate Bitrix24 CRM, each website subdomain https://[pilot-domain]/ Lithuania pilol is : Roman Catholic Churches websites ~ 700 Funeral Homes websites ~ 400 Cemetery Graves & Monument Cleaning Services ~200
professional expectation would be: ???
3. “Commercial” definition all whis and more:
https://github.com/journeyoflife-org/jol-ecommerce-engine service.
iOS Android.
AI Chat GPT.
Blockchain.
CRM.
E-Commerce.
Online ordering and delivery
Social Media Integration
27 Multilingual Support.
Live Streaming-Recordings
Boards in Chats, Tasks, Feed and Video calls.
Board for secure work
Sales scripts with AI.
Speech analytics with AI.
More
online service/product catalogue
enquiry/lead generation
quotation requests
booking
payments
e-commerce
advertising/sponsorship
marketplace integration
CRM sales pipeline
customer accounts
invoices/orders
Everything listed above has already been done on the back-end.
 
4. Bitrix24 architecture already been done;
https://github.com/journeyoflife-org/jol-bitrix24-integration
 
5. Django's role already been done
https://github.com/journeyoflife-org/jol-hub
6. Front-end technology - please add it in your professional opinion
Should the master prompt standardize on your existing architecture:
Next.js / React + TypeScript
with:
SSR/SSG/ISR where appropriate
reusable components
responsive design
Core Web Vitals
structured data/schema.org
multilingual SEO
WCAG 2.2 AA
CDN/cache strategy
tenant-aware routing The AI to evaluate Next.js against alternatives before selecting the final technology - please add it in your professional opinion
 
7. CMS/content editing - please add it in your professional opinion
will maintain the website content:
A. Church/organization administrators B. JOL administrators C. Both D. Bitrix24 users E. A dedicated JOL CMS F. Combination
This substantially affects the front end/backend architecture. (backend https://github.com/journeyoflife-org/jol-hub check if necessary please add it in your professional opinion )
 
8. The 10/15/20-page model - please add it in your professional opinion
Should “10 pages / 15 pages / 20 pages” should some functionality be reusable modules.
For example:
/about /services /news /events /contact
versus dynamic content such as:
/news/{slug} /events/{slug} /services/{slug}
This matters greatly for SEO and for scaling toward ~400,000 websites.
 
9. Multi-tenancy
Should every organization receive an isolated tenant such as:
planning domains/subdomains such as:
[tenant-slug].[pilot-domain]
I would like the final prompt to explicitly require the AI to design:
tenant → organization → website → domain → locale → template → content
if that matches your intended model. - please consider it in your professional opinion.
 
10. Languages
For the Lithuania pilot, should the prompt require:
Lithuanian first
English
Russian
Polish
 
11. SEO
Should the master prompt require a complete SEO architecture covering:
technical SEO
on-page SEO
local SEO
Google Business Profile integration
schema.org
Organization
Church
ReligiousOrganization
LocalBusiness where appropriate
FuneralHome
Service
Product
Event
BreadcrumbList
FAQ
sitemap architecture
hreflang
canonical URLs
robots.txt
Open Graph
Twitter/X cards
image SEO
Core Web Vitals
programmatic SEO
400,000-site indexing strategy
?
I recommend yes, especially because this is fundamentally a multi-tenant SEO platform.
 
12. UI/UX references - please consider it in your professional opinion.
You supplied these examples:
[pilot-tenant-url]
/[parish-a]/
/[funeral-tenant-a]/
/[cemetery-tenant-a]/
Should the final prompt tell the AI to critically audit these existing examples and identify:
what should be retained
what should be removed
what should be redesigned
what should become reusable components
what should become template-specific
what violates modern UX/SEO/accessibility/security practices
rather than simply copying their design?
 
13. Design system
Do you want the final architecture to establish a formal JOL design system such as:
JOL Design System → typography → colors → spacing → grid → buttons → navigation → cards → forms → calendars → events → news → galleries → maps → donations → services → testimonials → contact → footer → accessibility → mobile components
with Cheap / Normal / VIP being compositions of the same components?
 
14. Compliance scope - please consider it in your professional opinion.
You specified:
SOC 2 / GDPR / ISO 27001
Should the prompt treat these as architecture/design requirements, not merely documentation requirements?
For example:
GDPR privacy-by-design
data minimization
consent management
retention
deletion/export
audit logs
encryption
RBAC
tenant isolation
PostgreSQL RLS
secrets management
MFA
secure API
OWASP ASVS
logging/monitoring
incident response
backup/recovery
supply-chain security
vulnerability management
?
 
15. Commercial percentages
You mention:
Diaconate: 10% commercial
Russian Orthodox Churches: 10% commercial
Funeral Homes: commercial 10% commercial
Cemetery cleaning: commercial 10% commercial Does “10%” mean JOL commission = 10% of commercial transactions
 
16. Output expected from the final master prompt - please consider it in your professional opinion.
recommend making the resulting prompt require the AI to produce a structured deliverable like:
Executive architectural assessment
Business/product taxonomy
Website vertical matrix
Cheap/Normal/VIP package definition
Page/module matrix
UI/UX architecture
JOL Design System
Frontend architecture
Django/API architecture
Bitrix24 integration architecture
Database/content model
Multi-tenant architecture
Domain/subdomain architecture
SEO architecture
Accessibility architecture
Security architecture
GDPR/SOC 2/ISO 27001 controls
GitHub repository architecture
CI/CD architecture
Infrastructure/deployment architecture
Testing strategy
Observability
Backup/disaster recovery
Cost model
Scalability toward ~400,000 websites
Migration strategy from the existing pilot
Implementation phases
Prioritized backlog
P0/P1/P2 risks
Security / Compliance / Cost / Rollback-Fallback for every major recommendation
 
Please think twice. Acting as a paranoid compliance-driven architect, expert with 30+ years’ experience::Compliance Level: SOC2 / GDPR / ISO 27001

---
Before I draft the final master prompt, here is my read of the assignment, followed by the questions I need answered. I've grounded this in your verified estate: 100% on-prem Proxmox (no AWS), the <host> (Church) vs <host-m> (Marketplace) Tier-1 segregation, GDPR Art. 9 special-category scope, and the ratified Lithuania pilot specs (Steps 1–5, [pilot-domain] VM topology already documented in docs/servers/jol-*-pilot-lt01.md).
 
Part A — Preliminary professional positions (what I will bake into the prompt)
These I can already state with confidence; flag any you disagree with:
Package model: kill "10/15/20 pages." At 400,000-site scale, pages are the wrong unit. The correct unit is modules (static pages + dynamic collections like /news/{slug}, /events/{slug}). Cheap/Normal/VIP become capability tiers (modules × automation × commercial entitlements), not page counts. This is decisive for both SEO (programmatic pages) and cost.
Frontend: Next.js is the right candidate, but the prompt must force a scored evaluation (Next.js vs Astro vs Nuxt vs Django-templated) against your constraints — on-prem hosting, SSR for SEO, 27 languages, Core Web Vitals on modest hardware. The prompt will require a decision matrix, not a foregone conclusion.
"AI Chat GPT" must be re-scoped. OpenAI is a US processor; your ratified policy is EU-only providers (Mistral / OVH-AI) or on-prem inference via your Ollama/RAG stack. The prompt will require "AI assistant" as a capability, with the provider decision governed by your data-residency ADR — never a brand name.
Blockchain: GDPR Art. 17 conflict. Immutable ledgers cannot honor erasure. The prompt will constrain blockchain to off-chain hash anchoring (certificates, donation receipts) — no personal data on-chain.
CMS answer: option F (combination). JOL maintains 90% via the design system + structured content models; the 10% tenant-editable surface is a constrained block editor with moderation; Bitrix24 is the CRM/task layer, not the CMS. The prompt will require this to be validated against jol-hub.
The .bitrix24site.ru examples: audit, don't copy. A Russian TLD hosting a Lithuanian Catholic platform is an SEO-trust, brand, and sanctions-compliance problem. The prompt will require a keep/remove/redesign/componentize audit.
Pilot sizing reality check. Your verticals total ~1,300 LT addresses (700 churches + 400 funeral homes + 200 cleaning services). Professional expectation: pilot in waves — Wave 0: 3–5 reference sites (one per archetype: parish, funeral home, cleaning service); Wave 1: ~25; Wave 2: ~100; then industrialization. Deploying 1,300 tenants before the design system is proven is how platforms fail.
Multi-tenancy chain confirmed as the spine: tenant → organization → website → domain/subdomain → locale → template → content, with per-tenant database schema isolation (you already mandate separate DB per site) and RLS as defense-in-depth.
Compliance as architecture, not documentation — yes, as you proposed: GDPR privacy-by-design, PCI-DSS-scoped payment paths, tenant isolation, audit logs, RBAC, secrets management, all mapped to named controls (SOC 2 CC/ISO 27001 A.x/GDPR Articles).
 
Part B — Questions I need answered before writing the final prompt
Commercial & pricing
Confirm: "commercial 10%/20%" = JOL commission on commercial transactions processed through the tenant's site (not a content quota)? And is the commission rate per-vertical (Diaconate/Orthodox/Funeral/Cemetery at 10–20%)?
Are package prices to be proposed by the AI (EUR/month per tier per vertical), or do you have target price bands? Any anchoring (e.g., current Bitrix24 site costs)?
Scope & verticals 3. The "commercial" verticals (Dioceses, Diaconate, Orthodox, Funeral Homes, Cemetery services) — do they get the full commercial module list (e-commerce, booking, payments, marketplace), or a vertical-specific subset? E.g., cemetery cleaning = booking + payments + subscriptions; funeral homes = enquiry + booking + obituary notices + live streaming. 4. "Other Churches" — is this a catch-all template with denomination-agnostic content models (Greek Catholic, Evangelical, etc.), or a placeholder? 5. Live streaming: own infrastructure (your on-prem estate / EU provider) or embeds (YouTube/Vimeo)? This has GDPR implications for third-party trackers on church sites.
Domains & tenancy 6. Confirm the domain model: https://{tenant-slug}.[pilot-domain] subdomains for the pilot, with custom-domain (CNAME) support as a VIP entitlement later? Any other pilot domains? 7. Each tenant "separate database" — confirm this means separate logical database per tenant (your ratified pilot spec), i.e., schema-per-tenant is not acceptable?
Content & languages 8. Pilot languages: Lithuanian (primary) + English + Russian + Polish — all four required at pilot launch, or LT + one secondary first? (hreflang architecture differs.) 9. The 10% tenant-editable content: does that include images they upload (which then need moderation, malware scanning, Art. 9 review), or text blocks only?
Success criteria 10. What is the pilot's acceptance gate? (You have docs/runbooks/jol-pilot-acceptance-gate.md — should the prompt align deliverables to that gate, plus business KPIs like sites-onboarded, Lighthouse ≥ 90, conversion metrics?) 11. Timeline and budget envelope for the pilot, if any — the phased backlog should be time-boxed.
Deliverable format 12. Confirm the 30-section deliverable structure you listed (it is sound; I will refine it, merge overlaps, and add Security/Compliance/Cost/Rollback per recommendation as a hard requirement). One addition I propose: "Decision Log & Open Questions" as section 31 — acceptable?
 
Once you answer (or tell me to proceed with my recommended defaults — each question above has one), I will produce the complete master prompt: role definition, estate constraints, vertical taxonomy, package framework, mandatory deliverables with acceptance criteria, and the paranoid-compliance guardrails.

---

1. Conflict check — verified findings against <host>/repos/
Ground truth (verified):
jol-hub is a real integration monorepo: Django backend (apps: core, content, crm, organizations, users, donations, financial, integrations, countries, ai, analytics) + Turborepo/pnpm frontend with Next.js + Tailwind apps and shared packages (ui, i18n, auth, bitrix-sdk). 15 frontend apps already exist: admin-dashboard, master-site, parish-template, plus 12 hard-coded lt-* entity demos (basilica, cathedral, diocese, deanery, parish, funeral, cemetery, orthodox, lutheran, greek-catholic, monastery, chapel).
jol-frontend-platform / jol-backend-platform / jol-domain-taxonomy are stubs reserved for extraction from the monorepo.
Satellites are real: jol-ecommerce-engine (Stripe, PCI SAQ A), jol-bitrix24-integration, jol-auth (OAuth 2.1/OIDC), jol-compliance, jol-security.
countries/lt/ already has config/examples/templates.
Conflicts & risks the prompt must resolve (professional opinion):
Framework choice is no longer greenfield. Next.js already exists in jol-hub/frontend. The "scored evaluation" must therefore be framed as: confirm Next.js vs alternatives with a migration-cost column, and separately evaluate rendering strategy (SSG/ISR/SSR per page type). My opinion: Next.js stays; the real decision is single template-renderer app vs per-entity apps.
The 12 hard-coded lt-* demo apps contradict the tenant model. At 23 pilot → 400k sites, you cannot have one app per site. The prompt mandates migration to one template renderer + tenant resolution + data-driven content — the demo apps become seed fixtures.
README drift in jol-hub: README claims SQLAlchemy/Alembic + npm; reality is Django + pnpm. The prompt requires a doc-reconciliation task.
Hygiene findings (⚠ verify before pilot): untracked `.env`-style files and local SQLite/junk artifacts exist in the working tree of one or more pilot-relevant apps; specific paths/sizes tracked privately. Given your zero-tolerance secret policy, the prompt mandates a secret-scan + hygiene baseline before any frontend work.
DB model change: you ratified schema-per-tenant isolation (not separate DB per site) — this contradicts the earlier pilot spec (docs/servers/jol-db-pilot-lt01.md); the prompt forces an explicit ADR + spec-delta record.
Commission: you unified all commercial verticals at 10% (your original text said 20% for Orthodox). I've encoded 10% as ratified and forced the decision log to record the change.
Languages: you listed LT + EN + RU but wrote "all four" — I've encoded LT/EN/RU ratified, Polish as open question.
2. Final opinions on your remaining asks
Blockchain — confirmed: off-chain hash anchoring only (certificates, donation receipts); no personal data on-chain. Art. 17 makes anything else unlawful.
CMS option F — confirmed: JOL-controlled design system + structured content models; tenant 10% via constrained block editor with moderation queue; Bitrix24 = CRM/tasks, never CMS.
Acceptance gate — yes: bind deliverables to docs/runbooks/jol-pilot-acceptance-gate.md plus business KPIs (Lighthouse ≥ 90 mobile, CWV green, onboarding ≤ 1 day/site, zero Art. 9 leakage).
Section 31 Decision Log — approved and made mandatory.
3. The Master Prompt
Placement note: this is a product/frontend specification — per your repo rules it belongs in jol-hub (Tier 0), not jol-infrastructure (SECONDARY, infra only). I've targeted jol-hub/docs/decisions/:

++++

<host>/repos/jol-hub/docs/decisions/MASTER-PROMPT-LT-PILOT-FRONTEND.md
# MASTER PROMPT — JOL Lithuania Pilot: Multi-Tenant Front-End Platform

## 0. ROLE

You are a Principal Platform Architect with 30 years of combined SEO, UI/UX,
GitHub, and DevOps engineering experience, acting as a paranoid,
compliance-driven architect. Compliance level: SOC 2 Type II, GDPR
(including Art. 9 special-category data and Art. 17 erasure), ISO 27001:2022,
PCI-DSS (SAQ A via Stripe). You prioritize correctness over speed. Every
major recommendation MUST carry: (1) security impact, (2) compliance impact,
(3) cross-repository dependency impact, (4) rollback/fallback strategy.
If you cannot verify a claim, state: `⚠ UNVERIFIED — manual check required`.

## 1. ORGANIZATION & MISSION

Journey Of Life (JOL) — Roman Catholic digital mission platform, target
~400,000 websites across 27 EU member states. White-label, multi-tenant
publishing system for religious institutions, funeral services, cemetery
care, and memorial commerce. GitHub org: journeyoflife-org.

## 2. RATIFIED NON-NEGOTIABLES

1. **Estate is 100% on-prem Proxmox.** No AWS/GCP assumptions. New capacity
   arrives as purchased hardware (next: Dell R640). CDN/edge strategy must be
   EU-jurisdiction or self-hosted caching.
2. **Two Tier-1 trees never merge**: `<host>` (Church Platform: GDPR Art. 9,
   PCI-DSS donations, clergy/parishioner PII) and `<host-m>` (Marketplace:
   payments, KYC/AML, VAT OSS). Marketplace integration features require a
   DPIA and cross-tree segregation review before design.
3. **Secrets**: never in code, repos, or CLI args. Vaultwarden / Ansible
   Vault / cloud-init only. GitHub Secrets for CI tokens only.
4. **AI providers**: never name a brand as a requirement. AI capability is
   served by the on-prem Ollama/RAG stack (`llm-prod-lt01`, `rag-prod-lt01`)
   first; any external provider must be EU-jurisdiction and ADR-approved.
5. **Blockchain**: off-chain hash anchoring ONLY (certificates, donation
   receipts). No personal data on-chain — immutable ledgers violate
   GDPR Art. 17.
6. **Live streaming**: YouTube embeds ONLY, behind a two-click / consent-mode
   gate (no cookies before consent). Native streaming infrastructure is
   out of pilot scope.
7. **Change control (SOC 2 CC8.1)**: every production change gets an issue,
   snapshot, `.bak.<timestamp>`, CHANGELOG entry, and rollback plan.

## 3. VERIFIED ESTATE (GROUND TRUTH — DO NOT CONTRADICT)

| Repository | Role | Verified facts |
|---|---|---|
| `jol-hub` | Tier 0 integration monorepo | Django backend (apps: core, content, crm, organizations, users, donations, financial, integrations, countries, ai, analytics). Turborepo/pnpm/Next.js/Tailwind frontend: apps `admin-dashboard`, `master-site`, `parish-template` + 12 `lt-*` entity demos; shared packages `ui`, `i18n`, `auth`, `bitrix-sdk`. `countries/lt/` exists. |
| `jol-auth` | OAuth 2.1 / OIDC | Identity for all tenants; HS256→RS256 trajectory. |
| `jol-ecommerce-engine` | Commerce | Stripe ~99% integrated, PCI-DSS SAQ A scope. |
| `jol-bitrix24-integration` | CRM layer | Bitrix24 = CRM/sales-pipeline/tasks/telephony. NOT a CMS. 90-day token rotation. |
| `jol-frontend-platform` | STUB | Reserved for frontend extraction from jol-hub. |
| `jol-rag-server`, `jol-llm`, `jol-mcp-servers` | AI estate | On-prem inference; EU-only external fallback. |
| `jol-infrastructure` | Tier 4 SECONDARY | NEVER contains application logic; hosts pilot VM specs (`docs/servers/jol-*-pilot-lt01.md`) and acceptance gate (`docs/runbooks/jol-pilot-acceptance-gate.md`). |

Known conflicts you MUST resolve, not inherit:
- The 12 hard-coded `lt-*` demo apps violate the tenant model → become seed
  fixtures; runtime must be ONE template renderer + tenant resolution.
- `jol-hub` README drift (claims SQLAlchemy/Alembic/npm; reality Django/pnpm)
  → reconciliation task.
- Pre-work hygiene gate: secret-scan (`git-secrets`, `trufflehog`) the jol-hub
  working tree (`.env`, `.env.local` files present), remove stray SQLite
  artifacts (`counter_*.db`, `histogram_*.db`) and junk files, BEFORE any
  feature work.
- DB model ratified as **schema-per-tenant (PostgreSQL) + RLS defense-in-depth**,
  NOT database-per-site → produce ADR and delta against
  `jol-infrastructure/docs/servers/jol-db-pilot-lt01.md`.

## 4. PILOT SCOPE — LITHUANIA ([DIOCESE-CLUSTER])

Waves (ratified): Wave 0: 3–5 reference sites for presentation to Bishops,
priests, funeral homes, cleaning companies → Wave 1 ≈ 25 sites (below) →
Wave 2 ≈ 100 → industrialization to ~1,300 LT addresses (700 churches,
400 funeral homes, 200 cleaning services), then 27 EU countries.

Wave 1 structure (23 sites):
- 1 Diocese site: [diocese-cluster]
- 6 Deanery clusters, each = Deanery site + 1 church + 1 funeral home +
  1 cleaning service:
  - [diocese-city] Deanery | [diocese-city] funeral | [diocese-city] cleaning (+ [diocese-city] church)
  - [deanery-b] Deanery | [parish-b] | [tenant-b] funeral | [tenant-b] cleaning
  - [deanery-c] Deanery | [parish-c] | [tenant-c] funeral | [tenant-c] cleaning
  - [deanery-d] Deanery | [parish-d] | [tenant-d] funeral | [tenant-d] cleaning
  - [deanery-e] Deanery | [parish-e] | [tenant-e] funeral | [tenant-e] cleaning

All pilot sites: `https://{tenant-slug}.[pilot-domain]` subdomains.
Legacy `*.bitrix24site.ru` sites are retired when JOL hosting starts on the
Dell R640 — design a 301-migration + SEO equity transfer plan, do not copy
their design; produce a keep/remove/redesign/componentize audit of:
[pilot-tenant-url], /[parish-a]/, /[funeral-tenant-a]/,
/[cemetery-tenant-a]/.

## 5. VERTICAL TAXONOMY (10 VERTICALS)

1. Roman Catholic Basilicas · 2. Cathedrals · 3. Dioceses (+commercial) ·
2. Diaconate (+commercial) · 5. Roman Catholic Churches · 6. Protestant
Churches · 7. Russian Orthodox Churches (+commercial) · 8. Other Churches
(denomination-agnostic catch-all placeholder template) · 9. Funeral Homes
(+commercial) · 10. Cemetery Graves & Monument Cleaning (+commercial).

One JOL Design System + reusable component library + vertical-specific
templates and content models. Commercial entitlements are vertical-specific
subsets, e.g. cemetery cleaning = booking + payments + subscriptions +
e-commerce (flowers, bouquets); funeral homes = enquiry + booking + obituary
notices + live-stream embeds + e-commerce (coffins, urns, vestments).
Produce the full vertical × module entitlement matrix.

## 6. PACKAGE MODEL — CAPABILITY TIERS (PAGE COUNTS ARE BANNED)

The unit of composition is the **module** (static page OR dynamic collection
such as `/news/{slug}`, `/events/{slug}`, `/services/{slug}`). Packages are
modules × automation × commercial entitlements:

| Package | Setup fee | Recurring | Commercial |
|---|---|---|---|
| CHEAP | €1,000 one-off | €20/mo Bitrix24 CRM Phone app | No/limited |
| NORMAL | €2,000 one-off | €20/mo Bitrix24 CRM Phone app | Optional |
| VIP | €3,000 one-off | €20/mo Bitrix24 CRM Phone app | Full |

Prices are RATIFIED — do not change them; you may (must) justify tier
boundaries by module entitlements, automation level, and support SLA.

## 7. COMMERCIAL MODEL

JOL commission = **10%** of commercial transactions processed through the
tenant's site, uniform across all commercial verticals (Diaconate, Orthodox,
Funeral Homes, Cemetery services). Record in Decision Log that this
supersedes earlier 20% statements. Commercial capabilities already built in
`jol-ecommerce-engine` / backend: catalogue, lead-gen, quotations, booking,
payments (Stripe SAQ A), subscriptions, customer accounts, invoices/orders,
CRM sales pipeline, multilingual (27), social integration, AI assistant,
AI sales scripts/speech analytics, boards (chats/tasks/feed/video), secure
work board, iOS/Android. Map each capability to its owning repository and
pilot phase; flag anything not yet built as `⚠ UNVERIFIED`.

## 8. MULTI-TENANCY CONTRACT (THE SPINE)

Design the full chain: **tenant → organization → website → domain/subdomain
→ locale → template → content**. Requirements:
- Content ownership: JOL controls 90% (design system + structured content
  models); tenant edits 10% via a constrained block editor with moderation.
- Tenant uploads (images included) pass: malware scan (ClamAV-class),
  on-prem AI moderation (via the Ollama/RAG stack), Art. 9 review queue.
- CMS model = option F (combination): JOL admins + tenant admins + Bitrix24
  as CRM/task layer (never CMS). Validate against `jol-hub` backend apps.
- Data isolation: PostgreSQL **schema-per-tenant** + row-level security as
  defense-in-depth; per-site logical separation; Art. 17 erasure via
  `deleted_at` logical deletion + legal sign-off + erasure log per
  `jol-infrastructure/docs/compliance/erasure-log.md`.
- Tenant resolution at the edge/ingress: subdomain → tenant → template
  variant → locale → content, cacheable per (tenant, locale, template).

## 9. FRONTEND ARCHITECTURE MANDATE

Produce a **scored decision matrix** (weighted criteria: on-prem hosting fit,
SSR/SEO, i18n at 27 locales, Core Web Vitals on modest hardware, team
existing investment, migration cost, security surface) comparing Next.js vs
Astro vs Nuxt vs Django-templated — with the verified jol-hub Next.js
investment as the incumbent column. Then decide, per page class, the
rendering strategy (SSG/ISR/SSR) and the app topology: ONE template-renderer
application + tenant resolution vs per-entity apps (the latter is the
anti-pattern to migrate away from). Mandatory: WCAG 2.2 AA, structured
data, responsive design, tenant-aware routing, image optimization pipeline.

## 10. SEO ARCHITECTURE (MANDATORY — THIS IS AN SEO PLATFORM)

Full architecture: technical SEO; on-page; local SEO; Google Business
Profile integration; schema.org (ReligiousOrganization, Church,
Organization, LocalBusiness, FuneralHome, Service, Product, Event,
BreadcrumbList, FAQPage); sitemap architecture per tenant; hreflang
(LT/EN/RU); canonicals; robots.txt per tenant; Open Graph + X cards; image
SEO; Core Web Vitals budgets; programmatic SEO; and the 400,000-site
indexing strategy (crawl-budget management, sitemap sharding, index-now,
per-tenant search-console onboarding).

## 11. LANGUAGES

Pilot: Lithuanian (primary), English, Russian — all three at pilot launch.
Polish is an OPEN QUESTION (record in Decision Log). Architecture must
support 27 EU locales without redesign (`packages/i18n` extension path).

## 12. JOL DESIGN SYSTEM

Define formally: typography, color, spacing, grid, buttons, navigation,
cards, forms, calendars/events, news, galleries, maps, donations, services,
testimonials, contact, footer, accessibility primitives, mobile components.
CHEAP/NORMAL/VIP are **compositions of the same components** — no VIP-only
parallel codebase. Audit the four legacy bitrix24site.ru examples:
retain / remove / redesign / componentize / template-specific / violates-
modern-practice buckets.

## 13. COMPLIANCE AS ARCHITECTURE (NOT DOCUMENTATION)

Every control must be enforceable in code/config: GDPR privacy-by-design,
data minimization, consent management (cookie/YouTube gates), retention
schedules, Art. 15/17/20 export/erasure, audit logs, encryption in transit
+ at rest (ZFS aes-256-gcm verified on pilot VMs), RBAC, tenant isolation
(schema + RLS), secrets management (Vaultwarden), MFA, OWASP ASVS L2,
logging/monitoring, incident response (<72 h breach notification),
backup/DR (PBS, RPO 24 h/RTO 4 h pilot baseline), supply-chain security,
vulnerability management. Map each to named controls: SOC 2 CC/A-series,
ISO 27001:2022 Annex A, GDPR Articles. DPIA triggers: church+marketplace
data sharing, AI content generation, tenant upload moderation.

## 14. MANDATORY DELIVERABLES (ALL 31)

1. Executive architectural assessment · 2. Business/product taxonomy ·
2. Website vertical matrix · 4. CHEAP/NORMAL/VIP capability-tier definition ·
3. Module matrix (static vs dynamic collections) · 6. UI/UX architecture ·
4. JOL Design System · 8. Frontend architecture (incl. scored framework
matrix + rendering strategy) · 9. Django/jol-hub API architecture ·
5. Bitrix24 integration architecture · 11. Database/content model
(schema-per-tenant ADR) · 12. Multi-tenant architecture · 13. Domain/
subdomain architecture · 14. SEO architecture · 15. Accessibility (WCAG 2.2
AA) architecture · 16. Security architecture · 17. GDPR/SOC 2/ISO 27001
control map · 18. GitHub repository architecture (incl. jol-frontend-platform
extraction decision) · 19. CI/CD architecture · 20. Infrastructure/deployment
(on-prem Proxmox; Dell R640 capacity) · 21. Testing strategy ·
6. Observability · 23. Backup/DR · 24. Cost model · 25. Scalability path to
~400,000 sites · 26. Migration strategy (legacy bitrix24site.ru → JOL, 301s,
SEO equity) · 27. Implementation phases (Wave 0 → 1 → 2 → industrialization) ·
7. Prioritized backlog · 29. P0/P1/P2 risk register · 30. Per-recommendation
Security/Compliance/Cost/Rollback annex · 31. Decision Log & Open Questions
(must record: 10% commission unification; schema-per-tenant ratification;
PL language open; framework matrix outcome).

Every deliverable maps to owning repository(ies) from §3 and states what is
already built vs new work.

## 15. ACCEPTANCE GATE

Bind all deliverables to
`jol-infrastructure/docs/runbooks/jol-pilot-acceptance-gate.md` plus
business KPIs: Lighthouse ≥ 90 (mobile) per pilot site; Core Web Vitals
green; tenant onboarding ≤ 1 working day/site; zero Art. 9 data leakage in
cross-tenant tests; all 23 Wave-1 sites live on `[pilot-domain]`
before legacy retirement. Timeline: as soon as possible — therefore phase
the backlog so Wave 0 reference sites ship first.

## 16. OUTPUT RULES

- Structured Markdown; tables for matrices; no hand-waving.
- Every major recommendation: security impact / compliance impact /
  cross-repo impact / rollback strategy.
- Unverified claims marked `⚠ UNVERIFIED — manual check required`.
- No brand-name AI providers as requirements; no AWS assumptions;
  no page-count package definitions; no on-chain personal data.
- Respect repo sovereignty: application logic lives in PRIMARY repos
  (jol-hub and satellites); jol-infrastructure gets infra only; secrets
  never in inventory or code.
---
Next step: approve this file and I'll also produce the matching ADR stubs it references (schema-per-tenant; frontend extraction into jol-frontend-platform), and the pre-work hygiene/secret-scan checklist for the jol-hub working tree — that gate should run before any frontend feature work starts.


---
---

docs/decisions/ doesn't exist yet in jol-hub, and I've confirmed the house ADR format. Here are the four approved artifacts — apply them in order:
<font color="#f79646">1. The approved master prompt (new file):</font>
<font color="#f79646"><host>/repos/jol-hub/docs/decisions/MASTER-PROMPT-LT-PILOT-FRONTEND.md</font>
# MASTER PROMPT — JOL Lithuania Pilot: Multi-Tenant Front-End Platform

> Ratified: 2026-08-24 · Owner: JOL Principal Platform Architect · Companion docs: ADR-001, ADR-002 in this directory; hygiene gate: `docs/compliance/pre-work-hygiene-gate.md`.

## 0. ROLE

You are a Principal Platform Architect with 30 years of combined SEO, UI/UX,
GitHub, and DevOps engineering experience, acting as a paranoid,
compliance-driven architect. Compliance level: SOC 2 Type II, GDPR
(including Art. 9 special-category data and Art. 17 erasure), ISO 27001:2022,
PCI-DSS (SAQ A via Stripe). You prioritize correctness over speed. Every
major recommendation MUST carry: (1) security impact, (2) compliance impact,
(3) cross-repository dependency impact, (4) rollback/fallback strategy.
If you cannot verify a claim, state: `⚠ UNVERIFIED — manual check required`.

## 1. ORGANIZATION & MISSION

Journey Of Life (JOL) — Roman Catholic digital mission platform, target
~400,000 websites across 27 EU member states. White-label, multi-tenant
publishing system for religious institutions, funeral services, cemetery
care, and memorial commerce. GitHub org: journeyoflife-org.

## 2. RATIFIED NON-NEGOTIABLES

1. **Estate is 100% on-prem Proxmox.** No AWS/GCP assumptions. New capacity
   arrives as purchased hardware (next: Dell R640). CDN/edge strategy must be
   EU-jurisdiction or self-hosted caching.
2. **Two Tier-1 trees never merge**: `<host>` (Church Platform: GDPR Art. 9,
   PCI-DSS donations, clergy/parishioner PII) and `<host-m>` (Marketplace:
   payments, KYC/AML, VAT OSS). Marketplace integration features require a
   DPIA and cross-tree segregation review before design.
3. **Secrets**: never in code, repos, or CLI args. Vaultwarden / Ansible
   Vault / cloud-init only. GitHub Secrets for CI tokens only.
4. **AI providers**: never name a brand as a requirement. AI capability is
   served by the on-prem Ollama/RAG stack (`llm-prod-lt01`, `rag-prod-lt01`)
   first; any external provider must be EU-jurisdiction and ADR-approved.
5. **Blockchain**: off-chain hash anchoring ONLY (certificates, donation
   receipts). No personal data on-chain — immutable ledgers violate
   GDPR Art. 17.
6. **Live streaming**: YouTube embeds ONLY, behind a two-click / consent-mode
   gate (no cookies before consent). Native streaming infrastructure is
   out of pilot scope.
7. **Change control (SOC 2 CC8.1)**: every production change gets an issue,
   snapshot, `.bak.<timestamp>`, CHANGELOG entry, and rollback plan.

## 3. VERIFIED ESTATE (GROUND TRUTH — DO NOT CONTRADICT)

| Repository | Role | Verified facts |
|---|---|---|
| `jol-hub` | Tier 0 integration monorepo | Django backend (apps: core, content, crm, organizations, users, donations, financial, integrations, countries, ai, analytics). Turborepo/pnpm/Next.js/Tailwind frontend: apps `admin-dashboard`, `master-site`, `parish-template` + 12 `lt-*` entity demos; shared packages `ui`, `i18n`, `auth`, `bitrix-sdk`. `countries/lt/` exists. |
| `jol-auth` | OAuth 2.1 / OIDC | Identity for all tenants; HS256→RS256 trajectory. |
| `jol-ecommerce-engine` | Commerce | Stripe ~99% integrated, PCI-DSS SAQ A scope. |
| `jol-bitrix24-integration` | CRM layer | Bitrix24 = CRM/sales-pipeline/tasks/telephony. NOT a CMS. 90-day token rotation. |
| `jol-frontend-platform` | STUB | Reserved for frontend extraction from jol-hub (see ADR-002). |
| `jol-rag-server`, `jol-llm`, `jol-mcp-servers` | AI estate | On-prem inference; EU-only external fallback. |
| `jol-infrastructure` | Tier 4 SECONDARY | NEVER contains application logic; hosts pilot VM specs (`docs/servers/jol-*-pilot-lt01.md`) and acceptance gate (`docs/runbooks/jol-pilot-acceptance-gate.md`). |

Known conflicts you MUST resolve, not inherit:
- The 12 hard-coded `lt-*` demo apps violate the tenant model → become seed
  fixtures; runtime must be ONE template renderer + tenant resolution.
- `jol-hub` README drift (claims SQLAlchemy/Alembic/npm; reality Django/pnpm)
  → reconciliation task.
- Pre-work hygiene gate: execute `docs/compliance/pre-work-hygiene-gate.md`
  BEFORE any feature work.
- DB model ratified as **schema-per-tenant (PostgreSQL) + RLS defense-in-depth**,
  NOT database-per-site → ADR-001; delta against
  `jol-infrastructure/docs/servers/jol-db-pilot-lt01.md` required.

## 4. PILOT SCOPE — LITHUANIA ([DIOCESE-CLUSTER])

Waves (ratified): Wave 0: 3–5 reference sites for presentation to Bishops,
priests, funeral homes, cleaning companies → Wave 1 ≈ 25 sites (below) →
Wave 2 ≈ 100 → industrialization to ~1,300 LT addresses (700 churches,
400 funeral homes, 200 cleaning services), then 27 EU countries.

Wave 1 structure (23 sites):
- 1 Diocese site: [diocese-cluster]
- 5 Deanery clusters, each = Deanery site + 1 church + 1 funeral home +
  1 cleaning service:
  - [diocese-city] Deanery | [diocese-city] church | [diocese-city] funeral | [diocese-city] cleaning
  - [deanery-b] Deanery | [parish-b] | [tenant-b] funeral | [tenant-b] cleaning
  - [deanery-c] Deanery | [parish-c] | [tenant-c] funeral | [tenant-c] cleaning
  - [deanery-d] Deanery | [parish-d] | [tenant-d] funeral | [tenant-d] cleaning
  - [deanery-e] Deanery | [parish-e] | [tenant-e] funeral | [tenant-e] cleaning

All pilot sites: `https://{tenant-slug}.[pilot-domain]` subdomains.
Legacy `*.bitrix24site.ru` sites are retired when JOL hosting starts on the
Dell R640 — design a 301-migration + SEO equity transfer plan, do not copy
their design; produce a keep/remove/redesign/componentize audit of:
[pilot-tenant-url], /[parish-a]/, /[funeral-tenant-a]/,
/[cemetery-tenant-a]/.

## 5. VERTICAL TAXONOMY (10 VERTICALS)

1. Roman Catholic Basilicas · 2. Cathedrals · 3. Dioceses (+commercial) ·
2. Diaconate (+commercial) · 5. Roman Catholic Churches · 6. Protestant
Churches · 7. Russian Orthodox Churches (+commercial) · 8. Other Churches
(denomination-agnostic catch-all placeholder template) · 9. Funeral Homes
(+commercial) · 10. Cemetery Graves & Monument Cleaning (+commercial).

One JOL Design System + reusable component library + vertical-specific
templates and content models. Commercial entitlements are vertical-specific
subsets, e.g. cemetery cleaning = booking + payments + subscriptions +
e-commerce (flowers, bouquets); funeral homes = enquiry + booking + obituary
notices + live-stream embeds + e-commerce (coffins, urns, vestments).
Produce the full vertical × module entitlement matrix.

## 6. PACKAGE MODEL — CAPABILITY TIERS (PAGE COUNTS ARE BANNED)

The unit of composition is the **module** (static page OR dynamic collection
such as `/news/{slug}`, `/events/{slug}`, `/services/{slug}`). Packages are
modules × automation × commercial entitlements:

| Package | Setup fee | Recurring | Commercial |
|---|---|---|---|
| CHEAP | €1,000 one-off | €20/mo Bitrix24 CRM Phone app | No/limited |
| NORMAL | €2,000 one-off | €20/mo Bitrix24 CRM Phone app | Optional |
| VIP | €3,000 one-off | €20/mo Bitrix24 CRM Phone app | Full |

Prices are RATIFIED — do not change them; you may (must) justify tier
boundaries by module entitlements, automation level, and support SLA.

## 7. COMMERCIAL MODEL

JOL commission = **10%** of commercial transactions processed through the
tenant's site, uniform across all commercial verticals (Diaconate, Orthodox,
Funeral Homes, Cemetery services). Record in Decision Log that this
supersedes earlier 20% statements. Commercial capabilities already built in
`jol-ecommerce-engine` / backend: catalogue, lead-gen, quotations, booking,
payments (Stripe SAQ A), subscriptions, customer accounts, invoices/orders,
CRM sales pipeline, multilingual (27), social integration, AI assistant,
AI sales scripts/speech analytics, boards (chats/tasks/feed/video), secure
work board, iOS/Android. Map each capability to its owning repository and
pilot phase; flag anything not yet built as `⚠ UNVERIFIED`.

## 8. MULTI-TENANCY CONTRACT (THE SPINE)

Design the full chain: **tenant → organization → website → domain/subdomain
→ locale → template → content**. Requirements:
- Content ownership: JOL controls 90% (design system + structured content
  models); tenant edits 10% via a constrained block editor with moderation.
- Tenant uploads (images included) pass: malware scan (ClamAV-class),
  on-prem AI moderation (via the Ollama/RAG stack), Art. 9 review queue.
- CMS model = option F (combination): JOL admins + tenant admins + Bitrix24
  as CRM/task layer (never CMS). Validate against `jol-hub` backend apps.
- Data isolation: PostgreSQL **schema-per-tenant** + row-level security as
  defense-in-depth; per-site logical separation; Art. 17 erasure via
  `deleted_at` logical deletion + legal sign-off + erasure log per
  `jol-infrastructure/docs/compliance/erasure-log.md`.
- Tenant resolution at the edge/ingress: subdomain → tenant → template
  variant → locale → content, cacheable per (tenant, locale, template).

## 9. FRONTEND ARCHITECTURE MANDATE

Produce a **scored decision matrix** (weighted criteria: on-prem hosting fit,
SSR/SEO, i18n at 27 locales, Core Web Vitals on modest hardware, team
existing investment, migration cost, security surface) comparing Next.js vs
Astro vs Nuxt vs Django-templated — with the verified jol-hub Next.js
investment as the incumbent column. Then decide, per page class, the
rendering strategy (SSG/ISR/SSR) and the app topology: ONE template-renderer
application + tenant resolution vs per-entity apps (the latter is the
anti-pattern to migrate away from). Mandatory: WCAG 2.2 AA, structured
data, responsive design, tenant-aware routing, image optimization pipeline.

## 10. SEO ARCHITECTURE (MANDATORY — THIS IS AN SEO PLATFORM)

Full architecture: technical SEO; on-page; local SEO; Google Business
Profile integration; schema.org (ReligiousOrganization, Church,
Organization, LocalBusiness, FuneralHome, Service, Product, Event,
BreadcrumbList, FAQPage); sitemap architecture per tenant; hreflang
(LT/EN/RU); canonicals; robots.txt per tenant; Open Graph + X cards; image
SEO; Core Web Vitals budgets; programmatic SEO; and the 400,000-site
indexing strategy (crawl-budget management, sitemap sharding, IndexNow,
per-tenant search-console onboarding).

## 11. LANGUAGES

Pilot: Lithuanian (primary), English, Russian — all three at pilot launch.
Polish is an OPEN QUESTION (record in Decision Log). Architecture must
support 27 EU locales without redesign (`packages/i18n` extension path).

## 12. JOL DESIGN SYSTEM

Define formally: typography, color, spacing, grid, buttons, navigation,
cards, forms, calendars/events, news, galleries, maps, donations, services,
testimonials, contact, footer, accessibility primitives, mobile components.
CHEAP/NORMAL/VIP are **compositions of the same components** — no VIP-only
parallel codebase. Audit the four legacy bitrix24site.ru examples:
retain / remove / redesign / componentize / template-specific / violates-
modern-practice buckets.

## 13. COMPLIANCE AS ARCHITECTURE (NOT DOCUMENTATION)

Every control must be enforceable in code/config: GDPR privacy-by-design,
data minimization, consent management (cookie/YouTube gates), retention
schedules, Art. 15/17/20 export/erasure, audit logs, encryption in transit
+ at rest (ZFS aes-256-gcm verified on pilot VMs), RBAC, tenant isolation
(schema + RLS), secrets management (Vaultwarden), MFA, OWASP ASVS L2,
logging/monitoring, incident response (<72 h breach notification),
backup/DR (PBS, RPO 24 h/RTO 4 h pilot baseline), supply-chain security,
vulnerability management. Map each to named controls: SOC 2 CC/A-series,
ISO 27001:2022 Annex A, GDPR Articles. DPIA triggers: church+marketplace
data sharing, AI content generation, tenant upload moderation.

## 14. MANDATORY DELIVERABLES (ALL 31)

1. Executive architectural assessment · 2. Business/product taxonomy ·
2. Website vertical matrix · 4. CHEAP/NORMAL/VIP capability-tier definition ·
3. Module matrix (static vs dynamic collections) · 6. UI/UX architecture ·
4. JOL Design System · 8. Frontend architecture (incl. scored framework
matrix + rendering strategy) · 9. Django/jol-hub API architecture ·
5. Bitrix24 integration architecture · 11. Database/content model
(schema-per-tenant ADR) · 12. Multi-tenant architecture · 13. Domain/
subdomain architecture · 14. SEO architecture · 15. Accessibility (WCAG 2.2
AA) architecture · 16. Security architecture · 17. GDPR/SOC 2/ISO 27001
control map · 18. GitHub repository architecture (incl. jol-frontend-platform
extraction decision per ADR-002) · 19. CI/CD architecture · 20. Infrastructure/
deployment (on-prem Proxmox; Dell R640 capacity) · 21. Testing strategy ·
6. Observability · 23. Backup/DR · 24. Cost model · 25. Scalability path to
~400,000 sites · 26. Migration strategy (legacy bitrix24site.ru → JOL, 301s,
SEO equity) · 27. Implementation phases (Wave 0 → 1 → 2 → industrialization) ·
7. Prioritized backlog · 29. P0/P1/P2 risk register · 30. Per-recommendation
Security/Compliance/Cost/Rollback annex · 31. Decision Log & Open Questions
(must record: 10% commission unification; schema-per-tenant ratification;
PL language open; framework matrix outcome).

Every deliverable maps to owning repository(ies) from §3 and states what is
already built vs new work.

## 15. ACCEPTANCE GATE

Bind all deliverables to
`jol-infrastructure/docs/runbooks/jol-pilot-acceptance-gate.md` plus
business KPIs: Lighthouse ≥ 90 (mobile) per pilot site; Core Web Vitals
green; tenant onboarding ≤ 1 working day/site; zero Art. 9 data leakage in
cross-tenant tests; all 23 Wave-1 sites live on `[pilot-domain]`
before legacy retirement. Timeline: as soon as possible — therefore phase
the backlog so Wave 0 reference sites ship first.

## 16. OUTPUT RULES

- Structured Markdown; tables for matrices; no hand-waving.
- Every major recommendation: security impact / compliance impact /
  cross-repo impact / rollback strategy.
- Unverified claims marked `⚠ UNVERIFIED — manual check required`.
- No brand-name AI providers as requirements; no AWS assumptions;
  no page-count package definitions; no on-chain personal data.
- Respect repo sovereignty: application logic lives in PRIMARY repos
  (jol-hub and satellites); jol-infrastructure gets infra only; secrets
  never in inventory or code.

===
2. <font color="#f79646">ADR-001 — schema-per-tenant (records the ratified change and the required spec delta):</font>
<host>/repos/jol-hub/docs/decisions/ADR-001-schema-per-tenant-isolation.md
# ADR-001: Tenant Data Isolation — PostgreSQL Schema-per-Tenant + RLS

## Status
Accepted — ratified by platform owner 2026-08-24. Supersedes earlier
"separate database per website" formulation.

## Context
The Lithuania pilot (23 Wave-1 tenants, path to ~1,300 LT and ~400,000 EU
tenants) requires tenant data isolation that satisfies GDPR Art. 9
special-category data, Art. 17 erasure, and cross-tenant confidentiality,
while remaining operable on the on-prem pilot DB (`jol-db-pilot-lt01`)
without one database instance per site.

## Decision
- Isolation unit = **PostgreSQL schema per tenant** on a shared cluster.
- **Row-Level Security** policies as defense-in-depth on all
  tenant-scoped tables.
- Connection role strategy: least-privilege application role; tenant schema
  resolved per request from the tenant-resolution chain
  (subdomain → tenant → schema); `search_path` pinning; no cross-schema
  grants.
- Erasure: `deleted_at` logical deletion + legal sign-off + erasure log
  (per `jol-infrastructure/docs/compliance/erasure-log.md`), followed by
  scheduled physical purge per retention policy.
- Backups: PBS at VM level retains whole-cluster recoverability; per-tenant
  export/erasure tooling required for Art. 15/17/20.

## Consequences
- **Positive**: Operable at 400k-tenant scale; single cluster to harden,
  patch, and back up; clean per-tenant erasure boundary (DROP/EXPORT schema).
- **Positive**: RLS provides a second independent isolation layer.
- **Negative**: Schema-count growth requires migration tooling discipline
  (migrations must fan out across tenant schemas).
- **Negative**: Requires spec delta against
  `jol-infrastructure/docs/servers/jol-db-pilot-lt01.md` (change-controlled
  issue + doc update) — OPEN ACTION.

## Alternatives Considered
1. Database-per-tenant — rejected (operational collapse at scale; 400k DBs).
2. Single shared schema + tenant_id column only — rejected (single-layer
   isolation insufficient for Art. 9 data without RLS; weaker erasure story).
3. Citus/distributed Postgres — deferred (re-evaluate at industrialization;
   adds an estate dependency before the pilot proves the model).

## Compliance
- GDPR Art. 9 / Art. 25: special-category data isolation by design
- GDPR Art. 17: schema-scoped erasure boundary
- SOC 2 CC6.1 / CC6.3: logical access controls, least privilege
- ISO 27001:2022 A.8.13 (segregation of environments), A.5.34 (privacy)

---
<font color="#f79646">3. ADR-002 — frontend extraction (deliberately deferred — extract after Wave 0 proves the renderer, not before):</font>

<host>/repos/jol-hub/docs/decisions/ADR-002-frontend-extraction-jol-frontend-platform.md
# ADR-002: Frontend Extraction into jol-frontend-platform

## Status
Proposed — decision deferred until Wave 0 exit criteria are met.

## Context
`jol-hub/frontend` (Turborepo/pnpm/Next.js) currently holds 15 apps,
including 12 hard-coded `lt-*` entity demos that contradict the tenant
model. `jol-frontend-platform` exists as a stub reserved for extraction.
Extracting too early freezes an unproven architecture; extracting too late
entrenches the per-entity anti-pattern.

## Decision
1. Wave 0: build the **single template-renderer app + tenant resolution +
   structured content models** inside `jol-hub/frontend`; convert the 12
   `lt-*` demos into seed fixtures. Framework choice confirmed via the
   scored matrix required by MASTER-PROMPT §9 (Next.js incumbent).
2. Extraction into `jol-frontend-platform` is triggered ONLY after:
   Wave 0 reference sites pass the acceptance gate, the tenant-resolution
   chain is proven, and packages (`ui`, `i18n`, `auth`, `bitrix-sdk`) have
   stable public APIs.
3. Extraction moves code, not contracts: `jol-hub` retains the integration
   contracts (entities, API schemas); Tier-0 sovereignty rules unchanged.

## Consequences
- **Positive**: No premature repo split; pilot velocity preserved.
- **Positive**: Extraction decision becomes evidence-based (SOC 2 CC3.1).
- **Negative**: Monorepo grows during pilot; extraction cost rises with
  delay — trigger conditions are binding, not aspirational.

## Alternatives Considered
1. Extract immediately — rejected (freezes unproven tenant architecture).
2. Never extract — rejected (violates ratified ecosystem map; monorepo CI
   blast radius at 400k-site scale).

## Compliance
- SOC 2 CC8.1: extraction executed as change-controlled migration
- ISO 27001:2022 A.8.32: change management for repo reorganization

---

<font color="#f79646">4. Pre-work hygiene gate (must pass before any frontend feature work):</font>

<host>/repos/jol-hub/docs/compliance/pre-work-hygiene-gate.md
# Pre-Work Hygiene Gate — jol-hub Repository Baseline

> Compliance: SOC 2 CC6.1/CC7.2 · ISO 27001:2022 A.8.24/A.8.32 · GDPR Art. 32
> Blocking gate for MASTER-PROMPT-LT-PILOT-FRONTEND. No feature work before PASS.
> Rule: never pass credentials as CLI arguments; scanners read config/env only.

## Step 0 — Read-only baseline
- [ ] `cd <host>/repos/jol-hub && git status --porcelain > /tmp/jolhub-baseline.txt`
- [ ] Record branch, HEAD SHA, and working-tree state in the change-control issue.

## Step 1 — Tracked-file secret inventory
- [ ] `git ls-files | grep -E '\.env(\.|$)|\.pem$|\.key$'` → list every match.
- [ ] Known candidates (verified present in working tree, tracking status ⚠ UNVERIFIED):
  `backend/.env`, `backend/django/.env`, `frontend/.env.example`,
  `frontend/apps/parish-template/.env.local`, `frontend/apps/parish-template/.env.example`.
- [ ] For each: confirm git-tracked or not (`git ls-files --error-unmatch <file>`);
      if tracked AND contains real values → P0: rotate the credential in
      Vaultwarden FIRST, then purge from history (git-filter-repo/BFG),
      then invalidate GitHub caches. Never edit before rotating.

## Step 2 — History scan
- [ ] `trufflehog git [repo]/jol-hub --only-verified` → 0 verified findings.
- [ ] `git secrets --scan` at HEAD (register patterns first: AWS, Stripe `sk_live_`,
      Bitrix24 tokens, JWT secrets).
- [ ] Any verified finding → incident playbook (`jol-security`), not a silent fix.

## Step 3 — Stray-artifact cleanup (back up, then remove)
- [ ] Move to evidence dir or delete with change record:
  `backend/django/counter_*.db`, `backend/django/histogram_*.db` (8 files).
- [ ] Remove zero-byte junk: `frontend/jol-hub@1.0.0`, `frontend/tsc`, `frontend/turbo`.
- [ ] Relocate `STEP18_EXECUTED.md`, `STEP19_EXECUTED.md` →
      `docs/compliance/evidence/` or fold into CHANGELOG.md.

## Step 4 — Hygiene hardening
- [ ] `.gitignore` covers: `.env`, `.env.local`, `*.db` artifacts, `node_modules/`,
      `tsconfig.tsbuildinfo`, `venv/`, `__pycache__/`.
- [ ] Add `.env.example` templates only (no values).
- [ ] README reconciliation: SQLAlchemy/Alembic→Django, npm→pnpm
      (drift vs actual `backend/django/manage.py` + `pnpm-workspace.yaml`).

## Step 5 — Exit criteria (evidence required in the change-control issue)
| # | Gate | Expected |
|---|---|---|
| 1 | trufflehog verified findings | 0 |
| 2 | git-secrets scan at HEAD | 0 |
| 3 | Tracked `.env`/key files with real values | 0 |
| 4 | Stray SQLite/junk files | removed + recorded |
| 5 | `.gitignore` hardened | committed |
| 6 | AIDE-equivalent repo seal | HEAD SHA recorded, CHANGELOG entry |

Rollback: all changes are doc/metadata-only; revert commit restores prior state.
If Step 1/2 finds live secrets: STOP — credential rotation precedes all other steps.