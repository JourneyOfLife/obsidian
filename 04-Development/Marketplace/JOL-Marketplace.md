
Understood. As a seasoned architect and a "paranoid compliance officer who also codes" with 30 years of experience, I approach this task with an unwavering focus on security, regulatory adherence, and precise technical communication. My goal is to transform your request into a highly actionable and comprehensive prompt that will lay the definitive groundwork for the "Journey Of Life" (JOL) marketplace's system design.
Before I create the optimized prompt, I have a couple of clarifying questions to ensure the output perfectly aligns with your expectations:
1. Audience of the new PROMPT: Is the optimized prompt I'm creating intended to be given to another AI model (e.g., a large language model) to generate the system design, or is it a detailed specification for a human development team?
2. Output of the new PROMPT: When this new, optimized prompt is used, should its output be:
    ◦ A High-Level Overview: (e.g., summarizing components and interactions with diagrams like those in the README.mds)?
    ◦ Detailed System Design: (e.g., specifying individual software modules, API endpoints, data flow patterns, communication protocols, and specific database schemas, as described in your initial query)? The sources already contain a significant amount of high-level architectural information, so understanding if a deeper dive is expected from the prompt's output is crucial.

--------------------------------------------------------------------------------
Assuming you are seeking a detailed specification for a human development team (or a highly advanced AI), and that the prompt should request a comprehensive, detailed system design as its output, here is the optimized, comprehensive, structured, and actionable prompt for the "Journey Of Life" (JOL) AI-powered B2B/B2C marketplace:

--------------------------------------------------------------------------------
Optimized System Design Prompt: "Journey Of Life" (JOL) Marketplace
Role: You are the Principal Solutions Architect and Lead Backend Engineer for the "Journey Of Life" (JOL) project. You possess 30 years of experience in designing and building large-scale, enterprise-grade, highly compliant, multi-tenant B2B/B2C marketplaces. Your mandate is "Trust & Compliance First".
Project: "Journey Of Life" (JOL) - A pan-European, AI-powered B2B/B2C marketplace dedicated to the end-of-life industry (church equipment, funeral services/supplies, cemetery maintenance).
Objective: To define the specific software modules, detailed data flow patterns, and precise communication protocols that will govern the functionality of the JOL marketplace MVP. This blueprint must serve as the technical implementation groundwork, ensuring scalability, security, maintainability, cost-effectiveness, and unwavering regulatory compliance (especially GDPR and PCI-DSS) from day one.

--------------------------------------------------------------------------------
1. Project Vision & MVP Scope
• Vision: To become the trusted digital commons and primary digital infrastructure for Europe's end-of-life industry, fostering connection, dignity, and accessibility while preserving cultural and religious traditions through technology.
• Mission (MVP Focus): Seamlessly connect suppliers and buyers of physical goods across Church Equipment, Funeral Home Supplies, and Cemetery Maintenance in the Baltic states (Lithuania, Latvia, Estonia). The platform must demonstrably handle multilingual content, secure transactions, and full GDPR compliance, laying a solid foundation for future EU-wide expansion.
• Target Users: B2B (Churches, Funeral Homes, Cemetery Maintenance Companies) and B2C (small artisans, individuals).
• Monetization (MVP):
    ◦ Listing Fee: Freemium model: first 10 listings per user/month are free; subsequent listings cost €1 each.
    ◦ Transaction Fee: A 1% commission (minimum €1) on all successful sales processed through the platform.
2. Core Technical Architecture & Mandates
The system must be designed as an API-First, service-oriented architecture, potentially moving towards microservices, leveraging a hybrid cloud strategy (70% self-hosted / 30% Google GKE) for optimal cost-performance and security.
• Backend Framework: Django (Python) with Django REST Framework (DRF) for a robust, headless API architecture.
• Database: PostgreSQL with PostGIS for geospatial queries and pgvector for AI vector embeddings. All user PII must be encrypted at rest using pgcrypto.
• Frontend: Next.js / React for a dynamic, interactive, and SEO-friendly user interface, including RTL (Right-to-Left) support for Arabic.
• Caching & Asynchronous Tasks: Redis for session storage and caching, and Celery with Redis/RabbitMQ as a message broker for background jobs (e.g., email notifications, image processing, calling AI APIs).
• Search Engine: Elasticsearch or OpenSearch for blazing-fast, faceted, and AI-enhanced product search, separate from the primary database.
• Containerization & Orchestration: All application components must be Dockerized and orchestrated using Kubernetes (GKE for cloud, potentially self-hosted for private servers) for automated deployment, scaling, and management.
• Web Server / Proxy: Nginx as the web server, ingress controller, and load balancer, handling public-facing traffic and terminating HTTPS.
• API Gateway: Consider Kong or Tanzu Spring Cloud Gateway for advanced routing, rate limiting, and security.
3. AI Integration Strategy (MVP Priorities)
AI is a core utility, not a gimmick. The architecture must support a multi-model LLM strategy for cost-effectiveness and performance.
4. Multilingual AI Chatbot: The #1 MVP AI feature for trust-building. Must be context-aware, trained on JOL FAQs, policies, and product categories, handling basic support and guiding users in 30 languages.
5. Semantic Search: Critical for product discoverability, using vector embeddings (via pgvector) to allow natural language queries.
6. Real-time User-Generated Content (UGC) Translation: Foundational for breaking language barriers. Implement real-time, context-aware AI translation for product listings, descriptions, and potentially reviews across 30 languages (27 EU + UA, RU, AR), including RTL support for Arabic. Leverage DeepL API for high-quality EU languages for MVP, with a plan for self-hosted Llama 3 for cost-optimization post-MVP.
    ◦ Static Content Translation: Django's native i18n framework for static content.
