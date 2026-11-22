

Executing the prompt as written — starting with §5 Verification Contract against the live trees. Writing the probe script, then running it:

Can't write outside the workspace — placing the probe script in the repo root as a dot-file, running it, then removing it:

Verification round 2 — closing the gaps the first probe opened (jol CI actually uses `secrets.GITHUB_TOKEN` default, not a stored classic PAT; AWS CLI is present; dm-crypt probe needs root):

Two more read-only probes settle the open questions (dm-crypt absence without root, S3 deny in practice):

S3 SCP deny confirmed live from AWS's own error (SCP `p-r9i2lq77` named in the response). But `lsblk` shows 2 crypto entries — contradicting the "no dm-crypt" baseline. Pinning exact devices before I report either way:

SCP deny, FDE status, TF version — all settled. Cleaning up probe files, then delivering the answer:

# §0 Verification results

| Fact group | Status | Evidence / correction |
|---|---|---|
| §4.1 three planes, paths, entity mapping | CONFIRMED | all three dirs exist; `cloud` block org `jolarca-dev` ws `jolarca-control` |
| §4.2 state sizes/modes (179004/600, 195121/600, 0) | CONFIRMED | `stat` this session |
| §4.2 viavitae serial 13 / ~11 resources (older register) | STALE | actual: serial **143**, **82** resources, lineage `eff08856…`; jol serial 306, lineage `25425c0f…` |
| §4.3 age key 189 B/0600; archives viavitae 10‑17, jol 10‑13; no cadence control | CONFIRMED | `ls -t /opt/jol/backup/tfstate/` |
| §4.3 "dm-crypt returns 0 devices" | STALE — wrong method, right conclusion, different facts | `dmsetup` fails unprivileged (Permission denied). Correct evidence: `/dev/mapper` holds only LVM (`vgopt-*`); root boots by plain UUID (no `cryptdevice=` in `/proc/cmdline`); **two `crypto_LUKS` 14.6 G containers exist on sda/sdb, unmounted — swap/hibernate, not data**. System/data disk: not encrypted. CONFIRMED |
| §4.4 tooling | STALE | `aws` CLI **is present and configured** (`/usr/local/bin/aws`); `pass`/`bw`/`ykman`/`keepassxc-cli` absent CONFIRMED |
| §4.5 SCP `p-r9i2lq77` explicit S3 deny | CONFIRMED live | `aws s3api list-buckets` → `AccessDenied … explicit deny in a service control policy: arn:aws:organizations::376896792604:policy/o-omcubugbug/…/p-r9i2lq77`; caller = `viavitae-terraform@775093708742` |
| §4.6 Surface A (5 FG PATs) | SCREENSHOT-ONLY | org inventory endpoint 404s on all three orgs (re-probed) — not shell-enumerable |
| §4.6 B1 scopes | STALE | actual keyring scopes: `admin:org, gist, read:user, repo, user:email, workflow` (baseline omitted `gist`); active account `JourneyOfLife` CONFIRMED |
| §4.6 C3 "jol GITHUB_TOKEN = classic PAT admin:org…" | STALE — contradicted | jol workflows use the **Actions-injected default** `secrets.GITHUB_TOKEN` plus stored `ORG_READ_TOKEN` (runbook RB‑07). No classic PAT is stored in jol CI |
| §4.6 C4 jolarca | CONFIRMED, expanded | three secrets: `TFC_TOKEN`, `TF_GITHUB_TOKEN_READONLY` (repo), `TF_GITHUB_TOKEN` (production environment) — `plan.yml`/`apply.yml`; `key-custody.md` register exists (90 d rotation) |
| §4.7 org 2FA | CONFIRMED via API | `two_factor_requirement_enabled: true` for all three orgs |
| §4.8 boundary 1 | CONFIRMED + new finding | covers `jolarca-dev`↔`journeyoflife-org` only; **isolation-model.md still says jolarca state is "local-only single copy (D‑02)" — contradicted by the live `cloud` block** (doc drift) |
| Terraform version | CONFIRMED | v1.16.1 → S3 native locking (`use_lockfile`) available; no DynamoDB needed |

**Surface A↔C reconciliation (the §5.5 demand):**

| PAT (A) | Inferred consumer | Confidence |
|---|---|---|
| A4 `viavitae-control-ci-plan` | viavitae `TF_GITHUB_TOKEN` (plan + credential check) | CONFIRMED (token id in CI logs) |
| A3 `viavitae-control-drift` | viavitae `ORG_READ_TOKEN` (drift, assert-live) | strong inference |
| A1 `jol-control-drift-read` | jol `ORG_READ_TOKEN` (RB‑07) | strong inference |
| A2 `jolarca-control-tf-write-2026-10-17` | jolarca `TF_GITHUB_TOKEN` (production env, apply.yml) | inference — "Never used" = no apply since mint, **not** proof of orphan |
| A5 `jolarca-dev` | jolarca `TF_GITHUB_TOKEN_READONLY`? or interactive | **UNMAPPED — finding F‑04** |

Reconciled stored credential set = **8** (A1–A5, B1, B2, `TFC_TOKEN`) plus ephemeral injected `GITHUB_TOKEN`s.

# §1 Executive verdict

