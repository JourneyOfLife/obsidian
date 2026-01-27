

### Goal and scope

You want a single, professional prompt that you can give to an expert‑level systems architect/DevOps engineer (Kubernetes, Proxmox, GitHub, Linux) to design, procure, and operate a GDPR‑compliant, highly available platform that will monitor and serve **~400,000 church‑related websites across 27 European countries**, plus run a multi‑tier application ecosystem (church sites, funeral services, cemetery cleaning, marketplace, corporate JOL) and an LLM inference cluster. I will assume you want a single, self‑contained prompt that is **comprehensive, structured, and actionable** for vendor RFQs, architecture design, and implementation planning.

---

### Assumptions I will use

- **Scale**: ~400,000 websites; initial fleet ~100 servers; growth to multi‑rack cluster for Europe.
- **Regions**: Lithuania primary; EU regional presence (DE/NL) for data residency and latency.
- **Compliance**: Full GDPR compliance, Lithuanian legal constraints for local data where required.
- **Workloads**: Web/CRM (Bitrix24), marketplace, transactional services, background processing, monitoring, LLM inference (GPU cluster).
- **Tech choices**: Kubernetes for cloud and large clusters; Proxmox LXC for lightweight on‑prem edge nodes; GitHub for CI/CD; Linux servers (EPYC) and Supermicro hardware.
- **Security posture**: Paranoid by default — strict separation of duties, immutable logs, HSM for keys, per‑country data isolation.
- **Deliverables expected from the expert**: architecture diagrams, BOM with Supermicro SKUs, deployment playbooks, Kubernetes manifests, Proxmox templates, monitoring and incident runbooks, cost and capacity plan for 3 years.

---

### The optimized prompt to give an expert

Use this exact prompt when engaging an architect, vendor, or senior engineer:

> **Role**  
> You are a senior systems architect and DevOps engineer with 30+ years of experience in Linux, Kubernetes, Proxmox, GitHub, and enterprise compliance. Act as a paranoid, compliance‑driven architect designing a pan‑European platform for the “Journey Of Life” organization.
> 
> **Mission**  
> Design, specify, and produce an implementation plan for a platform that will:
> 
> - Monitor and serve **~400,000 church‑related websites** across 27 EU countries with per‑country domains (examples: gyvenimo‑kelias.lt, dzives‑cels.lv, elu‑tee.ee, podroz‑zycia.pl).
> - Host and operate five logical services: Church sites, Funeral Services, Cemetery Cleaning Services, Marketplace, and Journey Of Life corporate systems.
> - Provide an LLM inference cluster for chat and automation, with capacity planning for initial and 3‑year growth.
> - Meet GDPR and Lithuanian legal requirements for data residency, logging, and auditability.
> 
> **Nonfunctional requirements**
> 
> - **Availability**: N+1 for compute tiers; RTO ≤ 4 hours for critical services; RPO ≤ 1 hour for transactional DB.
> - **Security**: Zero trust network segmentation, HSM for keys, WAF, SIEM, immutable audit logs, per‑country data isolation.
> - **Scalability**: Horizontal scaling for web tier and GPU inference; autoscaling policies for Kubernetes.
> - **Observability**: OpenTelemetry tracing, Prometheus metrics, Grafana dashboards, Sentry for errors, centralized log retention with tamper evidence.
> - **Cost control**: CAPEX/OPEX plan for 3 years with cloud burst options for peak events.
> 
> **Architecture constraints and preferences**
> 
> - **Edge vs central**: Use Proxmox LXC for small on‑prem parish edge nodes where required by law; use Kubernetes clusters in EU DCs for central services.
> - **Hosting split**: 80% Linux on‑prem or colocation; 20% Google Cloud for burst, backups, and regional failover.
> - **Hardware**: Supermicro servers for on‑prem racks; specify exact SKUs and power/cooling requirements.
> - **Software stack**: Next.js for master SSR sites; Django + PostgreSQL + MongoDB for backend; Bitrix as CMS/CRM where required; i18next for localization; Cloudflare for edge protection; Web3/Ethers.js optional for donation contracts.
> 
> **Deliverables**  
> Provide the following artifacts, each as a separate deliverable:
> 
> 1. **High‑level architecture diagram** showing regions, clusters, network segmentation, and data flows.
> 2. **Detailed BOM** with Supermicro part numbers, CPU/RAM/GPU choices, NVMe/HDD counts, PSUs, rack units, and estimated EUR pricing for Year‑1 and Year‑2 purchases.
> 3. **3‑year capacity plan** with node counts, rack units, power, cooling, and cost projections (CAPEX + OPEX).
> 4. **Kubernetes design**: cluster topology, namespaces, ingress, service mesh recommendation, autoscaling policies, and sample manifests for web, API, and LLM gateway.
> 5. **Proxmox design**: LXC templates, backup/restore plan, and edge provisioning playbook for parish servers.
> 6. **Security and compliance plan**: data residency mapping, encryption at rest/in transit, key management, logging retention, and audit procedures.
> 7. **Monitoring and SRE runbooks**: Prometheus/Grafana dashboards, Sentry integration, alerting thresholds, incident response steps, and escalation matrix.
> 8. **LLM inference plan**: GPU selection options (H100 vs L40S vs Blackwell), per‑node tokens/s estimates for 8B/14B/34B/70B models, model governance, and cost per 1,000 tokens.
> 9. **CI/CD and GitHub workflow**: repo layout, branch strategy, IaC (Terraform/Ansible/Helm), and automated testing pipeline.
> 10. **Operational runbook**: backup schedule, DR test plan, patching cadence, and vendor support SLAs.
> 
> **Acceptance criteria**
> 
> - All diagrams and BOMs are vendor‑ready and RFQ‑ready.
> - Performance targets validated by load tests or conservative benchmarks.
> - Compliance checklist mapped to GDPR articles and Lithuanian requirements.
> - Implementation plan includes timelines, milestones, and resource estimates.
> 
> **Constraints for the vendor response**
> 
> - Provide separate pricing for H100 and L40S GPU options.
> - Provide lead times for each major component.
> - Provide 3‑ and 5‑year support/warranty options.
> - Provide a migration plan from Proxmox edge nodes to Kubernetes central services.
> 
> **Format of the response**  
> Deliver a single PDF and a GitHub repo containing: architecture diagrams (SVG), BOM spreadsheet (CSV), Kubernetes manifests (YAML), Proxmox templates, and runbooks (Markdown). Include a short executive summary and a 12‑month rollout schedule.

---

### Actionable next steps I will take for you

I will proceed using the assumptions above and produce the RFQ spreadsheet with exact Supermicro SKUs, part numbers, and a populated BOM if you confirm. I will also produce a concise Kubernetes cluster manifest template and a Proxmox LXC template for parish edge nodes.

---

### Final note about scope and clarifications

I assumed the following and will act on them unless you tell me otherwise: **use Supermicro hardware**, **Kubernetes for central clusters**, **Proxmox for edge**, **EU regional DCs for data residency**, and **H100 or L40S GPU options** for LLM inference. If any of these assumptions are incorrect, tell me which to change and I will regenerate the prompt and deliverables accordingly.