7. Compliance & Security Architecture (Non-Negotiable)
The architecture must embed "Paranoid Compliance".
• GDPR (General Data Protection Regulation): GDPR-compliant by design and by default.
    ◦ Data Protection Officer (DPO): Appointment is a mandatory first step.
    ◦ Data Encryption: All PII encrypted at rest (pgcrypto).
    ◦ User Rights: Automated workflows for "Right to Erasure" and data access.
    ◦ Consent Management: Middleware to ensure valid legal basis (consent, contract) for data processing.
    ◦ Audit Logging: Seamless integration with an auditing system for access and processing events.
    ◦ Geolocation-Based Compliance: Tailor data handling based on user's inferred EU member state.
• PCI-DSS (Payment Card Industry Data Security Standard): Achieve PCI-DSS Level 1 compliance by proxy through Stripe integration. Raw credit card numbers must never be handled or stored on JOL servers.
• Network Security: Forced HTTPS (TLS 1.3) everywhere. Secure VPN tunnel (e.g., WireGuard) between private servers and GKE. Database servers must have no public internet ingress.
• Secrets Management: Hashicorp Vault or pass for managing API keys, encryption keys – never in code.
• Authentication: Mandatory 2FA (Time-based OTP) for all user and admin accounts. User model extensible for future OAuth (Google, Facebook) and national e-IDs.
• Security Headers: Middleware to set privacy-focused headers (e.g., X-Content-Type-Options).
8. External Integrations & Data Flow
• Payments:
    ◦ Stripe Connect: Primary payment processor, leveraging Stripe Tax API for real-time, automated VAT calculation and remittance for all 27 EU countries, including SEPA Instant payments.
    ◦ PayPal Commerce Platform: Secondary payment option.
• Logistics (MVP): Integrate DPD API for shipping cost calculation, label generation, tracking numbers, and webhook integration for order status updates.
• CRM Integration: Implement a bi-directional sync with Bitrix24 CRM.
    ◦ User hierarchies (e.g., Diocese Admin managing multiple churches) created in Bitrix24 should reflect in JOL's Role-Based Access Control (RBAC).
    ◦ Key order/registration events should be pushed back to Bitrix24.
9. Deliverables: Comprehensive System Design
As the Principal Solutions Architect, you must produce the following detailed system design documents and artifacts:
10. Modular Architecture Breakdown:
    ◦ Define all specific software modules (e.g., Django apps like users_app, products_app, orders_app, search_app, bitrix24_integration_app, ai_service_app).
    ◦ Clearly delineate the responsibilities of each module and its boundaries.
    ◦ Justify how this modularity supports scalability, maintainability, and security.
11. Detailed Data Flow Patterns:
    ◦ Create sequence diagrams for all critical user journeys and background processes, including:
        ▪ User Registration & 2FA Setup.
        ▪ Product Listing (including AI translation and search indexing).
        ▪ Shopping Cart Management.
        ▪ VAT-Compliant Checkout & Payment (Stripe Tax workflow).
        ▪ Order Fulfilment (DPD integration).
        ▪ Bitrix24 User/Hierarchy Sync.
        ▪ AI Chatbot Interaction.
        ▪ User Data Erasure workflow.
    ◦ Illustrate data pathways between frontend, backend modules, database, caching, search engine, AI services, and external APIs.
12. Precise Communication Protocols:
    ◦ Specify communication mechanisms between modules (e.g., RESTful APIs via DRF, asynchronous messaging via Celery/Redis, secure VPN tunnels).
    ◦ Define API endpoints, request/response structures (JSON), and authentication/authorization mechanisms (e.g., JWT, OAuth2) for inter-service communication.
    ◦ Detail webhook integration specifications for payment and logistics providers.
13. Core Database Schema Designs:
    ◦ Provide Entity-Relationship Diagrams (ERDs) and detailed schema definitions for key models (e.g., User, UserProfile with RBAC fields, ProductListing, Order, OrderItem, Transaction, Translation tables).
    ◦ Highlight how GDPR compliance (e.g., PII encryption, anonymization fields, lawful basis tracking) is embedded in the schema.
14. Infrastructure Deployment Plan:
    ◦ Detail the deployment strategy for the hybrid cloud model, specifying which services reside on GKE (Kubernetes manifests, auto-scaling configurations) and which on private Ubuntu LTS servers (hardware specs, OS hardening, backup strategy using BorgBackup, VPN setup).
    ◦ Outline monitoring and logging solutions (e.g., Prometheus, Grafana, ELK stack).
15. Security & Compliance Protocols:
    ◦ Provide a concrete GDPR Compliance Checklist integrated directly with the design, detailing how each principle (data minimization, storage limitation, accuracy, accountability) is met.
    ◦ Specify security measures at each layer (e.g., WAF, DDoS protection, TLS 1.3 enforcement, secrets management with Hashicorp Vault, mandatory 2FA, secure coding practices).
This comprehensive blueprint will serve as the definitive guide for JOL's technical implementation, built on a foundation of unwavering trust, technical excellence, and paranoid compliance.
