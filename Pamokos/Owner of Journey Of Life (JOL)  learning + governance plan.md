
### Purpose and deliverables

You want a single, professional, actionable prompt and a learning + governance plan so you — as owner of **Journey Of Life (JOL)** — can hire, manage, and audit DevOps engineers without being misled. Below I deliver three things in one response:

1. **Optimized vendor/engineer prompt** you can give candidates or contractors.
2. **Owner’s governance checklist** and red‑flags to spot deception.
3. **Fast, practical learning syllabus** (student‑friendly, step‑by‑step) so you can get competent quickly in Proxmox, Kubernetes, Cloudflare, GitHub, and the hosting strategy.

---

### 1. Optimized prompt for hiring contractors and vendors

Use this exact text when posting a job, requesting proposals, or briefing a contractor. It is concise, testable, and vendor‑ready.

> **Role**  
> You are a senior DevOps/Kubernetes/Proxmox engineer and systems architect with 7+ years of production experience in Linux, container orchestration, GitOps, and secure multi‑tenant platforms. Act as a paranoid, compliance‑first engineer for a startup (Journey Of Life) building a hybrid hosting platform for church websites and services across the EU.
> 
> **Scope**  
> Deliver a production‑ready implementation for a pilot in Lithuania including: Proxmox LXC edge templates, a Kubernetes core cluster (control plane + workers), CI/CD pipelines (GitHub Actions + ArgoCD), Cloudflare integration, HashiCorp Vault secrets, and an LLM inference gateway prototype. Provide documentation, automated tests, and a 30‑day runbook.
> 
> **Nonfunctional requirements**
> 
> - **Data residency**: Lithuanian canonical data must remain in Vilnius; enforce via node affinity and admission controllers.
> - **Security**: Zero trust, mTLS service mesh, HSM or Vault for keys, immutable audit logs.
> - **Availability**: N+1 for critical services; RTO ≤ 4 hours; RPO ≤ 1 hour.
> - **Observability**: Prometheus + Grafana, OpenTelemetry traces, Sentry for errors, centralized logs with tamper evidence.
> 
> **Deliverables** (must be machine‑readable)
> 
> 1. Proxmox LXC templates and Ansible playbooks for edge provisioning.
> 2. Kubernetes manifests and Helm charts for Next.js frontend, Django API, and LLM gateway.
> 3. GitHub repo with CI pipelines, IaC (Terraform), and ArgoCD app definitions.
> 4. BOM for Supermicro hardware with two GPU options (L40S and H100) and lead times.
> 5. Security & compliance checklist, audit log design, and a 30‑day SRE runbook.
> 6. Automated tests and load test results for the pilot scale.
> 
> **Acceptance criteria**
> 
> - All code in GitHub with PRs, code review, and passing CI.
> - Demo: deploy a parish site from repo to Proxmox and to K8s with identical container images.
> - Proof: show immutable audit log entry for a sample sync and a successful Vault secret rotation.
> - Documentation: architecture diagram, runbooks, and a 12‑month capacity plan.
> 
> **Constraints**
> 
> - No secrets in code or CI logs.
> - Use Cloudflare for edge protection.
> - Provide separate pricing for H100 vs L40S GPU options.
> 
> **Proposal must include** timeline, milestones, fixed price for pilot, and 3‑year support options.

---

### 2. Owner’s governance checklist and anti‑deception controls

Use this checklist during hiring, sprint reviews, and acceptance gates. Each item is testable and non‑negotiable.

#### A. Contract and procurement controls

- **Fixed deliverables and milestones** with payment tied to acceptance tests.
- **Source code escrow clause** and IP assignment.
- **SLA and support terms**: response times, on‑site spares, warranty.
- **Proof of competence**: require links to public repos, references, and a short technical take‑home task.

#### B. Technical acceptance tests you must require

