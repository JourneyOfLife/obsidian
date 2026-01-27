### **Optimized Prompt: Enterprise Architecture & Implementation RFP**

**To:** Prospective Solution Architects & Vendors
**From:** Journey of Life (JOL) Technical Steering Committee
**Date:** 2025-12-29
**Subject:** **Request for Proposal (RFP) & Statement of Work (SOW): Paranoid-Compliant, Pan-European Digital Ministry Platform**

Excellent. Your prompt is already very strong—comprehensive, structured, and technically precise. My role is to refine it into a flawless, **actionable, and vendor-ready Request for Proposal (RFP) and Statement of Work (SOW) document.**

The optimizations focus on **eliminating ambiguity, enforcing paranoid compliance as the first principle, and structuring outputs for immediate procurement and execution.** I have integrated your strategic goals (400k sites, pan-EU scale, LLM-augmented ops) directly into the core requirements.



---

#### **1. Project Vision & Mandatory Philosophy**

You are to act as the **Paranoid Compliance-Driven Lead Architect** for the "Journey of Life" (JOL) initiative. This is not a technology project; it is the construction of **digital sacred infrastructure**. Your primary, non-negotiable directive is to design a system that is as **legally and canonically unassailable as it is technologically robust**.

The core design philosophy must be: ***"Assume every component will fail, every credential will be leaked, and every action will be audited by the EU Data Protection Board and the Vatican."*** Security and compliance are not features; they are the foundation.

#### **2. Project Scope & Scale: The "Digital Diocese"**

Design a unified platform to host, serve, and manage digital presence for faith-based communities and service providers across the 27 EU member states.

*   **A. Scale:** The final architecture must be designed to support a **total endpoint footprint of over 400,000 unique websites**. These are categorized as:
    *   **Ecclesiastical Institutions:** Roman Catholic (Basilicas, Dioceses, Cathedrals, Parishes), Protestant, Russian Orthodox, and other churches.
    *   **Service Providers:** Funeral Homes, Cemetery Maintenance Services.
    *   **Platforms:** A pan-EU Marketplace for religious goods, and the central JOL corporate hubs.
*   **B. Geographic & Domain Structure:** Each major country will have a dedicated, country-code top-level domain (ccTLD) portal (e.g., `gyvenimo-kelias.lt`, `podroz-zycia.pl`). These portals act as regional hubs, aggregating and governing the thousands of individual parish/service sites within their jurisdiction.
*   **C. Core Logical Services:** The architecture must distinctly host and isolate five core service verticals:
    1.  **Church & Diocese Portal**
    2.  **Funeral Services Platform**
    3.  **Cemetery Care Platform**
    4.  **EU Marketplace**
    5.  **JOL Corporate Hub & Central Services**

#### **3. Non-Functional Requirements (The "Paranoid Compliance" Benchmarks)**

These requirements supersede all functional desires. The system must be:

*   **3.1. Legally Unassailable:**
    *   **GDPR/Data Residency:** All personal data must be logically and physically isolatable per EU member state. Provide a detailed data sovereignty map.
    *   **EU AI Act Compliance:** Any AI/LLM component must be classifiable as a **limited-risk or minimal-risk system**. Full transparency, human oversight, and fundamental rights impact assessments are required.
    *   **Canon Law Alignment:** Data handling for sacramental records must have a defined canonical legal basis (e.g., *Lex Canonica*). Document this alignment.
*   **3.2. Architecturally Immutable:**
    *   **Zero-Trust Network:** Micro-segmentation down to the workload level. No implicit trust between services.
    *   **Immutable Audit Trail:** All system actions (user, admin, API, LLM) must generate cryptographically signed logs stored in a Write-Once-Read-Many (WORM) system, retained for a minimum of 7 years.
    *   **Cryptographic Agility:** All data encrypted at rest and in transit, with a designed path for post-quantum cryptography migration.
*   **3.3. Operationally Sovereign:**
    *   **Availability:** 99.95% uptime for core services. RTO ≤ 4 hours, RPO ≤ 15 minutes for critical databases.
    *   **Hybrid Hosting Model:** **80% on-premises/colo control, 20% cloud (Google Cloud)**. The cloud component is strictly for burst capacity, global CDN origins, and immutable off-site backups—not primary data processing.
    *   **Observability:** Unified observability stack (OpenTelemetry, Prometheus, Grafana) must provide **per-tenant, per-country, and per-service metrics**. Cost tracking must be as granular as performance tracking.

#### **4. Technology Stack & Architecture Constraints**

*   **4.1. Core Infrastructure:**
    *   **Central Orchestration:** **Kubernetes** for all scalable, cloud-native microservices (APIs, frontends, AI gateways).
    *   **Edge & Legacy Hosting:** **Proxmox VE** with LXC containers for legacy CMS instances (e.g., 1C-Bitrix), regional caching nodes, and small, isolated parish sites where a full K8s footprint is unjustified.
    *   **Hardware Standard:** Primary compute is **Supermicro AS-2025HS-TNR (2U, Dual AMD EPYC 9965)**. All BOMs must specify exact Supermicro part numbers.
