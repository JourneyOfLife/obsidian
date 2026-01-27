

## **Comprehensive MCP Server Architecture Design Prompt for Journey Of Life (JOL)**

### **Role Specification**
You are a **Senior Enterprise Architect** with 30+ years of experience in mission-critical systems design, specializing in AI infrastructure, regulatory compliance (GDPR, ISO 27001), and distributed systems. You operate with **paranoid-level security consciousness** and zero-tolerance for architectural debt. Your designs must withstand penetration testing, regulatory audits, and production-scale operational demands.

---

### **Project Context**
**Organization:** JourneyOfLife (GitHub: https://github.com/JourneyOfLife)  
**Mission:** Revolutionizing ministry and ceremonial services across Europe through multilingual, AI-powered, GDPR-compliant technology  
**Infrastructure Profile:**
- **Development Environment:** Obsidian (Windows PC) managing ~100 knowledge files
- **Code Repository:** GitHub organization with core repositories including `journey-of-life-core`
- **Production Infrastructure:**
  - Proxmox VE 9.1 Hypervisor (192.168.8.101) - Primary JOL orchestration host
  - Ubuntu Server 22.04 LTS (192.168.8.55) - Dedicated JOL application server
- **Compliance Requirements:** GDPR Article 32 (security of processing), ISO 27001:2022, EU AI Act preliminary compliance

---

### **Architectural Design Requirements**

#### **1. MCP (Model Context Protocol) Server Core Architecture**
**Objective:** Design a decoupled, metadata-driven MCP server that serves as the AI orchestration layer for JOL services.

**Step-by-Step Implementation Specification:**

**1.1. Decoupled Tool Implementation Framework**
```
a) Tool Abstraction Layer:
   - Implement interface-based tool definitions using TypeScript interfaces or Python ABCs
   - Each tool must expose: name, description, parameters schema, execution context, and compliance metadata
   - Example tool structure:
     {
       "tool_id": "ceremony_scheduling",
       "version": "1.2.0",
       "description": "GDPR-compliant scheduling for religious ceremonies",
       "parameters": {
         "type": "object",
         "properties": {
           "participant_count": {"type": "integer", "minimum": 1, "maximum": 500},
           "location": {"type": "string", "format": "iso-3166-1-alpha2"},
           "data_processing_consent": {"type": "boolean", "required": true}
         },
         "required": ["participant_count", "location", "data_processing_consent"]
       },
       "compliance": {
         "gdpr_article": "6(1)(a)",
         "data_retention_days": 30,
         "encryption_at_rest": true,
         "audit_logging": true
       }
     }

b) Execution Isolation:
   - Containerize each tool using Docker with resource limits (CPU: 0.5 cores, Memory: 512MB)
   - Implement network segmentation: tools communicate via internal MCP bus only
   - Enforce read-only filesystems except for designated /tmp and /data volumes
   - Apply seccomp profiles and AppArmor policies for system call restriction

c) Dynamic Tool Registration:
   - Create REST API endpoint /api/v1/tools/register with mutual TLS authentication
   - Implement automated schema validation against JSON Schema Draft 2020-12
   - Require digital signatures for tool manifests using ECDSA P-384
   - Maintain audit trail of all tool registration/modification events
```

**1.2. Metadata Management System**
```
a) Metadata Schema Design:
   - Define metadata taxonomy covering: tool capabilities, compliance attributes, performance characteristics, and operational constraints
   - Implement metadata versioning with semantic versioning (SemVer 2.0)
   - Store metadata in PostgreSQL with JSONB columns for flexible querying
   - Create metadata validation pipeline using JSON Schema validation

b) Real-time Metadata Synchronization:
   - Implement WebSocket-based metadata push notifications to connected clients
   - Build metadata cache with 5-second TTL and cache invalidation on schema changes
   - Design metadata backup strategy: hourly snapshots to encrypted S3 bucket
   - Implement metadata integrity checks using SHA-256 checksums

c) Compliance Metadata Enrichment:
   - Automatically tag metadata with GDPR processing categories based on tool functionality
   - Integrate with EU AI Act risk classification system (minimal/limited/high/unacceptable)
   - Generate automated compliance reports in PDF/CSV formats
   - Implement metadata access controls using RBAC with 4-tuple permissions (user, role, resource, action)
```

#### **2. Qwen.ai Integration Architecture**
**Objective:** Establish secure, compliant communication between JOL MCP Server and Qwen.ai API endpoints.

**Step-by-Step Implementation Specification:**

**2.1. Secure Connection Framework**
```
a) Connection Security:
   - Implement mutual TLS (mTLS) with certificate pinning for all outbound connections
   - Use TLS 1.3 with ECDHE key exchange and AES-256-GCM encryption
   - Configure connection pooling with maximum 10 concurrent connections per Qwen endpoint
   - Implement circuit breaker pattern with 5-second timeout and exponential backoff

b) Data Protection in Transit:
   - Apply end-to-end encryption using AES-256-GCM for sensitive payloads
   - Implement payload signing with HMAC-SHA256 using rotating keys (key rotation every 24 hours)
   - Strip PII from request payloads before transmission using automated redaction engine
   - Maintain detailed audit logs of all data transmissions with cryptographic hashes

c) Rate Limiting and Quotas:
   - Implement token bucket algorithm with configurable rates per Qwen API endpoint
   - Create emergency brake system that stops all Qwen communications if error rate exceeds 5%
   - Monitor API usage against contractual limits with real-time alerts at 80% threshold
   - Maintain local cache of Qwen responses with 15-minute TTL to reduce API calls
```

**2.2. Request/Response Transformation Layer**
```
a) GDPR-Compliant Data Processing:
   - Implement automatic PII detection and redaction using spaCy NER models
   - Create data minimization pipeline that removes non-essential fields before Qwen transmission
   - Apply differential privacy techniques for statistical queries (epsilon = 0.1)
   - Generate data processing records (DPRs) for every Qwen interaction

b) Context Preservation:
   - Design context window management system that maintains session state across multiple Qwen calls
   - Implement context compression using semantic summarization to stay within token limits
   - Create context versioning system with cryptographic hashes for auditability
   - Build context fallback mechanism that gracefully degrades when Qwen is unavailable

c) Response Validation and Sanitization:
   - Implement JSON schema validation for all Qwen responses against expected schemas
   - Apply content safety filtering using multiple AI safety models (toxicity, bias, compliance)
   - Create response caching layer with content-based invalidation (SHA-256 hashing)
   - Implement response signing verification to prevent man-in-the-middle attacks
```

#### **3. Infrastructure Deployment Specification**
**Objective:** Deploy the MCP server across the specified JOL infrastructure with zero-downtime capabilities.

**Step-by-Step Implementation Specification:**

**3.1. Proxmox VE 9.1 (192.168.8.101) Configuration**
```
a) Virtual Machine Template:
   - Create Ubuntu 22.04 LTS template with 4 vCPUs, 8GB RAM, 100GB SSD storage
   - Apply CIS Benchmark Level 1 hardening guidelines
   - Configure network interfaces: management (VLAN 10), application (VLAN 20), backup (VLAN 30)
   - Implement full-disk encryption using LUKS with TPM 2.0 integration

b) Container Orchestration:
   - Deploy Kubernetes cluster using k3s with 3-node HA configuration
   - Configure Calico CNI with network policies enforcing zero-trust architecture
   - Implement resource quotas: CPU (2 cores per namespace), Memory (4GB per namespace)
   - Set up pod security policies with restricted privilege levels

c) Backup and Recovery:
   - Configure Veeam Backup & Replication with 15-minute RPO and 1-hour RTO
   - Implement immutable backups stored on separate physical media
   - Create automated disaster recovery runbooks with step-by-step validation checkpoints
   - Perform quarterly backup restoration drills with full validation
```

**3.2. Ubuntu Server (192.168.8.55) Application Deployment**
```
a) System Hardening:
   - Apply CIS Ubuntu 22.04 Level 1 benchmark with additional custom rules
   - Configure AppArmor profiles for all application processes
   - Implement file integrity monitoring using AIDE with daily checks
   - Set up centralized logging with rsyslog forwarding to SIEM system

b) MCP Server Deployment:
   - Deploy using Helm charts with GitOps workflow (ArgoCD)
   - Implement blue-green deployment strategy with automated rollback triggers
   - Configure horizontal pod autoscaling based on CPU (70% threshold) and request queue depth
   - Set up liveness and readiness probes with 30-second initial delay

c) Monitoring and Alerting:
   - Implement Prometheus/Grafana monitoring stack with custom dashboards
   - Configure alerts for: CPU > 80%, memory > 85%, error rate > 1%, latency > 500ms
   - Set up distributed tracing using Jaeger for end-to-end request tracking
   - Create compliance monitoring dashboard showing real-time GDPR compliance status
```

#### **4. Compliance and Security Controls**
**Objective:** Ensure the MCP server architecture meets all regulatory requirements and security best practices.

**Step-by-Step Implementation Specification:**

**4.1. GDPR Compliance Framework**
```
a) Data Protection by Design:
   - Implement data minimization principles at every architectural layer
   - Create automated data retention policies with automatic deletion workflows
   - Design pseudonymization system for user identifiers using cryptographic hashing
   - Implement consent management system with granular opt-in/opt-out capabilities

b) Data Subject Rights Automation:
   - Build automated DSAR (Data Subject Access Request) processing pipeline
   - Implement right to erasure workflows with cryptographic deletion verification
   - Create data portability system supporting JSON, CSV, and PDF formats
   - Design automated consent withdrawal propagation across all system components

c) Breach Notification System:
   - Implement real-time breach detection using ML-based anomaly detection
   - Create automated notification workflow with 72-hour SLA tracking
   - Build evidence preservation system for breach investigations
   - Design notification templates for different breach scenarios
```

**4.2. Security Testing and Validation**
```
a) Penetration Testing Framework:
   - Perform quarterly external pentests by CREST-certified professionals
   - Implement automated DAST scanning in CI/CD pipeline using OWASP ZAP
   - Conduct monthly internal vulnerability scans with Nessus
   - Perform annual red team exercises with scope covering all attack surfaces

b) Code Security Analysis:
   - Integrate SAST tools (SonarQube, Checkmarx) in pull request workflow
   - Implement dependency scanning using Snyk with CVSS score thresholds
   - Require peer code reviews with minimum 2 approvers for security-critical code
   - Maintain software bill of materials (SBOM) for all dependencies

c) Runtime Protection:
   - Deploy RASP (Runtime Application Self-Protection) using Sqreen
   - Implement WAF rules using ModSecurity with OWASP Core Rule Set
   - Set up behavioral anomaly detection using machine learning models
   - Create automated incident response playbooks with containment procedures
```

---

### **Deliverables Specification**

#### **Phase 1: Architecture Design Document (Due: 5 business days)**
1. **System Context Diagram** showing all components and data flows
2. **Security Threat Model** using STRIDE methodology with mitigation strategies
3. **Compliance Matrix** mapping controls to GDPR, ISO 27001, and EU AI Act requirements
4. **Infrastructure Blueprint** with network topology and security zones
5. **Data Flow Diagrams** showing PII handling across all system boundaries

#### **Phase 2: Implementation Plan (Due: 10 business days)**
1. **Detailed Build Instructions** for each infrastructure component
2. **Configuration Management Scripts** (Ansible/Terraform) for repeatable deployment
3. **Test Plans** for functional, security, and compliance testing
4. **Disaster Recovery Runbooks** with step-by-step recovery procedures
5. **Operational Runbooks** for day-to-day management and monitoring

#### **Phase 3: Validation and Certification (Due: 15 business days)**
1. **Security Assessment Report** from independent third-party auditor
2. **Compliance Certification Package** ready for regulatory submission
3. **Performance Benchmark Report** showing scalability and reliability metrics
4. **User Acceptance Testing Results** with sign-off from key stakeholders
5. **Production Deployment Plan** with zero-downtime cutover strategy

---

### **Success Criteria**
- **Security:** Zero critical/high vulnerabilities in final security assessment
- **Compliance:** 100% audit pass rate for GDPR and ISO 27001 requirements
- **Performance:** 99.95% uptime SLA with <100ms p95 response time
- **Scalability:** Support for 10x current load with linear resource scaling
- **Maintainability:** Full infrastructure as code with automated testing coverage >85%

### **Quality Gates**
1. **Architecture Review Board** approval before implementation begins
2. **Security Champion** sign-off at each development milestone
3. **Compliance Officer** validation before production deployment
4. **Performance Testing** passing all SLA requirements
5. **User Acceptance Testing** with minimum 95% satisfaction score

---

**Final Architectural Principle:** *"When in doubt, choose the more secure, more compliant, and more auditable option - even if it requires additional engineering effort. The cost of failure in this domain is measured in human trust, regulatory penalties, and reputational damage that cannot be quantified in monetary terms."*