- **No secrets in repo**: run `git grep` for `secret`, `password`, `client_secret`. CI must fail if found.
- **Vault demo**: contractor must rotate a secret in Vault and show the app picks it up without redeploy.
- **Immutable audit log**: show a signed log entry stored in the audit store and demonstrate inability to alter it without detection.
- **Data residency test**: attempt to replicate a Lithuanian record to a non‑LT node; system must block and log the attempt.
- **End‑to‑end deploy**: from PR → CI → ArgoCD → production namespace with a single button.
- **Load test**: run a scripted load test and provide metrics (RPS, p95 latency, error rate).

#### C. Code review and CI rules you must enforce

- **Require PRs for all changes**; no direct pushes to `main`.
- **Mandatory reviewers**: at least one security reviewer and one architect.
- **Automated checks**: linting, unit tests, container image scanning, SCA (software composition analysis).
- **Signed commits** for release branches.

#### D. Operational controls and transparency

- **Daily standups and weekly demos** with recorded sessions.
- **Access control**: contractors get time‑limited, least‑privilege accounts; all actions logged.
- **Change windows**: no production changes outside agreed windows without emergency approval.
- **Monthly security review**: vulnerability scan, dependency updates, and patching report.

#### E. Red flags that mean “stop and audit”

- Vague answers to “where are secrets stored” or “show me the Vault policy”.
- Refusal to provide a reproducible demo in your environment.
- Frequent “works on my machine” excuses without CI evidence.
- Requests to bypass procurement or to use personal cloud accounts.
- Deliverables that are “hand‑wavy” diagrams but no code or tests.

---

### 3. Practical hiring tests and interview questions

Use these to vet candidates quickly.

#### Technical take‑home task (4–8 hours)

- **Task**: Provide a GitHub repo with:
    - A Dockerfile for a simple Django API and a Next.js frontend.
    - A GitHub Actions workflow that builds images and pushes to a registry.
    - A Helm chart that deploys both to a local K8s cluster (kind/minikube).
    - A short README showing how to run the app and how to rotate a secret via Vault (mocked).
- **Evaluation**: code quality, CI passing, clear README, and a short video walkthrough.

#### Interview questions (must be answered live)

- Explain how you would enforce Lithuanian data residency in Kubernetes.
- Show how you would rotate a DB password in Vault without downtime.
- Describe how you would detect and respond to a compromised node.
- Walk me through an immutable audit log design and how to prove tamper evidence.
- Explain how you would scale LLM inference when latency spikes.

---

### 4. Learning path for the owner and students

A fast, practical syllabus to become competent in 1–3 months with daily 6–8 hour sprints. Tailored for **intermediate learners** (some Linux and networking). Each week has labs and deliverables.

#### Structure and pacing

- **Target audience**: intermediate (some Linux).
- **Timeframe**: 8 weeks intensive or 12 weeks paced. Daily 4–6 hours recommended.
- **Format**: mixed — short videos (20–40 min), written guides, hands‑on labs, GitHub projects, quizzes.
- **Environment**: use your AlmaLinux 9.6 home server with Proxmox; use VirtualBox on Windows for small labs; use free GCP credits for cloud labs.

#### Week 0 Preparation

- **Install tools**: Git, Docker, Podman, kubectl, kind, Helm, Ansible, Terraform, Vault CLI, Proxmox access.
- **Lab**: set up Proxmox on AlmaLinux or install Proxmox VE on a spare disk; create one LXC container.

#### Week 1 Proxmox LXC fundamentals

- **Goals**: understand LXC vs VM, create templates, backup/restore, networking.
- **Labs**: create an AlmaLinux LXC template; deploy a Django container; snapshot and restore.
- **Deliverable**: a documented LXC template and an Ansible playbook to provision it.

#### Week 2 Linux server ops and networking

- **Goals**: systemd, users, SSH hardening, firewall (firewalld/ufw), basic networking, DNS.
- **Labs**: configure SSH keys, set up a bastion host, configure VLANs in Proxmox.
- **Deliverable**: security checklist and hardened LXC image.

#### Week 3 Docker, containers, and images