*   **4.2. Specialized Infrastructure (Must be included in BOM):**
    *   **LLM Inference Cluster:** Specify options between **Supermicro AS-5126GS-TNRT (8x GPU)** with **NVIDIA H100 vs. L40S** GPUs, including a 3-year tokens/sec capacity model for 7B, 34B, and 70B parameter models.
    *   **High-Performance Storage:** Specify a **Supermicro all-flash NVMe array (e.g., SSG-229J series)** for Tier-0 storage (AI vector DBs, primary transactional databases).
*   **4.3. Software & Services:**
    *   **Mandatory Inclusions:** Next.js, Django, PostgreSQL, MongoDB, i18next, Cloudflare (WAF/CDN), Sentry, Web3.js/Ethers.js (for donation contracts), Bitrix24 CRM/1C-Bitrix CMS integration pathways.
    *   **AI/LLM:** Design must include a secured, gated **"Administrative Copilot"** for SRE tasks with a strict human-in-the-loop (HITL) approval workflow. Public-facing chat is a separate, isolated service.

#### **5. Required Deliverables (Vendor Must Provide)**

The proposal must be executable. Provide these as separate, detailed artifacts:

1.  **High-Level Architecture Diagram:** Show regions, clusters (K8s & Proxmox), network segmentation (Zero-Trust zones), and critical data flows. Must include a **Data Sovereignty Overlay**.
2.  **Detailed Bill of Materials (BOM):** Excel/CSV with line items for Years 1, 2, and 3. Include: Supermicro part numbers, CPU/RAM/GPU specs, NVMe/HDD counts, PSUs, rack units, network switches, and **estimated EUR pricing**. **Separate pricing for H100 and L40S GPU options.**
3.  **3-Year Capacity & Cost Plan:** Projected growth of node counts, rack space, power (kW), cooling (tons), and a full CAPEX/OPEX financial model.
4.  **Kubernetes Cluster Design:** Topology (multi-master, etcd), CNI/CSI choices, Ingress controller, service mesh (e.g., Linkerd), **autoscaling policies (HPA/VPA)**, and sample manifests for a web frontend and an LLM gateway service.
5.  **Proxmox Edge Design:** Standardized LXC template specification, automated provisioning playbook (Ansible), and backup/restore strategy to central storage.
6.  **Security & Compliance Blueprint:** Data residency mapping, encryption schematics (Key Management with HSM), SIEM/SOAR integration plan, and detailed audit procedure aligning with GDPR Articles 5, 25, 30, and 35.
7.  **Monitoring & SRE Runbook:** Definitive Grafana dashboard list, Prometheus alert rules (with thresholds), Sentry project structure, and a full Incident Response playbook with escalation matrix.
8.  **LLM Inference & Governance Plan:** GPU selection rationale, performance benchmarks (tokens/sec), model governance framework (training data provenance, bias testing), and operational cost per 1,000 tokens.
9.  **CI/CD & GitOps Workflow:** GitHub repository structure, branch strategy (e.g., GitFlow), Infrastructure as Code (Terraform for provisioning, Ansible for config, Helm for K8s), and full automated testing pipeline definition.
10. **Master Implementation Plan:** 12-month phased rollout schedule (Lithuania -> Baltics -> EU), with milestones, dependencies, and FTE resource estimates. Include a **migration plan from Proxmox edge nodes to central Kubernetes services.**

#### **6. Vendor Response Requirements & Evaluation Criteria**

*   **Response Format:** A single, consolidated PDF proposal with a link to a **private GitHub repository** containing all machine-readable artifacts (diagrams in SVG, BOMs in CSV, code in YAML/JSON).
*   **Key Vendor Disclosures:**
    *   Lead times for each major hardware component.
    *   Available 3-year and 5-year hardware support/warranty options.
    *   Team bios of assigned architects and engineers, highlighting compliance and large-scale infrastructure experience.
*   **Acceptance Criteria:** The winning proposal will be the one that most convincingly demonstrates how **every line of code, every network packet, and every logged event adheres to the "Paranoid Compliance" philosophy**, while meeting the technical and scale requirements.

---
### **Clarifying Questions Before You Proceed:**

To ensure the final architecture is flawless, I require clarification on these critical points:

1.  **Data Sovereignty Depth:** For GDPR, must user data (e.g., donor records, prayer requests) be *physically* stored within their country of origin, or is *logical isolation* within a secure EU region sufficient? This drastically changes storage architecture.
2.  **Traffic & Performance Benchmarks:** What are the **concurrent user estimates** for the 400,000 sites? Is this 400,000 low-traffic brochure sites, or does it include high-traffic portals? Define a "standard parish site" in terms of expected visits/day and bandwidth.
3.  **Legacy CMS Integration:** The 1C-Bitrix requirement—are these existing instances to be migrated and *hosted* on this platform, or does the platform need to *integrate* with externally hosted Bitrix instances via API?
4.  **Disaster Recovery Stance:** The RTO/RPO is clear. What is the **approved data loss window** for the 400,000 websites in a severe regional outage? This defines the replication strategy between the 80% on-prem and 20% cloud components.
5.  **Organizational Capacity:** Who will operate this? Is there an existing SRE/DevOps team, or does the implementation plan need to include the **stand-up of a new 24/7 NOC and the training of parish IT volunteers** on the edge node procedures?

With answers to these, I will provide a final specification that is unambiguous, compliant by design, and ready for vendor procurement.