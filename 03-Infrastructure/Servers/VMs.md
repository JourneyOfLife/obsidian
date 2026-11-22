
Let me audit the VM layout across both hypervisors to determine the scope of the password exposure.

## VM Layout Audit — Password Exposure Impact Scope

Based on the inventory and topology docs, here's the complete VM layout:

### Hypervisor: **pve-prod-hv01** (10.60.60.20)
**Status**: Production — hosts AI service VMs

| VMID | Hostname | IP | Purpose | Data Sensitivity |
|------|----------|----|---------|------------------|
| 100 | rag-prod-lt01 | 10.40.40.10 | RAG services, vector DB, embeddings | **HIGH** — GDPR Art. 9 (religious data) |
| 101 | mcp-prod-lt01 | 10.40.40.11 | MCP tool server, agent integrations | MEDIUM — tool registry |
| 102 | her-prod-lt01 | 10.40.40.12 | Hermes agent orchestration | MEDIUM — workflows |

### Hypervisor: **prox01** (10.60.60.10) — **PASSWORD EXPOSED HERE**
**Status**: Production — hosts pilot + terraform infra VMs

| VMID | Hostname               | IP          | Purpose                         | Status                |
| ---- | ---------------------- | ----------- | ------------------------------- | --------------------- |
| 205  | tfstate-prod-01        | 10.60.60.32 | Terraform state DB (PostgreSQL) | **To be created**     |
| 206  | tf-runner-prod-01      | 10.60.60.31 | Terraform/Actions runner        | **To be created**     |
| 200  | jol-app-pilot-lt01     | 10.40.40.20 | JOL application tier            | Planned (not created) |
| 201  | jol-db-pilot-lt01      | 10.40.40.21 | PostgreSQL (parishioner PII)    | Planned (not created) |
| 202  | jol-bitrix-pilot-lt01  | 10.40.40.22 | Bitrix24 integration connector  | Planned (not created) |
| 203  | jol-ingress-pilot-lt01 | 10.45.45.10 | TLS termination, WAF            | Planned (not created) |
| 204  | jol-observ-pilot-lt01  | 10.60.60.30 | Monitoring (Prometheus/Loki)    | Planned (not created) |

### Other Bare-Metal Hosts (separate systems)
| Hostname | IP | Purpose | Password Reuse Risk |
|----------|----|---------|---------------------|
| llm-prod-lt01 | 10.30.30.10 | LLM inference (Ollama) | Check if reused |
| pbs01 | 10.10.10.30 | Proxmox Backup Server | Check if reused |

## Impact Analysis

**prox01 root password exposed** means:
- ✅ **Direct impact**: All VMs on prox01 (once created) — hypervisor root can console in, snapshot, migrate, destroy any VM
- ⚠️ **Potential impact**: If password reused on **pve-prod-hv01**, then RAG/MCP/Hermes VMs are also compromised
- ✅ **No direct impact**: VMs use SSH key-only access (per inventory), so hypervisor password doesn't give direct SSH to VMs
- ⚠️ **BUT**: Hypervisor root = full control over VM lifecycle (can access VM console, inject keys, etc.)

## Immediate Actions Required

**1. Rotate prox01 root password NOW** (you're already on prox01):
```bash
passwd
# Generate new: openssl rand -base64 24
```

**2. Check if password was reused on pve-prod-hv01**:
```bash
# From your workstation, try SSH to pve-prod-hv01 with the OLD password
# If it works, rotate it immediately too
ssh root@10.60.60.20
```

**3. Check other bare-metal hosts** (llm-prod-lt01, pbs01):
```bash
# These use SSH key-only per inventory, but verify password wasn't reused
ssh root@10.30.30.10  # llm-prod-lt01
ssh root@10.10.10.30  # pbs01
```

**4. After rotation, verify no suspicious access**:
```bash
# On prox01
last | head -20
journalctl -u ssh --since "2026-09-25" | grep -i "accepted"

# On pve-prod-hv01 (if password was reused)
last | head -20
journalctl -u ssh --since "2026-09-25" | grep -i "accepted"
```

## Summary

- **prox01**: Password exposed → rotate immediately
- **pve-prod-hv01**: Check if password reused → rotate if yes
- **VMs on prox01**: Not yet created, but will be at risk until hypervisor password is rotated
- **VMs on pve-prod-hv01**: At risk ONLY if hypervisor password was reused
- **Other hosts**: Use SSH key-only, but verify password wasn't reused

**Priority**: Rotate prox01 NOW, then check pve-prod-hv01.