




## On "stop the Stripe test while there's no host"

Agreed — and the program is already structured so this costs nothing:

1. **The boundary is CLOSED by ratification** (ADR-009): test-mode only, no LIVE until SAQ A + change-controlled opening + owner authorization. With no production host, there is nothing to test *against* — continuing to exercise PSP test credentials would create audit noise without producing evidence.
2. **Hub-side testing survives the stop, unaffected**: the payment-events receiver's conformance suite (M-2's 13 tests + the dry-run evidence) uses **synthetic HMAC envelopes — zero Stripe credentials involved**. That stays green offline forever. What stops is PSP-side activity, not contract conformance.
3. **The stop completes with one owner action**: record the D-030 test-key revocation completion. Revoked credentials + CLOSED boundary + donations-OFF-per-tenant default + zero PSP surface in hub (guard-enforced) = the cleanest possible pre-host posture. Nothing to leak, nothing to scope you into PCI.
4. **Reactivation path is unchanged and documented**: SAQ A verification + opening plan + your explicit authorization (ADR-009 §4, donation-flow-spec §5).

## On the domains — they match the ratified topology exactly

`gyvenimo-kelias.lt` is named in the SEO strategy itself (weight matrix: ccTLD per country + tenant subdomains, D-006); `dzives-cels.lv` / `elu-tee.ee` are its LV/EE counterparts; `jol-hub.com` stays brand/marketplace-hub only. Two notes:

- **One open question**: `jol-marketplace` as a **subdomain of the LT ccTLD** is not what D-006 describes (ratified text places the marketplace hub on `jol-hub.com`). If the pilot runs marketplace at `jol-marketplace.gyvenimo-kelias.lt`, that's a deliberate pilot-scope addition and worth a one-line decision so the topology docs stay true.
- Your naming also partially answers **ASSUME-SEO-006** (LV/EE domains) — but registrar **ownership/control verification** for the three non-hub domains remains the outstanding action.

## To make it binding

If you want this recorded as ratification rather than just opinion, a decision line like:

```
stripe-test=SUSPENDED (until host + SAQ A) · marketplace-lt-subdomain=<APPROVED / jol-hub.com-only>
```


—and I'll record it verbatim, close the loop on D-030 when you confirm the revocation, and annotate the topology docs. No action taken until then.

---


**STOP — one field of your decision line is still the template.**

> `marketplace-lt-subdomain=<APPROVED / jol-hub.com-only>`

Both options still present inside the `<…>` bracket, neither chosen — and this one is purely your architecture call (pilot-scope marketplace on the LT ccTLD vs. the ratified jol-hub.com-only placement), so I won't pick it.

Noting for clarity: **`stripe-test=SUSPENDED (until host + SAQ A)` is complete and unambiguous** — I have it ready to record verbatim the moment the line is whole; nothing is blocking it except my unwillingness to split your decision line unilaterally.

Fill the second field in either direction, e.g.:

```
stripe-test=SUSPENDED (until host + SAQ A) · marketplace-lt-subdomain=APPROVED
```


…or, if you'd rather decouple them: say "record stripe-test only" and I'll record the suspension now, D-030-style, and leave the topology question open as a named item. Either way I'll annotate the SEO/topology docs to match, and no action happens before the ratification is on the record.

---

---

## What was recorded

| Decision | Entry | Effect |
|---|---|---|
| `stripe-test=SUSPENDED (until host + SAQ A)` | **D-050** | PSP-side test activity suspended — nothing to test against without a host. Scoped precisely: hub-side contract-conformance (synthetic HMAC suite) is **unaffected**; reactivation requires the full ADR-009 §4 chain (SAQ A + opening plan + your authorization). Suspension completes with your D-030 revocation-completion record |
| `marketplace-lt-subdomain=APPROVED` | **D-051** | `jol-marketplace.gyvenimo-kelias.lt` ratified as a **pilot-scope addition ON TOP of D-006** — the `jol-hub.com` brand/marketplace-hub role is explicitly unchanged, so the two marketplace placements coexist honestly in the docs |

**Docs updated to match the record** (topology truth kept single-sourced): SEO strategy §1 carries the domain-set addendum (all four owner-provided names, with the ASSUME-SEO-006 caveat), and §3.2's LT row now names the marketplace subdomain. ASSUME-SEO-006 partially answered — names on record, **registrar ownership/control verification of the three ccTLDs remains the outstanding action**.

## Professional opinion

1. The suspension decision is audit-clean in both directions: it creates a dated boundary ("no PSP activity after 2026-08-28") that any future examiner can verify against credentials logs, and it preserves the only testing that produces real evidence (the synthetic-envelope conformance suite).
2. Approving the LT marketplace subdomain was the right pragmatic call — but note it creates a **dual-marketplace topology** (pilot ccTLD subdomain + hub-domain marketplace). Before LV/EE rollout, decide whether they replicate the subdomain pattern or wait for the hub placement — one line in a future decision prevents topology drift.
3. Queue after this: D-030 revocation record (closes the suspension loop), O-010 safety data, registrar verification, and the push of the 2 tails when you call it.
---

Freeze recorded; O-021 reconciled — commit a14aedf9. Unpushed: 3 (4b603931, 787e4e83, a14aedf9).
1. What was recorded (verbatim, D-052…D-054)
D-052 — payment-track=FROZEN, your reason verbatim, plus the STANDING RULE: no payment/PSP/donation-implementation work without your explicit unfreeze. Frozen inventory: receiver dormant (flag default false) · donations OFF per tenant · SAQ A pending (sole LIVE condition) · D-030 revocation completion still NEEDS-OWNER
D-053 — donation-components=DEFERRED: the full donation component set is struck through the DS §6 backlog with a deferral note, and donation-flow-spec §5's build-ordering is superseded in priority. Pages 16/17 stay valid as design artifacts — only implementation is deferred. The store/marketplace UI trio is commerce UI, not payment-track work, and stays eligible under its own gates
D-054 — o021-widget-removal=PROCEED: reconciled, not re-executed. Measured this turn before touching anything: widget absent, zero PSP refs, guard exit 0, O-021 CLOSED. The work landed previously (2a79c3bf/2eca1a5e + infra fe75daf/71a33e2), and I will not fabricate the three stage commits that would duplicate it — the granularity gap stays recorded honestly. Your scope note stands: boundary hygiene ≠ payment work, so the freeze doesn't unwind it
2. Queue state after the freeze
Build-eligible now (non-payment): store/marketplace commerce UI trio, ChatbotEntry shell (AI-gated via safety.yml), CemeteryMapCanvas, CourseList, BreadcrumbBar/PaginationBar finishes, RecurrenceMaterializer — each under its own gates. Deferred until unfreeze: all donation components. Still yours: O-010 safety data, D-030 record, registrar verification, O-020, O-008, LV/EE retention, push of the 3 tails.
Professional opinion
The freeze is the strategically correct call and it lands cleanly because the program front-loaded the boundary work: everything payment-adjacent is already in its safest possible state (dormant receiver, OFF defaults, CLOSED boundary, mechanically-enforced zero-PSP surface). Freezing now costs zero rollback and preserves a crisp reactivation path — unfreeze → SAQ A → opening plan → your authorization. One benefit worth naming: deferring the donation components also defers their DPIA-adjacent dependency, which de-risks the AI safety work from being rushed to unblock payments.