]\



## "Where to park" is the wrong decision — "where to consolidate" is the right one

**Parking ≠ registrar ≠ hosting.** Parking is just the placeholder page a registrar shows when DNS points nowhere. The strategic decision is **which registrar holds the whole portfolio** — because at 27 countries you will manage 28+ domains, and each registrar account is a security perimeter, an MFA policy, a billing/audit trail (SOC 2), and an API integration for your Terraform/Ansible DNS automation.

## Hostinger vs GoDaddy — against your actual criteria

| Criterion | Hostinger | GoDaddy |
|---|---|---|
| Jurisdiction | **EU (HQ Vilnius)** — GDPR-native, your pilot country | US — CLOUD Act exposure, EU entity secondary |
| Current JOL state | jol-hub.com, gyvenimo-kelias.lt, dzives-cels.lv already there | none |
| ccTLD coverage for 27 markets | **Limited** — the real weakness at scale | Broad catalog |
| API/automation for infra | Limited | Mature |
| Renewal pricing | Stable | notorious hikes |
| Splitting cost | — | doubles audit + attack surface for zero pilot benefit |

**Recommendation: consolidate everything at Hostinger for the pilot, do not open a GoDaddy account.** Reasons, all cited: (1) EU jurisdiction matches your compliance posture (GDPR is a hard constraint); (2) three of four domains are already there — migrating working domains is change-control risk for no gain; (3) one perimeter beats two. **Flag for industrialization**: Hostinger's ccTLD catalog may not cover all 27 EU ccTLDs (several have local-presence rules) — when you reach that scale, the defensible move is an EU specialist registrar (EuroDNS/INWX class), not GoDaddy. Defer that decision; record it so it isn't rediscovered mid-rollout.

## Parking-specific advice (the thing actually visible to the world)

1. **Kill the Hostinger ad page on `dzives-cels.lv`** — a parked page advertising third-party hosting on a faith-and-memorial platform's domain is a brand and trust leak; crawlers index it too. Best-to-worst: undelegated DNS (exactly what `elu-tee.ee` does today — cleanest), or a branded HTTPS "coming soon" placeholder on your own infra.
2. Parking is transitional. The end state per the ratified SEO strategy §2.2 is **wildcard DNS per country domain → Proxmox ingress + wildcard TLS** — plan parking to end there, not to live anywhere.

## Compliance note (registrar account = critical credential tier)

MFA enforced, org-owned email (not personal), registry/registrar lock on production domains once live, WHOIS privacy (automatic for EU natural persons), and one consolidated billing account as the audit trail. Treat the registrar account with the same discipline as Vaultwarden.

**Status note:** this remains advisory — gate state is still PENDING. This finding partially resolves **ASSUME-SEO-006** (LV confirmed Hostinger; EE owner-declared, independently unverifiable). Say the word and I'll log the domain-status verification + consolidation recommendation as one DECISION-LOG entry (candidate D-011), pending your sign-off.