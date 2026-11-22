


Before I plan, I need ground truth from last night — a plan built on "it should have run" is worthless. Verifying the first *scheduled* cycle and the custodian's own health:

pbs01 half landed; the hv01 half came back empty. Also note the clock: **pbs01 reads 21:06Z Oct 22 = 00:06 EEST**, so the 02:00 job is ~2 h in the future — it is not overdue, and I won't pretend otherwise. Re-probing with corrected syntax:

Ground truth first — three things your plan must not be built on hope:

| Measured (00:06 EEST)    | Value                                            | Meaning                                                    |
| ------------------------ | ------------------------------------------------ | ---------------------------------------------------------- |
| job `hv01-nightly-pbs`   | `next-run 1792710000`, **no `last-run`**         | first scheduled run is **~2 h away**, unproven             |
| snapshots on pbs01       | `vm/100 1`, `vm/101 1`, `vm/102 1` (manual ones) | 2nd copy exists, **1 location**                            |
| `df /backup-pool`        | 69 G / 2.2 T (4 %)                               | headroom fine                                              |
| chrony                   | 0.00046 s, `ns1.telecom.lt`, 6 sources           | audit timestamps trustworthy ✔                             |
| token + ACL              | enabled, `Backup+Prune` on `jol-hv01`            | intact                                                     |
| **postfix**              | **active but 9 deferred requests, 38 KB**        | `mailnotification failure` → **failure alerts go nowhere** |
| `pbs-store-prune`        | 03:00, `keep-last=1, 7/4/6`, **ns empty**        | retention authority is split with the PVE job              |
| `remote list`            | empty                                            | **no replication — 3-2-1 violated, 0 off-site copies**     |
| hv01                     | `NO_EXPORTER`, `ufw inactive`                    | no metrics, no host firewall                               |
| `/systembackup` on pbs01 | absent                                           | the custodian's own config is unbacked                     |