1. Q1: Three planes is correct; machine-credential separation is real, human-identity separation is cosmetic (one keyring, one 2FA, one password owns all three controllers).
2. Q2: The set is 8, not 5. Retire B2 (`IterVitae` orphan), map-or-revoke A2, finish the A4 org de-grant, stagger the Jan‑2027 ten-day expiry cluster.
3. Q3: S3 is dead on arrival (SCP deny proven live, again). jolarca stays HCP; viavitae moves local→HCP; jol must **not** follow its `architecture.md` HCP plan while the charity's in-house principle stands — interim local+off-host, target self-hosted HTTP backend.
4. Q4: Excel+Dropbox+TrueCrypt is rejected outright. Keep the `age` pattern you already run, per-entity vaults, metadata-only inventory in each repo (jolarca's `key-custody.md` is the template), XOR-sealed break-glass for Tier 0.
5. Top risk: state + its only backup + the age key + the vault all live on **one unencrypted disk** — one failure ends all three entities' recovery.
6. Top risk: two classic `admin:org`-grade human tokens with unknown expiry (B1) and an orphan second account (B2).
7. Top risk: compliance registers contradict the tree in both directions (dm-crypt claim, C3 claim, D‑02 text) — an auditor finding these treats every other claim as false.

# §2 Question 1 — Three planes

**1.1 Legal justification.** a. Each plane is the operational arm of a distinct controller: Via‑Vitae (commercial), journeyoflife-org (charity — GDPR Art. 5(1)(f) custodian of parish/diocese data), jolarca-dev (PCI-DSS 4.0 Req. 7). b. A merged plane puts all three controllers' resource inventories (repo names, node IDs, protection lists — all present in `terraform.tfstate`) in one file readable by any one of the three credentials → violates boundary 1, Art. 5(1)(f), ISO A.5.34, PCI Req. 7.2. c. Verdict basis: state content is the data; separation of planes is separation of controllers.

**1.2 What separation requires — achieved today?**

| Dimension | viavitae | jol | jolarca |
|---|---|---|---|
| Separate state | ✅ local file | ✅ local file | ✅ HCP ws |
| Separate backend/encryption | ✅ (disk) | ✅ (disk) | ✅ HCP-managed |
| Separate machine credentials | ✅ A3/A4 | ✅ A1 + injected | ✅ A5?/A2/TFC |
| Separate CI identity | ✅ | ✅ | ✅ read/write split (boundary 3) |
| Separate drift detection | ✅ own scripts | ✅ own workflow | ✅ own scripts |
| Separate **human** identity | ❌ one gh keyring, one GitHub account, one 2FA across all three | ❌ |  |

d. The  row is structural for a solo operator; compensating control = per-entity machine credentials (done) + org 2FA required (CONFIRMED all three) + blast-radius separation so one leaked CI token never yields org admin.

**1.3 Shared code.** a. May copy freely: workflow skeletons, guard-script logic, README patterns (jol/viavitae already diverged copies — that is the sanctioned model). b. May become a shared module: nothing today — only viavitae has `modules/`; jol and jolarca have none. c. Must never share: provider configs, allow-lists, state, tokens, org webhooks config. d. Supply-chain rule if a shared registry is ever created: pin by `?ref=<tag>` **and** `.terraform.lock.hcl` hash, and treat a registry write as a change to all three entities simultaneously — with one reviewer, that means a plan-diff gate per consumer plane, not trust.

**1.4 Via-Vitae gap remediation.** 1. Edit `jolarca-control/docs/security/isolation-model.md` boundary 1 → name all three pairings, or declare viavitae's own boundary doc in `viavitae-control/docs/` and link from the other two (single source, no duplicate doctrine — their own A.5.2 argument). 2. Fix the stale D‑02 "local-only" paragraph in the same edit. 3. Add a `fleet-separation`-style guard (jolarca has `fleet-separation-guard.yml`) to viavitae CI asserting no jolarca/jol resource or repo appears in its tree/state. 4. Register the change in `docs/COMPLIANCE.md`.

**1.5 Verdict.** Three planes: **correct**. Separation stops being cosmetic when: boundary 1 covers all pairings (1.4), the human-identity SPOF is registered with compensating controls (1.2d), and the registers stop contradicting the tree (§0).

# §3 Question 2 — The credential set

**7.1 Full credential matrix** (cells compressed; "used" = evidenced in tree/CI, not granted):

| ID | surf | name | entity | type | granted | used-evidence | storage | consumer | expiry | rot | blast radius | revocation | compensating today |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| K1 | A | jol-control-drift-read | jol | FG PAT | read (inferred) | drift/compliance steps | GitHub settings + jol `ORG_READ_TOKEN` secret | jol CI reads | 2027‑01‑17 | 90d | org-wide private-repo read | UI Delete | fail-loudly step if absent |
| K2 | A | jolarca-control-tf-write-2026-10-17 | jolarca | FG PAT | write | **none** ("Never used") | GitHub + jolarca `TF_GITHUB_TOKEN` (prod env) | apply.yml (inferred) | 2027‑01‑15 | 90d | repo-admin on PCI fleet | UI Delete | prod-environment gate, `prevent_destroy`, plan-refusal gate |
| K3 | A | viavitae-control-drift | Via‑Vitae | FG PAT | read | drift + assert-live | GitHub + viavitae `ORG_READ_TOKEN` | detect_drift.sh, assert_live_controls.sh | 2027‑01‑15 | 90d | org-wide private-repo read | UI Delete | CI step skipped-if-missing |
| K4 | A | viavitae-control-ci-plan (id 21301575) | Via‑Vitae | FG PAT | Repos 2 (Adm+Meta RO) **+ 2 org grants still present** | plan green run 79 | GitHub + viavitae `TF_GITHUB_TOKEN` | audit.yml plan + check_ci_credential | 2027‑01‑15 | 90d | read admin metadata of 2 repos + org member lists (until de-grant) | UI Delete | credential-contract gate prints unused-power notice |
| K5 | A | jolarca-dev | jolarca | FG PAT | unknown | weekly use | GitHub + (likely) `TF_GITHUB_TOKEN_READONLY` | plan.yml read step (inferred) | 2027‑01‑08 | 90d | unknown — **unmapped** | UI Delete | key-custody register row exists for READONLY |
| K6 | B | terraform-control-plane | all three (human) | classic PAT | admin:org, gist, read:user, repo, user:email, workflow | interactive plan/apply, `gh` | `gh` keyring (JourneyOfLife active) | laptop: `export GITHUB_TOKEN="$(gh auth token)"` | unknown | manual | **org admin on all three orgs** | UI revoke + invalidate sessions | org 2FA required; plan-only habit |
| K7 | B | IterVitae token | — | classic PAT | gist, read:org, repo, workflow | none needed (inactive) | `gh` keyring | none | unknown | — | outside-collab repo access | `gh auth logout` + UI revoke | none |
| K8 | C | TFC_TOKEN | jolarca | HCP token | workspace access | apply.yml/plan.yml CONFIRMED | GitHub env secret + HCP | terraform init/apply CI | unknown | manual | PCI state read/write | HCP UI rotate | HCP org isolation |
| K9 | C | injected GITHUB_TOKEN | jol (+others) | Actions default | per `permissions:` block | CONFIRMED in ymls | ephemeral per-run | jol apply steps | hours | auto | scoped, short-lived | n/a | least-priv `permissions:` blocks |

**7.2 K2 dormant write token.** a. It is *wired* (apply.yml reads `TF_GITHUB_TOKEN` in the `production` environment) but *unexercised* since 2026‑10‑17 — "never used" means no apply ran, not that the token is orphaned. b. Verdict: **keep, with a dated trigger** — if no apply has used it by 2026‑11‑15, revoke; a write credential held against a PCI fleet with no observed use offends ISO A.8.2 (privileged access management) and CC6.3. c. Add the mapping row to `key-custody.md` — an unmapped credential is the actual finding (F‑04), not the dormancy.

**7.3 Unused power.** K4 still carries Organization Administration+Members read (register memory + CI notice confirm the de-grant was never applied). Steps: UI `github.com/settings/personal-access-tokens/21301575` → Permissions → remove both org grants → keep `Repositories 2` → press the **bottom** Update. Token string unchanged → no secret re-paste. `STOP — human approval required before this step` (scope edit = mutation). Verify: rerun main audit, require the "unused power" notice ABSENT in a new attempt.

**7.4 Classic vs fine-grained.** a. K6: retain classic for now — org-level resources (`github_organization_webhook` in jol's `org-settings.tf`, org members/settings in viavitae) need `admin:org`-class rights; FG "Organization admin read-write" coverage of every provider resource is not proven → migration verdict: **migrate after a per-resource FG probe in a scratch run**, order: mint FG → plan with FG → apply-equivalence diff → revoke classic; rollback = re-paste classic env var (token string preserved in keyring until you delete it). b. K7: no migration, retirement.

**7.5 Charity scope minimality.** `admin:repo_hook` is unused (tree has zero `github_repository_webhook`); `admin:org` **is** used (org audit-stream webhook, org-level repo creation per apply.yml header comment). Narrow any human token to drop `admin:repo_hook`, `write:org` if unproven, and `gist` (present on K6, used by nothing).

**7.6 IterVitae (K7).** **Retire.** Outside-collaborator with `repo` scope, inactive, no named consumer; pending-invite history makes it latent, not load-bearing. `gh auth logout --hostname github.com --username IterVitae` (removes keyring entry only; also revoke the token server-side — UI path `github.com/settings/tokens`). `STOP — human approval` before server-side revoke.

**7.7 Expiry calendar.** Current cluster: 2027‑01‑08 (K5), 01‑15 (K2,K3,K4), 01‑17 (K1) — five credentials, three entities, ten days. One distracted week = simultaneous loss of plan+drift across all controllers. Staggered target (rotate early in December, set 180‑d expiries): K5→2027‑04‑08, K1→2027‑05‑06, K3→2027‑06‑03, K4→2027‑07‑01, K2→2027‑08‑05 — ≤1 per entity per 7‑day window. Reminder mechanism one person keeps: expiry dates live as metadata rows in each repo's inventory file (§9.6); a CI step fails loudly when `today > expiry − 14d` for any row — the machine remembers, not you.

**7.8 Rotation & revocation.** Per credential, <30 min: 1. mint replacement in UI (same scope, minus §7.3/§7.5 excesses); 2. update the Actions secret (`gh secret set NAME -R org/repo`) — `STOP — human approval`; 3. run audit CI green on new secret; 4. revoke old token in UI — `STOP`; 5. update inventory row. Emergency (token in public commit): GitHub push protection blocks the push; secret scanning auto-revokes *GitHub-issued* tokens only — PATs are flagged, not revoked, so: revoke immediately (step 4), then rotate anything that token could read; note on GitHub Free, enabling org secret scanning forces visibility public for private repos — the coupling documented in your register — so for private-repo orgs treat scanning as a paid decision, and use `gh api /repos/{repo}/commits/{sha}` to confirm the leaked string is gone from history.

**7.9 Verdict.** Eight stored credentials today; target **seven** (retire K7; K2 conditional; resolve K5 mapping) — and no further consolidation, because merging any K1–K5 pair across entities would violate boundary 1. Five is the right *machine* number; the extras are human and HCP tokens with distinct jobs.

# §4 Question 3 — S3 and the backend

**8.1 SCP algebra, student level.** a. AWS evaluates: service control policies (org ceiling) → identity policies → resource policies. An **explicit deny at any layer wins over any allow at any other layer**. b. Proven live: `list-buckets` returned `…not authorized … with an explicit deny in a service control policy: …/p-r9i2lq77` — so bucket creation for `viavitae-terraform` is impossible regardless of IAM. c. Paths: (1) management-account carve-out — ask the org admin (account `376896792604`); effort: their 10 minutes, risk: they say no or the deny is deliberate policy; (2) org admin creates bucket + delegates via resource policy — effort: theirs, risk: shared-account residency for a charity-adjacent plane; (3) new AWS account outside the org — effort: yours, ~15 min + payment setup, risk: another billing surface for ~$0–2/mo. Copy-paste request for (1): *"Please add an SCP exemption for account 775093708742 (or attach a more permissive SCP to its OU) allowing s3:* for the principal arn:aws:iam::775093708742:user/viavitae-terraform, needed for a Terraform state backend (bucket + DynamoDB lock table would be created in-region, versioned, Object Lock enabled). Alternative: create the bucket on my behalf with a resource policy scoped to that principal."*

**8.2 Does ADR‑0006 still hold?** jolarca: **yes** — HCP free tier fits (<500 resources), PCI-friendly isolation, already accepted and running. viavitae: the ADR logic (single-copy local state = D‑02-class blocking risk) applies equally → adopt the same decision, own HCP org. jol: `architecture.md:107` plans HCP by reference — **that plan contradicts the charity's in-house-hardware principle**; it was written by copying jolarca's reasoning without the residency constraint. Verdict: do not execute it as written (8.3).

**8.3 Residency.** a. jol state contains parish/diocese repo inventories — controller data of a charity whose stated principle is "built and operated in-house on hardware it owns". HCP = US processor → Art. 28 DPA + Art. 44–50 transfer analysis for a *plan file of infrastructure metadata*: low personal-data content, but the principle is a governance commitment, not a GDPR minimum — breaking your own published principle is an ISO A.5.2 (policy) finding even when GDPR tolerates it. b. Self-hosted HTTP backend on charity hardware: Terraform's `http` backend gives state + **ETag-based locking** + TLS; obligations: systemd service + reverse proxy + cert (ACME internal), the box joins backup scope, and its loss = loss of state unless §8.6 cadence holds. c. Interim (hardware not ready): local state + **off-host** age archive (the current same-disk archive does not qualify).

**8.4 Decision matrix** (verdict per row/plane below; ✅/⚠️/❌):

| Backend | enc-at-rest | locking | off-host dur. | residency | cost | solo burden | PCI | audit | key owner | key rests |
|---|---|---|---|---|---|---|---|---|---|---|
| local (today) | ❌ (no FDE) | ❌ | ❌ (same-disk backup) | ✅ | 0 | low | ❌ | ❌ | operator | disk |
| S3+native lock | ✅ SSE-KMS | ✅ `use_lockfile` | ✅ | ⚠️ US region | ~$1/mo/plane | med | ⚠️ | ✅ (CloudTrail) | operator (CMK) | AWS |
| HCP per org | ✅ HashiCorp-managed | ✅ | ✅ | ❌ for charity | 0 (free tier) | low | ✅ | ⚠️ (vendor) | **HashiCorp** | HashiCorp |
| self-hosted HTTP | ✅ (your disk, your TLS) | ✅ ETag | ⚠️ needs your backup | ✅ | ~0 (owned hw) | **high** | ✅ | ✅ | operator | charity box |
| git-crypt state | ❌ (key in clone) | ❌ | ✅ | ⚠️ GitHub | 0 | med | ❌ | ❌ | operator+GitHub | repo |

Verdicts: viavitae → **HCP**; jolarca → **HCP (stay)**; jol → **self-hosted HTTP on charity hardware, interim local+off-host age backup**; S3 → **contingency only**, and only for viavitae if the HCP free-tier limit ever binds (82 resources today, TF 1.16.1 supports `use_lockfile` — no DynamoDB needed).

**8.5 Isolation spec (HCP).** a. Three separate HCP organizations — one per GitHub org/entity — never three workspaces in one HCP org (that would recreate the merged-controller problem at the vendor). b. Each HCP org has its own workspace, its own `TFC_TOKEN` service account, its own variable set; no shared teams (none exist anyway). c. If S3 ever happens: one bucket per entity, one customer-managed KMS key per bucket, versioning + Object Lock (compliance mode) + Public Access Block at account and bucket level, bucket policy with `aws:PrincipalOrgID` absent (standalone account), no cross-account principals, `use_lockfile` = same-bucket lockfile.

**8.6 Backup cadence for hosted state.** a. HCP retains state versions, but the free-tier retention count/quota is **not documented in your registers and I will not assert it** — verify by API: `curl -H "Authorization: Bearer $TFC_TOKEN" https://app.terraform.io/api/v2/workspaces/ws-w6NSnTXXmYgh78Su/state-versions` (run from anywhere; count returned versions). b. Cadence control that would catch silent failure: monthly `terraform state pull | age -r <recipient> > <entity>-state-<UTC>.age` written to the **off-host** medium from §5 roadmap, plus a CI step that fails when the newest archive is >35 days old. c. This also gives jolarca a residency-neutral recovery path independent of the vendor's terms change (§3.2 paranoia clause).

**8.7 Migration runbook — viavitae local→HCP.** 1. Authorisation: written ADR amendment + explicit human sign-off. `STOP — human approval required before this step` applies to 5–8. 2. Baseline: `sha256sum terraform.tfstate` recorded in the plan doc; fresh age archive. 3. Capability probe: create HCP org `viavitae`, empty workspace, `TFC_TOKEN`; confirm `terraform login`-free CI auth works. 4. Destination build: workspace vars/secrets (`GITHUB_TOKEN` → K4's replacement — never K6). 5. Port the plan_gate guard first: `audit.yml` currently runs `terraform init -backend=false`, which **masks a create-heavy plan against empty state** — the exact hazard jol's `plan_gate.py` refuses; import that refusal before any migration step runs in CI. 6. `terraform init -migrate-state` — `STOP — human approval`; verify lineage `eff08856…` preserved in HCP (lineage change = you migrated onto, not into, state). 7. Equivalence diff: new plan must be **empty**; any non-no-op diff halts per §3.7 (destroy/replace → escalate with blast radius). 8. Stray cleanup: remove local `terraform.tfstate`/`.backup` after step 7 passes, keep the age archive. 9. Offline restore rehearsal: pull state version from HCP → age → diff against baseline hash. 10. Register bookkeeping: COMPLIANCE.md, D‑02-equivalent closure, inventory row.

**8.8 Verdicts.** viavitae: migrate to HCP per 8.7. jol: stay local with off-host age copies now, self-hosted HTTP backend when charity hardware exists; do not HCP. jolarca: HCP stands; add the §8.6 monthly export. S3: not an answer for anyone today — proven blocked, and every problem it solves is solved better per plane above.

# §5 Question 4 — Where tokens and passwords live

**9.1 Tiering.**

| Tier | Contents | Reader | Frequency | Encrypting key | Ciphertext lives | Key lives | Lost ⇒ |
|---|---|---|---|---|---|---|---|
| T0 root-of-trust | age identity, GitHub password + 2FA secrets/recovery codes, AWS root | operator, rarely | yearly | master passphrase (password mgr) / none (raw file) | host + sealed paper | host + sealed paper | total portfolio loss |
| T1 human creds | K6, replacement PATs, TFC_TOKEN | operator + CI setup | monthly | age (T0 recipient) | host vault | host | re-mint everything, prove identity to GitHub/AWS |
| T2 CI secrets | TF_GITHUB_TOKEN, ORG_READ_TOKEN, READONLY, TFC_TOKEN copies | Actions runtime | every run | **GitHub/HCP at-rest crypto** | vendor secret store | vendor | CI red, plan-only day |
| T3 app/runtime | per-plane app creds | workloads | continuous | platform vault | workload platform | platform | outage in one entity |

**9.2 Excel + Dropbox + TrueCrypt — named failures.** a. TrueCrypt: abandoned by its own authors in 2014, audit incomplete, unfixed CVE-class risks → ISO A.8.8 (technical vulnerabilities) alone rejects it. b. Single container = single decrypt = every secret of every controller → violates boundary 1 and PCI Req. 7; blast radius is the whole portfolio (CC6.1). c. No per-item ACL, no audit log (A.8.15), no expiry/rotation metadata — your §7.7 problem becomes unsolvable inside the tool. d. Dropbox: US processor, lawful-access exposure, Art. 28 DPA + Art. 44–50 transfer for a charity controller's credential map; sync + version history = a second, uncontrolled copy of every plaintext Excel write; shared-link misfire is the classic disclosure. e. Excel itself writes plaintext shadow/temp files, MRU lists, thumbnails **outside** the container — the ciphertext boundary is bypassed by the application layer (A.8.24, CC6.1) — exactly the worked example. f. Even encrypted-container sync leaks metadata: existence, size, mtime, access pattern. g. "Where's the master passphrase?" → it goes in… a spreadsheet. The design has no root-of-trust answer (the §12 anti-pattern).

**9.3 Scored comparison** (0–5; criteria order: crypto/auditability, offline survivability, solo usability, granular ACL, rotation/expiry tracking, breach detectability, GDPR/residency, PCI acceptability, incapacitated-recovery, cost):

| Option | cry | off | use | acl | rot | det | gdpr | pci | rec | cost | Σ |
|---|---|---|---|---|---|---|---|---|---|---|---|
| age file vault (existing pattern) | 4 | 5 | 4 | 2 | 2 | 2 | 5 | 4 | 4 | 5 | 35 |
| pass (age backend) | 4 | 5 | 5 | 3 | 3 | 2 | 5 | 4 | 4 | 5 | 40 |
| KeePassXC (KDBX local) | 4 | 5 | 4 | 2 | 3 | 2 | 5 | 4 | 3 | 5 | 35 |
| Vaultwarden (self-host) | 4 | 3 | 5 | 4 | 4 | 3 | 5 | 4 | 3 | 4 | 39 |
| Bitwarden cloud | 4 | 2 | 5 | 4 | 4 | 4 | 3 | 4 | 4 | 4 | 38 |
| FIDO2 key for T0 | 5 | 4 | 3 | 5 | 3 | 5 | 5 | 5 | 2 (loss=lockout) | 3 | 40 |
| Actions secrets (T2 only) | 4 | 0 | 5 | 5 | 3 | 4 | 4 | 5 | 0 | 5 | 35 |
| AWS SM / HCP Vault (T3) | 5 | 1 | 3 | 5 | 5 | 5 | 3 | 5 | 1 | 3 | 36 |

Winner for T0–T1: **pass + age backend** (highest Σ without a vendor), **plus** a FIDO2 key later as an upgrade to T0, not as a prerequisite.

**9.4 Verdict and target architecture.** a. Layered: T2/T3 stay exactly where they are (GitHub/HCP secret stores — they are correct tools for those tiers). T0+T1 move into a `pass`-style store backed by the **existing age identity pattern** — one sub-store per entity (`secrets/viavitae/…`, `secrets/jol/…`, `secrets/jolarca/…`), so the §9.2b single-container blast radius is avoided: decrypting one entity's tree never exposes another's. b. Ciphertext: `~/.password-store/` (age-encrypted files), versioned by a **local-only** git repo (history without a vendor). Key: the age identity, which itself gets the §9.5 sealed-copy treatment. c. Bootstrap (uses only host-present tools; `pass` is the one addition — install `sudo apt-get install pass` from the Ubuntu 24.04 archive, package `pass` 1.7.4, provenance: Ubuntu main/universe, signed):

```bash
umask 077
PUB=$(age-keygen -y ~/.config/age/state-backup-key.txt)
mkdir -p ~/.password-store/{viavitae,jol,jolarca}
git init -q ~/.password-store && git -C ~/.password-store config user.name op
for e in viavitae jol jolarca; do
  printf 'placeholder\n' | age -r "$PUB" -o ~/.password-store/$e/.keep.age
done
git -C ~/.password-store add -A && git -C ~/.password-store commit -qm 'ai:top vault skeleton'
# store:  age -r "$PUB" -o ~/.password-store/jol/K6.age <<<"$SECRET"   # then shred input file
# read:   age -d -i ~/.config/age/state-backup-key.txt ~/.password-store/jol/K6.age
```
d. Rejection records: Bitwarden/Vaultwarden rejected for T0/T1 (vendor or self-host service between you and your root keys; §9.3 offline scores); KeePassXC rejected (GUI-first, no per-entity tree discipline, git-unfriendly binary format); Excel/Dropbox/TrueCrypt rejected at §9.2.

**9.5 Single-owner recoverability — the deciding constraint.** a. Workstation total loss: age-encrypted state archives + vault git bundle copied **monthly to a second physical medium** (USB SSD kept off-host); restore = new machine + `apt install age terraform gh` + sealed T0. b. Loss of the age identity: it is the root — mitigate with a 2‑share XOR split of the 32‑byte seed: `share2 = share1 ⊕ seed`, `share1` random; two sealed paper envelopes (hex + CRC of the recipient public key as integrity proof: anyone can check `age-keygen -y` of the recombined seed matches the recorded recipient **without** revealing it). Envelopes live in two physical locations (home safe + bank/parish box). c. 90-day incapacitation: a third sealed envelope (instructions + one share) with a named person; they can *start* recovery but not complete it alone (2-of-2 needs both envelopes) — the one-person-witness problem replaced by a two-location control. d. Brain-wallet: every secret must exist in the vault or in GitHub/HCP custody — the §9.6 inventory's "storage location" column with value `memory` is a red flag the quarterly audit rejects. e. Annual rehearsal, dated: first Sunday of March — recombine shares in a temp dir, verify recipient match, decrypt one archive, re-seal; log line in the inventory. f. Honest limit: no design here gives *live* access during incapacitation without trusting someone; the residual is "portfolio frozen, not lost" — recorded in §10.

**9.6 Inventory, not a spreadsheet.** a. The real need is metadata without values. b. Schema — a markdown/CSV table per plane (adopt jolarca's existing `key-custody.md` pattern; columns): `secret_id | tier | entity | type | storage_location | encrypting_key | created | expires | rotation_interval | last_rotated | last_verified | blast_radius | control_id | recovery_path`. c. Three sample rows:

```
K4, T1, Via-Vitae, fine-grained PAT, GitHub+Actions secret TF_GITHUB_TOKEN, GitHub account, 2026-10, 2027-01-15→2027-07-01, 90d, -, plan+credential-check job, repo-admin-read+org member lists, CC6.1/A.5.17, UI delete
K6, T1, all-three, classic PAT, gh keyring, GitHub account, -, unknown, manual, -, org admin on 3 controllers, A.8.2/CC6.2, UI revoke+invalidate sessions
T0-age, T0, all-three, age X25519 identity, ~/.config/age/, passphrase-less, 2026-09-28, never, on-compromise, 2026-10-17, every backup+vault, total loss, A.5.30/CC6.1, 2-of-2 sealed envelopes
```
d. Location: committed **unencrypted** in each plane's repo (docs/) — names/expiries/blast-radius are low-severity disclosure (your PAT names are already visible in screenshots and CI logs) and the availability of the register *is* a control (A.5.30); what must never appear: values, URLs-with-tokens, the XOR shares. e. The expiry-CI gate of §7.7 reads this file — that pairing is what makes it not-Excel.

**9.7 Rotation calendar.** T0: never on schedule, only on compromise/loss. T1: 180 d max lifetime (FG PATs support it; classic K6 until migrated). T2: 90 d (already the `key-custody.md` rule — enforce portfolio-wide). T3: per workload, 90 d default. Out-of-band triggers: any leak notice, any unused-power notice left un-actioned 14 d, any person/account joining or leaving the GitHub accounts (IterVitae case), vendor ToS change, framework audit finding.

**9.8 Verdict.** Keep Excel out of it entirely: an age-backed per-entity vault with a metadata-only inventory file, off-host sealed 2‑of‑2 recovery of the root key, and CI that reads the inventory and yells before expiry — all of it reusing the `age` machinery you already run.

# §6 Control mapping (abridged to the load-bearing rows)

| Clause | Control | Q | Evidence artifact | Status |
|---|---|---|---|---|
| CC6.1 / A.5.15 | per-entity machine credentials, no cross-org token | 1,2 | §0 reconciliation, workflows' env blocks | implemented |
| CC6.3 / A.5.18 | expiry + staggered rotation + inventory gate | 2,4 | §7.7, inventory file | gap → §8 |
| A.8.2 | dormant write token trigger date | 2 | key-custody row for K2 | gap |
| CC6.7 / A.8.24 | age for state+vault; TLS for HTTP backend | 3,4 | state/README.md, §9.4 | partial (no FDE) |
| A1.2/A1.3 / A.5.30 | off-host encrypted copies + restore rehearsal | 3,4 | §8.6, §9.5 | gap → §8 |
| Art. 5(1)(f)/28/44–50 | residency-matched backends per controller | 1,3 | §8.3–8.4 | jol gap |
| PCI Req. 7/8/10 | boundary 3/4 read-write split, prod env gates | 1,2 | isolation-model.md, apply.yml | implemented |
| A.5.2 | single-source doctrine, registers match tree | 1 | §0 drift rows | gap → §8 |
| CC6.6 | push protection + secret-scanning plan-tier coupling noted | 2 | §7.8 | compensating |

# §7 Findings register

| ID | Sev | Entity | Root cause | Compensating today | Remediation | Effort |
|---|---|---|---|---|---|---|
| F‑01 | High | all | state + only backup + age key on one unencrypted disk (no FDE; LUKS only on swap) | age archives, sha256 round-trip | off-host medium + monthly cadence; FDE decision at next OS reinstall | 2 h + 1 h |
| F‑02 | Med | all | no backup cadence control; jol archive 6 d old at check | manual runbook | CI "newest archive >35 d ⇒ red" | 1 h |
| F‑03 | Med | jolarca | K2 write token unmapped/dormant | prod-env gates, prevent_destroy | map in key-custody.md; revoke 2026‑11‑15 if unused | 15 m |
| F‑04 | Med | Via‑Vitae | K4 org grants = unused power, de-grant never applied | CI notice | §7.3 UI edit + rerun proof | 15 m |
| F‑05 | Med | all | K7 IterVitae orphan classic PAT | inactive account | §7.6 retire | 5 m |
| F‑06 | Med | all | K6 classic admin:org, unknown expiry, extra `gist` scope | org 2FA | §7.4 migration; drop unused scopes now | 30 m + 2 h |
| F‑07 | High | Via‑Vitae | boundary 1 silent on Via‑Vitae pairings | convention | §1.4 four edits | 2 h |
| F‑08 | Low | jolarca | isolation-model D‑02 text contradicts live HCP state | ADR‑0006 | doc fix in same PR as F‑07 | 15 m |
| F‑09 | High | all | five expiries in one 10‑day window | none | §7.7 stagger | 1 h |
| F‑10 | Low | jol | register claimed classic PAT in CI; tree shows injected default + ORG_READ_TOKEN | — | correct register (v2 prompt already did) | 15 m |
| F‑11 | Med | jolarca | HCP state-version retention unverified | HCP vendor | §8.6a probe + monthly export | 30 m |
| F‑12 | Structural | all | single human identity = single point across 3 controllers | org 2FA, plan-only authority | §9.5 break-glass; accept residual | — |

# §8 Ordered roadmap (one person, part-time; → = prerequisite)

1. F‑08 + F‑10 register corrections (pure docs, no risk) — do first, cheap credibility.
2. F‑04 de-grant + CI proof (15 min UI, needs the human gate).
3. F‑05 retire IterVitae; F‑06 drop `gist` from K6.
4. F‑03 map K2 in `key-custody.md`, set the 2026‑11‑15 revoke trigger.
5. F‑01/F‑02: buy second medium → age archives off-host → CI staleness gate (→ nothing, parallel to 1–4).
6. F‑07 boundary 1 extension + viavitae fleet-separation guard (→ 1).
7. §9.6 inventory files in all three repos + §7.7 expiry CI gate (→ 1).
8. F‑09 December rotation to the staggered calendar (→ 7, so new dates land in the inventory).
9. §9.4 vault bootstrap + migrate T1 secrets (→ 5, because the vault needs the off-host copy too).
10. §9.5 sealed 2‑of‑2 envelopes + first rehearsal logged (→ 9).
11. F‑11 HCP retention probe + monthly export job (→ 5).
12. §8.7 viavitae→HCP migration (→ 5 plan-gate port inside step 5's CI work; last, because it's the only infra-mutating item).
13. jol self-hosted HTTP backend — when charity hardware exists (→ 10; unbounded date, interim controls cover it).

# §9 What I could not determine

- Exact permission sets of K1, K2, K3, K5 (screenshot shows names only; org FG-PAT inventory endpoint 404s — re-proven this session).
- Whether K2 is truly wired to `TF_GITHUB_TOKEN` and K5 to `TF_GITHUB_TOKEN_READONLY` (Actions secrets are write-only; inference from names only).
- Expiry/creation dates of K6, K7, K8 (classic PATs and HCP tokens not enumerated here).
- HCP free-tier state-version retention count (probe given, not run — needs TFC_TOKEN).
- Whether the SCP deny is deliberate org-wide policy or oversight (needs the management account).
- Current TrueCrypt/Dropbox terms-of-service text (not fetched this session; the §9.2 rejections do not depend on it).
- `dmsetup` as root (used `lsblk`/`/dev/mapper`/cmdline triangulation instead — conclusion unchanged, method corrected).
- Whether jol's apply.yml can create org-level repos on the injected default token (header comment says no; the successful-apply path from CI was not exercised).

# §10 Residual risk accepted

After the full roadmap: (1) one human is root of trust for three controllers — break-glass makes loss survivable, not access during incapacitation possible; (2) no FDE until reinstall — mitigated to "physical theft exposes unencrypted disk, but all backed-up secrets are age-encrypted and the age key itself is sealed 2‑of‑2"; (3) free-tier dependency (GitHub, HCP) — mitigated by export paths, not eliminated; (4) SCP carve-out outside operator control — accepted because the S3 path is contingency-only; (5) vendor metadata (GitHub sees your credential names/expiries) — inherent to the platform, tolerated per risk-based reading of Art. 5(1)(f). These are tolerable for a charity and a free-tier commercial entity because each has a named compensating control and a rehearsal date; none is silently waived.

# §11 Self-audit checklist

| Req | Answered in | Verdict given |
|---|---|---|
| 1.1 legal justification | §2.1.1 | yes |
| 1.2 separation table | §2.1.2 | yes |
| 1.3 shared-code | §2.1.3 | yes |
| 1.4 Via-Vitae gap | §2.1.4 | yes |
| 1.5 verdict | §2.1.5 | yes |
| 7.1 matrix (14 cols) | §3.7.1 | yes (cells compressed) |
| 7.2 dormant token | §3.7.2 | yes |
| 7.3 unused power | §3.7.3 | yes |
| 7.4 classic→FG | §3.7.4 | yes |
| 7.5 scope minimality | §3.7.5 | yes |
| 7.6 IterVitae | §3.7.6 | yes |
| 7.7 expiry calendar | §3.7.7 | yes |
| 7.8 rotation/revocation | §3.7.8 | yes |
| 7.9 count verdict | §3.7.9 | yes |
| 8.1 SCP + request text | §4.8.1 | yes |
| 8.2 ADR-0006 per plane | §4.8.2 | yes |
| 8.3 residency | §4.8.3 | yes |
| 8.4 decision matrix (10 cols) | §4.8.4 | yes |
| 8.5 isolation spec | §4.8.5 | yes |
| 8.6 hosted-state backup | §4.8.6 | yes |
| 8.7 migration runbook w/ gates | §4.8.7 | yes |
| 8.8 verdicts | §4.8.8 | yes |
| 9.1 tiers | §5.9.1 | yes |
| 9.2 rejection w/ clauses | §5.9.2 | yes |
| 9.3 scored table | §5.9.3 | yes |
| 9.4 architecture + bootstrap | §5.9.4 | yes |
| 9.5 recoverability + break-glass | §5.9.5 | yes |
| 9.6 inventory schema + rows | §5.9.6 | yes |
| 9.7 rotation calendar | §5.9.7 | yes |
| 9.8 verdict | §5.9.8 | yes |
| fmt: terms defined | §0, §4.8.1 (SCP), §5.9.1 | yes |
| fmt: tables ≥3 items | throughout | yes |
| fmt: commands + cwd | §8.6, §8.7, §9.4, §7.8 | yes |
| fmt: should ⇒ clause | §2.1.1, §4.8.3, §9.2 | yes |
| fmt: mutating steps gated | §7.3, §7.6, §7.8, §8.7 | yes |

---

Next (copy-paste):
a. `sudo apt-get install -y pass` — the only new tool the §9.4 architecture needs
b. Do the 15-minute UI items: F‑04 de-grant, F‑05 logout+revoke, F‑06 drop `gist`
c. Buy the second off-host medium (F‑01) and run the §9.4 bootstrap + §8.6b retention probe `curl -H "Authorization: Bearer $(gh secret… # see key-custody)" https://app.terraform.io/api/v2/workspaces/ws-w6NSnTXXmYgh78Su/state-versions`
d. Say the word and I'll turn roadmap items 1, 6, 7 into actual files (register corrections, boundary extension, inventory + expiry CI gate) as plan-only PRs