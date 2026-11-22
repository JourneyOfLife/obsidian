

New VG per scope (vgvia, vgjol) rather than one shared vgopt, and mount both by /dev/mapper/<vg>-<lv>. /opt/jolarca is still fstab-by-UUID today (line 26), which is exactly the trap that nearly mounted the evidence image this evening.
Never put both orgs' only copy on one physical disk — today lv_viavitae and lv_jolarca share nvme2n1, so the drive is a shared failure domain for church and marketplace production.
Get state onto a backend (ADR-0006 / ADR-010 are already drafted) — viavitae-control/terraform.tfstate was this incident's only unrecoverable loss, and a remote backend deletes that class of loss permanently.
Runtime data (postgres/redis dumps, uploads, Docker volumes) is the part git clone cannot restore; that needs the off-device tier, and note the DR sticks sda/sdb are only 14.6 GB — they will not hold a 56 GiB image.