- **Goals**: build Dockerfiles for Django and Next.js, multi‑stage builds, image scanning.
- **Labs**: build and push images to GitHub Container Registry.
- **Deliverable**: CI pipeline that builds and scans images.

#### Week 4 Kubernetes basics and local cluster

- **Goals**: pods, services, deployments, ConfigMaps, Secrets, Ingress.
- **Labs**: run kind or minikube, deploy the sample app, expose via ingress.
- **Deliverable**: Helm chart for the sample app.

#### Week 5 Kubernetes cluster ops and GitOps

- **Goals**: cluster architecture, namespaces, RBAC, ArgoCD, HelmOps.
- **Labs**: install ArgoCD, create an app that syncs from GitHub.
- **Deliverable**: GitOps flow demo (PR → merge → ArgoCD deploy).

#### Week 6 Cloudflare and edge protection

- **Goals**: DNS, SSL, WAF rules, caching, rate limiting, Workers basics.
- **Labs**: configure Cloudflare for a test domain, set page rules and firewall rules.
- **Deliverable**: Cloudflare policy document and test results for DDoS simulation.

#### Week 7 Secrets, Vault, and observability

- **Goals**: HashiCorp Vault basics, OpenTelemetry, Prometheus, Grafana, Sentry.
- **Labs**: store a secret in Vault, inject into a pod; instrument Django with OpenTelemetry and view traces.
- **Deliverable**: Vault policy and a trace showing a request across services.

#### Week 8 CI/CD, Proxmox→K8s migration, and security

- **Goals**: GitHub Actions, Terraform basics, Ansible for Proxmox, migration checklist.
- **Labs**: create a pipeline that provisions infra (local), builds images, and deploys via ArgoCD.
- **Deliverable**: End‑to‑end demo and a personal study portfolio on GitHub.

#### Optional Weeks 9–12 Advanced topics

- LLM basics and inference deployment (vLLM/Triton), GPU basics, monitoring LLM latency.
- Multi‑region clusters, data residency enforcement, and compliance documentation.

---

### 5. Assessment, deliverables, and GitHub usage

- **GitHub roles**: use GitHub for source control, Actions for CI, Pages for docs, and a separate repo for IaC (Terraform/Ansible).
- **Assessments**: weekly quizzes, lab PRs, and a final capstone: deploy a parish site to Proxmox and to K8s via GitOps.
- **Grading**: pass/fail for labs; code review required for all PRs.

---

### 6. Questions I need from you before I generate the final, exportable materials

Answer these so I can produce the exact prompt, syllabus files, and starter GitHub repo scaffold.

1. **Target learner level** — absolute beginner or intermediate (some Linux)?
2. **Time commitment** — do you prefer an 8‑week intensive or 12‑week paced plan?
3. **Hands‑on hardware** — will you use the AlmaLinux machine as Proxmox host or a separate dedicated machine?
4. **Cloud account** — do you have GCP credits or should I design labs using free tiers?
5. **Preferred GPU option for LLM labs** — L40S (cheaper) or H100 (enterprise)?
6. **Language** — English only or bilingual (Lithuanian + English) materials?
7. **Deliverable format** — single AI prompt, multi‑stage prompt series, week‑by‑week syllabus, and GitHub repo scaffold — which combination do you want first?
8. **Hiring constraints** — budget for contractors per month and preferred contract type (fixed price pilot vs time & materials)?
9. **Compliance signoff** — do you have legal counsel to validate the Lithuanian residency mapping or should I include a template DPIA for review?
10. **Branding** — do you want alternate project names or keep “Journey Of Life / JOL”?

---

### Final recommendation and next step

If you want immediate, practical output I will:

- Produce the **final optimized hiring prompt** as a downloadable text and a **GitHub repo scaffold** with starter manifests and CI templates.
- Produce the **8‑week learning syllabus** as Markdown files and a checklist you can follow.

Tell me which deliverable to generate first and answer the 10 questions above so I can tailor the materials precisely to your environment and constraints.