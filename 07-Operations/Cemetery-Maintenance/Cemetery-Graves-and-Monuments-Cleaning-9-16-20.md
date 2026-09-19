

# Optimized Prompt (for architects, product, dev & AI teams)

https://chatgpt.com/c/6f912fd7-f7ba-42e0-a96d-7817aa2e2ad6

**Title:** Product Brief — “InterVitea / [Brand]” — Pan-European Cemetery Graves & Monuments Cleaning Platform (AI-enabled, Catholic-tone)

**Short description (1–2 lines):**  
Design and build a GDPR-compliant, multilingual web + mobile platform to connect families, churches, funeral services, municipalities and professional cleaners across Europe for graves & monuments cleaning, maintenance and memorial services. Leverage AI for image analysis, scheduling, quoting, and compassionate chat assistance with an optional Catholic tone for church users.

**Goals / Success metrics**

- Launch MVP in X EU markets (configurable) with 3 language clusters.
    
- 10,000 registered users and 500 verified service partners in 12 months.
    
- Average booking conversion ≥ 5% from visits; NPS ≥ 40.
    
- Automated AI quotes accuracy within ±15% of human estimates; image classification accuracy ≥ 85%.
    

**Target audiences**

- Families / Individuals
    
- Church administrators (parish priests, sacristans)
    
- Funeral homes & agencies
    
- Cleaning technicians & companies
    
- Municipal cemetery managers
    
- E-commerce sellers (flowers, monuments)
    
- Volunteers & charities
    

**Core value propositions**

- Fast, dignified, and compliant cemetery cleaning bookings across Europe.
    
- AI-assisted visual assessments and quote estimation from uploaded photos.
    
- Church features: community coordination, volunteer events, parish fundraising.
    
- Marketplace for memorial products and subscription maintenance plans.
    

**Scope & packages (quick)**

- Economy (9–12 pages) — basic brochure + booking flow — €999
    
- Normal (15–20 pages) — multi-role dashboards, multilingual, improved UX — €1999
    
- VIP (20+ pages + e-commerce) — full marketplace, subscriptions, advanced AI & integrations — €2999
    

**Must-haves (MVP)**

- Multilingual UI, GDPR cookie & consent, data minimization.
    
- Roles & RBAC: family, church admin, funeral agency, provider, platform admin.
    
- Image upload + AI image analysis pipeline for grave condition scoring.
    
- Secure payments (Stripe/Adyen + local PSPs).
    
- Scheduling & route optimization for technicians.
    
- “My Church” and “Funeral Services” links/widgets; Marketplace link in header.
    

**Non-functional requirements**

- High availability (SLA target 99.9%), EU data residency (option), encryption at rest & in transit, accessible (WCAG 2.1 AA), logging & audit trails, automated backups.
    

**Integrations**

- Maps/GPS (Google/Here/OpenStreetMap), Payment Gateways (Stripe/Adyen/local), SMS/Email (Twilio/SendGrid), CRM (Bitrix24), Identity (Auth0 / Keycloak), optional blockchain for donation traceability.
    

**AI/Modeling brief**

- Image classifier (mobile/resolution tolerant) → damage/risk/cleaning complexity tags.
    
- Quote estimator — regression model using image score + dimensions + material type.
    
- Conversational assistant (ChatGPT fine-tuned persona with Catholic tone for parish pages).
    
- Scheduling optimizer (vehicle routing problem solver).
    

**Regulatory & Compliance**

- Per-country legal pages (privacy, cookie, T&Cs), DPA with partners, data retention policies, opt-in marketing, mechanisms to anonymize data for analytics.
    

**Deliverables**

- Detailed 20-page sitemap + content brief per page (this doc).
    
- Functional & technical requirements (epics & user stories).
    
- UI kit & responsive prototypes (desktop + tablet + mobile).
    
- Security & compliance checklist.
    
- Roadmap for phased rollout (MVP → v1 → Marketplace).
    

---

# 20-Page Website Structure (page name → purpose → core content/features → target user / SEO hint)

> Below pages are grouped by **Baseline (Economy)**, **Extended (Normal)**, **Full (VIP)**. Marked pages with * are recommended for VIP / Marketplace.

---

## Baseline (Economy: 9–12 pages) — pages included at minimum

1. **Home**
    
    - Purpose: Hero value proposition, primary CTAs (Get Quote / Request Service), overview of AI assessment, trust signals (churches, partners).
        
    - Features: Quick photo upload CTA, languages selector, top locales, "My Church" & "Funeral Services" links in header.
        
    - Target: All visitors. SEO: “grave cleaning services”, “monument cleaning [country]”.
        
2. **How It Works**
    
    - Purpose: Step-by-step flow (Upload photo → AI assessment → Quote → Schedule → Service).
        
    - Features: Diagrams, timeline, pricing tiers summary, FAQ snippet.
        
    - Target: Families and churches. SEO: “how to clean grave”, “grave cleaning process”.
        
3. **Services (Landing)**
    
    - Purpose: Directory of offered services (one-time cleaning, restoration assessment, subscription care).
        
    - Features: Filter by service type, link to instant quote, featured providers.
        
    - Target: All. SEO: “grave maintenance services”.
        
4. **Instant Quote / Upload**
    
    - Purpose: Primary conversion page where users upload photos, enter cemetery location, and get an AI estimate.
        
    - Features: Image uploader, AI result, manual verification option, add to cart / schedule.
        
    - Target: Families, funeral homes. Integration: Image analysis, maps, payments.
        
5. **Book & Schedule**
    
    - Purpose: Booking flow with calendar, technician assignment, address confirmation.
        
    - Features: Time slots, provider profiles, payment, cancellation policy.
        
    - Target: Bookers. Integration: Scheduling engine, payment gateway.
        
6. **About / Mission**
    
    - Purpose: Company story, Catholic / community values, social impact (volunteer, eco choices).
        
    - Features: Team, partners, press, charity ties.
        
    - Target: Churches, donors. SEO: “church cemetery care”.
        
7. **Contact & Support**
    
    - Purpose: Contact form, phone, live chat, emergency service line.
        
    - Features: Local office finder, support hours, Church & Funeral Services contact shortcuts.
        
    - Target: All.
        
8. **Pricing & Packages**
    
    - Purpose: Clear presentation of Economy / Normal / VIP website packages (for your web offering) and the platform’s service pricing tiers.
        
    - Features: Comparison grid, benefits per tier, CTA to get custom quote.
        
    - Target: Buyers and resellers.
        
9. **Legal (Privacy / Cookies / Terms)**
    
    - Purpose: GDPR privacy policy, cookie management, platform terms, provider DPA summary.
        
    - Features: Country selector for localized legal text.
        
    - Target: All. Compliance focal point.
        

---

## Extended (Normal: adds pages 10–15; total 15–20 pages)

10. **Provider / Partner Portal (Overview)**
    
    - Purpose: Landing for cleaners and funeral partners to apply and manage account.
        
    - Features: Onboarding checklist, verification flows, pricing for partners.
        
    - Target: Service providers.
        
11. **Provider Dashboard (sample page)**
    
    - Purpose: Demo page showing schedule, jobs, earnings, messages.
        
    - Features: Job accept/decline, route map, invoice export.
        
    - Target: Providers.
        
12. **Church Portal**
    
    - Purpose: Church admin features: volunteer events, parish cemetery roster, donation management.
        
    - Features: “My Church” page (see #18), mass/grave blessing scheduling, Catholic-tone templates.
        
    - Target: Church administrators.
        
13. **Funeral Services Directory**
    
    - Purpose: Directory & partnership hub for funeral agencies with direct contact CTA.
        
    - Features: Sponsored listings option, integration with funeral services link.
        
    - Target: Funeral homes.
        
14. **Volunteer & Community Projects**
    
    - Purpose: Organize community cleanups, register volunteers, sign waivers.
        
    - Features: Event builder, check-in, certification tracking.
        
    - Target: Churches, NGOs, volunteers.
        
15. **Testimonials & Case Studies**
    
    - Purpose: Social proof and sector case studies (parish projects, municipal contracts).
        
    - Features: Video testimonials, before/after galleries (image rights management).
        
    - Target: B2B & B2C.
        

---

## Full (VIP: 20+ pages + e-commerce + marketplace)

16. **Marketplace (Landing)***
    
    - Purpose: Central marketplace for flowers, candles, cleaning products, memorial items and services.
        
    - Features: Categories, seller storefronts, promotions, search & filters.
        
    - Target: Consumers & churches. Integration: E-commerce engine.
        
17. **Product Category / Product Page***
    
    - Purpose: Detail pages for flowers, monuments, care kits; upsell subscription plans.
        
    - Features: Variants, shipping, personalization, reviews, related items.
        
    - Target: E-commerce customers.
        
18. **My Church (Parish Hub)**
    
    - Purpose: Public parish page with cemetery care offers, donation widget, volunteer signups, memorial wall.
        
    - Features: Catholic-tone content templates, Mass & blessing scheduling, parish announcements.
        
    - Target: Parishioners, visitors. SEO: “parish cemetery care [parish name]”.
        
19. **Memorial Pages / Digital Memorials***
    
    - Purpose: Create evergreen memorial pages with photos, prayers, donations, & virtual candles.
        
    - Features: Privacy options, share links, grief resources (Catholic resources), legacy subscription.
        
    - Target: Families, churches.
        
20. **Admin / Operations Dashboard (Platform)**
    
    - Purpose: For platform admins: partner approvals, KYC, audits, financial reports, GDPR toolbox.
        
    - Features: Role-based access, automated DPA signatures, incident & escalation workflows.
        
    - Target: Internal ops team.
        

> **Optional VIP pages** (if >20): Subscription Plans, Insurance & Claims, Enterprise / Municipal Contracts, Newsroom / Blog, Careers.

---

# Which pages belong to each website package (quick mapping)

- **Economy (€999)** — pick any **9–12** core pages: 1 (Home), 2 (How it Works), 3 (Services), 4 (Instant Quote), 5 (Book & Schedule), 6 (About), 7 (Contact), 8 (Pricing), 9 (Legal). _(Good for small operators or local parish initiatives.)_
    
- **Normal (€1999)** — includes Economy pages + pages 10–15 (Provider Portal, Provider Dashboard, Church Portal, Funeral Directory, Volunteer Projects, Testimonials). _(Best for multi-region rollouts, with provider management.)_
    
- **VIP (€2999)** — full 20 pages above including Marketplace & E-commerce pages (16–19) + Admin dashboard (20). Also includes subscription plan setup, advanced AI features, multilingual launch pack and integration with PSPs and CRM. _(Enterprise & pan-EU rollout, includes marketplace monetization.)_
    

---

# UX/UI, Accessibility & Compliance Notes (as a paranoid compliance officer + UX architect)

**Tone & Content:**

- Provide a configurable “Tone” layer—neutral/compassionate or Catholic-aligned phrasing. Church pages default to Catholic tone; family-facing pages default compassionate but secular-friendly copy.
    

**Accessibility & Inclusion:**

- WCAG 2.1 AA: keyboard navigation, color contrast, scalable type, alt text for all images.
    
- Keyboard + screen-reader tested booking flows; avoid CAPTCHAs that block accessibility—use invisible risk-scoring + challenge fallback.
    

**GDPR & Legal:**

- Per-country localized privacy & cookie banner with granular opt-ins; records of consent stored with timestamps and IPs.
    
- DPA templates for providers; role-based data minimization (providers should not see unnecessary PII).
    
- Data residency toggle for EU sovereign cloud; retention policy (e.g., 7 years for financial, configurable shorter for images unless consented).
    

**Security:**

- TLS everywhere, HSTS, CSP, OWASP Top 10 mitigations.
    
- Encryption at rest; key management in KMS.
    
- Rate limiting, WAF, SIEM + incident playbooks.
    
- PCI DSS scope reduction via PSP (hosted checkout) recommended for MVP.
    

**Privacy & Image Handling:**

- Explicit consent checkbox when users upload grave images; image retention policy and option to auto-anonymize before model training.
    
- Clear rights management for before/after images used in marketing; include opt-out.
    

**Localization & Languages:**

- Launch with 3 clusters (e.g., English + Lithuanian/Polish + German/Spanish), then scale. Use human-validated translations for legal & church texts; machine translation acceptable for supporting content with “suggested translation” flag.
    

**AI Ethics & Guardrails:**

- Clearly label AI outputs (“Estimated by a machine; final human review recommended”).
    
- Provide appeals / manual review channel for quotes and image analyses.
    
- No religious proselytizing by AI—Catholic tone limited to church pages and optional templates only.
    

---

# Tech stack & architecture (high-level)

**Frontend:** React + Next.js (SSR for SEO), React Native for apps.  
**Backend:** FastAPI or Django (modular microservices); GraphQL or REST.  
**DB:** PostgreSQL (primary), Redis (caching, task queues), Mongo/Elasticsearch (search & logs).  
**AI pipeline:** Separate microservice: image preprocessing → model inference (ONNX/TorchServe) → result store. Use worker queue (Celery/RQ) for heavy tasks.  
**Cloud:** AWS/GCP/Azure with EU data region options. Use Kubernetes for orchestration.  
**Payments:** Stripe + Adyen + local PSPs. PCI offloaded.  
**Monitoring:** Prometheus, Grafana, Sentry.  
**CI/CD:** GitHub Actions / GitLab CI with branch protections; infra as code (Terraform).

---

# Pricing tiers (refined & rationale)

**1. Economy — €999**

- 9–12 pages (basic set above)
    
- Single language, single country, standard contact & booking flow
    
- Basic AI: photo upload → manual review (no automated pricing)
    
- Payment integration: hosted checkout only
    
- 1 admin user, 1 provider account
    
- 30 days support, optional maintenance retainer
    

**Why:** Low barrier for local parishes / small operators.

**2. Normal — €1999**

- 15–20 pages (adds Provider Portal, Church Portal, Volunteer pages)
    
- Multi-language (3), multi-region support
    
- AI: basic automated image analysis + suggested quotes
    
- Provider Dashboard + scheduling, SMS notifications, basic analytics
    
- 3 admin seats, onboarding for 5 providers
    
- 3 months support + SLA options
    

**Why:** For regional operators needing scaling & automation.

**3. VIP — €2999**

- 20+ pages + full E-commerce Marketplace
    
- Full AI suite (image classifier + automated quotes + conversational assistant with Catholic tone)
    
- Subscriptions, recurring maintenance plans, enterprise integrations (CRM, municipal contracts)
    
- Multi-currency, multi-PSP support, advanced analytics, automated invoicing, partner onboarding flow
    
- Dedicated onboarding + 6 months support, EU data residency option, priority SLA
    
- Optional add-ons: advanced custom integrations, ISO checklist validation, branded mobile apps.
    

**Why:** Built for pan-EU or enterprise customers where monetization & scale are priorities.

---

# Implementation & rollout recommendation (phased)

1. **Phase 0 — Discovery (2–4 weeks)**: finalize scope, legal requirements per country, language list, provider verification policy.
    
2. **Phase 1 — MVP (8–12 weeks)**: build core 9 pages, Instant Quote (manual human review), booking & payments, basic admin. Launch in 1–2 pilot countries.
    
3. **Phase 2 — Scale & AI (12 weeks)**: deploy AI inference, provider portal, church portal, multilingual expansion.
    
4. **Phase 3 — Marketplace (12+ weeks)**: add e-commerce, subscriptions, advanced analytics, and enterprise contracts.
    
5. **Ongoing:** continuous model retraining (with consent), SEO / content strategy, partnerships with dioceses & funeral associations.
    

---

# Final deliverable checklist I will provide now

-  Polished, professional prompt (above).
    
-  Complete 20-page sitemap with page details and which tier includes which pages.
    
-  Refined pricing tiers and rationale.
    
-  UX / compliance / architecture guidance and rollout plan.
    

---

If you want, I can now:

- Convert this sitemap into a content brief (one paragraph + H1/H2/H3 + CTA + metadata + wireframe notes) for **each** of the 20 pages (ready for copywriters and devs), OR
    
- Produce a prioritized development backlog (epics & user stories) for the MVP launch.
    

Tell me which of those two you'd like me to produce next and I’ll create it immediately.

---

Convert this sitemap into a content brief (one paragraph + H1/H2/H3 + CTA + metadata + wireframe notes) for **each** of the 20 pages (ready for copywriters and devs)


---

Excellent — below is the **full Content Brief for all 20 pages** of the _Cemetery Graves & Monuments Cleaning Services Platform_, structured and ready for **copywriters, designers, and developers**.

Each page contains:  
✅ One-paragraph purpose  
✅ H1 / H2 / H3 structure  
✅ CTA (Call To Action)  
✅ Metadata (title + description)  
✅ Wireframe notes (for UX/UI & developers)  
✅ Placement of **My Church / Funeral Services / Marketplace** when relevant

---

## 1. HOME

**Purpose (1 paragraph)**  
The Home page introduces the platform’s mission: dignified, professional grave and monument cleaning through technology and compassion. It serves as the main entry point for families, churches, and funeral service providers. It highlights trust, speed, AI-based assessment, and European coverage.

**H1:**  
Revolutionizing Grave & Monument Care Across Europe

**H2:**  
Respectful. Professional. Powered by Technology.

**H3:**  
Upload a photo. Receive a quote. Restore dignity.

**CTA:**  
➡️ Get Instant Quote  
➡️ Request Cleaning Service  
➡️ Visit My Church  
➡️ Funeral Services

**Metadata:**  
**Title:** Professional Cemetery & Monument Cleaning Services in Europe  
**Description:** Book grave and monument cleaning across Europe. AI-powered quotes, trusted partners, church & funeral service integrations.

**Wireframe notes:**

- Hero banner image (cemetery + sunlight + clean monument)
    
- 3 big CTA buttons
    
- “How it works” in 3 steps
    
- Trust badges (Churches / Partners / Municipalities)
    
- Footer links: My Church | Funeral Services | Marketplace
    

---

## 2. HOW IT WORKS

**Purpose**  
Explains the platform process step-by-step in a simple visual way for families, churches, and organizations.

**H1:**  
How Our Service Works

**H2:**  
From photo to professional cleaning in 4 steps

**H3:**  
Technology + Care + Precision

**Steps:**

1. Upload grave photo
    
2. AI analyzes & suggests service
    
3. Choose schedule
    
4. Professional cleaning is completed
    

**CTA:**  
➡️ Upload Photo Now

**Metadata:**  
**Title:** How Cemetery Cleaning Works – Step by Step  
**Description:** Simple 4-step process to book professional grave & monument cleaning using AI.

**Wireframe notes:**

- Infographic (4 steps)
    
- Icons + explanation
    
- FAQ snippet block
    
- CTA bar fixed at bottom
    

---

## 3. SERVICES (LANDING)

**Purpose**  
Displays all services with details and links to request.

**H1:**  
Our Professional Cemetery & Monument Services

**H2:**  
More than cleaning — complete care

**H3:**  
Traditional, Restoration & Ongoing Care

**Services list:**

- Basic cleaning
    
- Deep restoration
    
- Moss & algae removal
    
- Gold lettering renewal
    
- Photo engraving cleaning
    
- Marble / granite protection
    
- Seasonal care plans
    

**CTA:**  
➡️ Choose Your Service  
➡️ Book Cleaning Now

**Metadata:**  
**Title:** Our Cemetery & Monument Cleaning Services  
**Description:** Explore professional cleaning, restoration & maintenance services for graves and monuments.

**Wireframe notes:**

- Service cards with icons
    
- Filter by type
    
- “Add to quote” button
    
- Price range display
    

---

## 4. INSTANT QUOTE / UPLOAD

**Purpose**  
Primary conversion page for AI-powered price estimates.

**H1:**  
Get Your Instant Cleaning Quote

**H2:**  
Upload a photo of the grave or monument

**H3:**  
AI analyzes condition & size

**Fields:**

- Photo upload
    
- Cemetery location
    
- Material type
    
- Damage/condition
    

**CTA:**  
➡️ Get AI Quote  
➡️ Talk to Expert

**Metadata:**  
**Title:** Get Instant Quote for Grave Cleaning  
**Description:** Upload a photo and receive an instant AI-powered estimate for monument cleaning.

**Wireframe notes:**

- Large upload box center
    
- Progress bar & AI analysis status
    
- Result screen: price + service options
    
- Continue to Scheduling
    

---

## 5. BOOK & SCHEDULE

**Purpose**  
Appointment selection and payment.

**H1:**  
Schedule Your Cleaning Service

**H2:**  
Choose your preferred date & time

**H3:**  
Certified professionals, guaranteed respect

**Features:**

- Calendar
    
- Technician availability
    
- Payment confirmation
    
- SMS / email confirmation
    

**CTA:**  
➡️ Confirm Booking

**Metadata:**  
**Title:** Schedule Cemetery Cleaning  
**Description:** Select date, time and confirm booking for your grave or monument care.

**Wireframe notes:**

- Timeline calendar
    
- Map location
    
- Technician rating card
    
- Payment form
    

---

## 6. ABOUT / MISSION

**Purpose**  
Explain vision, values, and faith-respecting approach.

**H1:**  
Our Mission: Dignity in Memory

**H2:**  
A respectful approach inspired by faith & community

**H3:**  
Technology in service of remembrance

**CTA:**  
➡️ Join Our Mission  
➡️ Partner With Us

**Metadata:**  
**Title:** About Us – Cemetery Care with Purpose  
**Description:** Learn about our mission to bring dignity, respect, and innovation to cemetery care.

**Wireframe notes:**

- Founder message
    
- Mission + values
    
- Photos of clean restorations
    

---

## 7. CONTACT & SUPPORT

**Purpose**  
Support page with contact forms.

**H1:**  
Contact & Support

**H2:**  
We are here to help you

**H3:**  
24/7 digital support available

**CTA:**  
➡️ Chat with Support  
➡️ Call Us

**Metadata:**  
**Title:** Contact Cemetery Care Support  
**Description:** Get help with bookings, quotes, services or partnerships.

**Wireframe notes:**

- Contact form
    
- Phone & WhatsApp
    
- Map + local branches
    

---

## 8. PRICING & PACKAGES

**Purpose**  
Show Economy/Normal/VIP website + service pricing.

**H1:**  
Our Packages & Pricing

**H2:**  
Transparent plans for all needs

**H3:**  
From simple care to full memorial solutions

**CTA:**  
➡️ Choose Plan  
➡️ Request Custom Quote

**Metadata:**  
**Title:** Pricing for Cemetery & Monument Care  
**Description:** Compare packages and choose the best plan for your cemetery cleaning needs.

**Wireframe notes:**

- 3 pricing cards
    
- Comparison table
    
- Advantages in checklist format
    

---

## 9. LEGAL (GDPR, TERMS, COOKIES)

**Purpose**  
Full compliance and trust.

**H1:**  
Privacy & Legal Compliance

**H2:**  
Your data & dignity matter

**H3:**  
GDPR compliant in all EU countries

**CTA:**  
➡️ Manage Cookie Settings

**Metadata:**  
**Title:** Privacy Policy & Terms  
**Description:** GDPR-compliant privacy and legal information for users and partners.

**Wireframe notes:**

- Tabs: Privacy / Cookies / Terms
    
- Download PDF option
    

---

## 10. PROVIDER / PARTNER PORTAL

**Purpose**  
For cleaners & businesses joining.

**H1:**  
Become a Service Partner

**H2:**  
Join Europe’s largest memorial care network

**H3:**  
Earn. Serve. Respect.

**CTA:**  
➡️ Apply as Partner

**Metadata:**  
**Title:** Partner with Cemetery Care Network  
**Description:** Apply to offer your grave cleaning services on our platform.

**Wireframe notes:**

- Requirements list
    
- Register button
    
- Benefits icons
    

---

## 11. PROVIDER DASHBOARD

**Purpose**  
Shows functionality for partners.

**H1:**  
Your Provider Dashboard

**H2:**  
Manage jobs & income in one place

**H3:**  
Smart route planning & scheduling

**CTA:**  
➡️ View Demo

**Metadata:**  
**Title:** Cleaning Partner Dashboard  
**Description:** Manage service requests, routes, invoices and communication.

**Wireframe notes:**

- Sidebar menu
    
- Job cards
    
- Earnings graph
    

---

## 12. CHURCH PORTAL

**Purpose**  
Dedicated to Church Admin.

**H1:**  
Church Portal

**H2:**  
For Parishes, Priests & Church Communities

**H3:**  
Care, prayer & service in one place

**CTA:**  
➡️ Register Your Church

**Metadata:**  
**Title:** Church Cemetery Care Portal  
**Description:** Digital tools for churches to manage cemetery care and community projects.

**Wireframe notes:**

- Parish dashboard preview
    
- Donation tracker
    
- Volunteer management
    

---

## 13. FUNERAL SERVICES DIRECTORY

**Purpose**  
Directory of funeral agencies.

**H1:**  
Funeral Services Network

**H2:**  
Partnered funeral agencies across Europe

**H3:**  
Trusted, verified, professional

**CTA:**  
➡️ Find Funeral Services  
➡️ Add Your Agency

**Metadata:**  
**Title:** Funeral Services Directory  
**Description:** Find or join verified funeral services connected to cemetery care.

**Wireframe notes:**

- Country filter
    
- Agency cards
    
- Rating + CTA
    

---

## 14. VOLUNTEER & COMMUNITY

**Purpose**  
Community engagement.

**H1:**  
Volunteer & Community Projects

**H2:**  
Serve in honor of those who came before

**H3:**  
Join cleaning & remembrance events

**CTA:**  
➡️ Join as Volunteer

**Metadata:**  
**Title:** Volunteer Cemetery Projects  
**Description:** Join community-driven cemetery restoration projects.

**Wireframe notes:**

- Event calendar
    
- Sign up form
    
- Map
    

---

## 15. TESTIMONIALS & CASE STUDIES

**Purpose**  
Show trust.

**H1:**  
Stories of Respect & Restoration

**H2:**  
Families, Churches & Communities Speak

**H3:**  
Before and After

**CTA:**  
➡️ View More Stories

**Metadata:**  
**Title:** Cemetery Cleaning Success Stories  
**Description:** Read testimonials and see before/after transformations.

**Wireframe notes:**

- Slider
    
- Video testimonial
    
- Case tiles
    

---

## 16. MARKETPLACE (Landing)

**Purpose**  
E-commerce hub.

**H1:**  
Memorial Marketplace

**H2:**  
Everything for remembrance in one place

**H3:**  
Flowers, candles, monuments & more

**CTA:**  
➡️ Visit Marketplace

**Metadata:**  
**Title:** Cemetery Memorial Marketplace  
**Description:** Shop flowers, candles, monuments and cleaning products online.

**Wireframe notes:**

- Categories grid
    
- Featured products
    
- Bestsellers
    

---

## 17. PRODUCT / CATEGORY PAGE

**H1:**  
Explore Memorial Products

**H2:**  
Choose with love & care

**H3:**  
Quality for remembrance

**CTA:**  
➡️ Add to Cart

**Metadata:**  
**Title:** Buy Memorial Products Online  
**Description:** Purchase cemetery and memorial products with delivery.

**Wireframe notes:**

- Product grid
    
- Filter + price slider
    
- Cart icon
    

---

## 18. MY CHURCH (Parish Hub)

**Purpose**  
Personal church page.

**H1:**  
My Church & Cemetery

**H2:**  
A sacred space of remembrance

**H3:**  
Care, prayer & community

**CTA:**  
➡️ Support this Parish  
➡️ Request Cemetery Service

**Metadata:**  
**Title:** My Church Cemetery Page  
**Description:** Dedicated parish page for care, community, and remembrance.

**Wireframe notes:**

- Church image
    
- Donation widget
    
- Cemetery map
    
- Event announcements
    

---

## 19. MEMORIAL PAGES

**Purpose**  
Digital remembrance.

**H1:**  
Digital Memorials

**H2:**  
Honor, remember, pray

**H3:**  
A place that never fades

**CTA:**  
➡️ Create Memorial

**Metadata:**  
**Title:** Create a Digital Memorial  
**Description:** Create an eternal online memorial for your loved one.

**Wireframe notes:**

- Memorial template
    
- Candle animation
    
- Prayer section
    

---

## 20. ADMIN / SYSTEM DASHBOARD

**Purpose**  
Operations control.

**H1:**  
Platform Administration

**H2:**  
Control, monitor & protect

**H3:**  
Security & governance

**CTA:**  
➡️ Admin Login

**Metadata:**  
**Title:** System Administration Dashboard  
**Description:** Control users, data, compliance, and partners.

**Wireframe notes:**

- KPIs
    
- User role panel
    
- Security alerts
    
- Audit trail
    

---

## ✅ NEXT STEP (You choose)

I can now generate (in order of maximum power for you):

1. **Full UX Wireframe Layout (Figma/Page Structure diagram in text)**
    
2. **Developer Technical Specification per page (frontend + backend)**
    
3. **Copywriting-ready content for all 20 pages**
    
4. **API Design + Database Schema**
    
5. **Investor-ready pitch deck content structure**
    

Just reply with:  
**“Proceed with option X”** (X = 1–5)

---

###### <font color="#ffff00">1. Full UX Wireframe Layout (Figma/Page Structure diagram in text</font>


Understood. I will now deliver a **Full UX Wireframe Layout in text form** for a **20-page professional website** for your:

> **Cemetery Graves & Monuments Cleaning Services Platform – AI-powered, Catholic-aligned, EU-wide**

This is written as a **Figma-style / page-structure diagram in text** that designers, developers, stakeholders, and compliance teams can directly follow.

It reflects:  
✅ 30+ years senior UX/Systems thinking  
✅ Catholic tone + respect + dignity  
✅ Multi-role platform (Families, Churches, Municipalities, Funeral Agencies)  
✅ AI + Future scalability  
✅ Your 3 critical links:

- **My Church**
    
- **Funeral Services**
    
- **Marketplace**
    

---

# ✅ GLOBAL STRUCTURE – ALL PAGES SHARE THIS SHELL

Every page follows this main layout:

```
-----------------------------------------------------
| TOP BAR (Language | Region | Login | Register)   |
-----------------------------------------------------
| LOGO | Menu | CTA BUTTON ("Request Service")      |
-----------------------------------------------------
| HERO SECTION – Image / Video + Main Message        |
-----------------------------------------------------
| PAGE-SPECIFIC SECTIONS                              |
-----------------------------------------------------
| TRUST STRIP (Partners, Churches, Cities, Logos)     |
-----------------------------------------------------
| FOOTER (Full Sitemap + Legal + Contacts + Social)   |
-----------------------------------------------------
```

**Main menu for ALL pages:**

- Home
    
- Services
    
- How It Works
    
- My Church
    
- Funeral Services
    
- Marketplace
    
- Pricing
    
- AI & Technology
    
- About Us
    
- Contact
    

---

# ✅ PAGE 1 — HOME (Main Landing Page)

```
[ HERO SECTION ]
- Background: Peaceful cemetery + church + sunrise
- H1: “Caring for Sacred Resting Places with Technology & Dignity”
- CTA 1: Request Cleaning
- CTA 2: For Churches

[ 3 MAIN CARDS ]
[] Individual & Family Services
[] Church & Parish Solutions
[] Funeral & Municipality Services

[ HOW IT WORKS – 4 Steps ]
1. Search grave / location
2. Choose service
3. AI assessment
4. Cleaning & report

[ AI SECTION ]
- Image: AI scanning a monument
- Text: AI-assisted care & tracking

[ LINKS HUB ]
- My Church
- Funeral Services
- Marketplace

[ TESTIMONIALS ]
[ MAP OF EUROPE – coverage areas ]
[ CTA – Start in your country ]
```

---

# ✅ PAGE 2 — ABOUT US

```
[ HERO ]
- H1: “A Mission of Respect, Restoration & Remembrance”

[ OUR STORY ]
- Why it was created (Covid + War victims)
- Respect for history & faith

[ VISION & MISSION ]
[ VALUES: Faith / Respect / Technology / Service ]
[ FOUNDERS / CORE TEAM ]
[ PARTNERS ]
[ CTA – Join our mission ]
```

---

# ✅ PAGE 3 — SERVICES (Overview)

```
[ HERO: Our Services ]

[ SERVICE GRID ]
[] Basic Cleaning
[] Deep Restoration
[] Monument Repairs
[] Seasonal Care
[] Winter Protection
[] Annual Plan
[] AI Inspection

[ FILTER: Individuals / Churches / Agencies ]

[ CTA – Get Quote ]
```

---

# ✅ PAGE 4 — GRAVE CLEANING SERVICE PAGE

```
[ HERO: Professional Grave Cleaning ]

[ BEFORE / AFTER SLIDER ]
[ WHAT’S INCLUDED ]
- Cleaning
- Moss removal
- Weed removal
- Polishing
- Cross restoration

[ AI DAMAGE SCAN ]
[ PACKAGE OPTIONS ]
[ CTA – Book Now ]
```

---

# ✅ PAGE 5 — MONUMENT RESTORATION

```
[ HERO: Monument & Stone Restoration ]

[ TYPES OF STONE ]
[ PROCESS DIAGRAM ]
[ SAFETY & COMPLIANCE ]
[ REQUEST FORM ]
```

---

# ✅ PAGE 6 — SUBSCRIPTION PLANS

```
[ HERO: Ongoing Care Plans ]

[] Bronze
[] Silver
[] Gold
[] VIP Church Plan

[ WHAT YOU CAN TRACK ]
✔ Maintenance history
✔ Photos
✔ AI reminders

[ CTA – Subscribe ]
```

---

# ✅ PAGE 7 — HOW IT WORKS

```
[ 6 STEP TIMELINE ]

1. Find grave via map
2. Upload photo
3. AI Review
4. Service schedule
5. Cleaning process
6. Report & archive

[ VIDEO EXPLAINER ]
[ CTA – Try Now ]
```

---

# ✅ PAGE 8 — MY CHURCH (Dedicated Portal)

```
[ HERO: Digital Tools for Churches & Parishes ]

[ BENEFITS ]
- Cemetery management
- Member database
- Donation tracking
- Event announcements

[ DASHBOARD MOCKUP ]
[ CTA – Register Your Church ]
```

---

# ✅ PAGE 9 — FUNERAL SERVICES (Agency Portal)

```
[ HERO: Partner with Funeral Agencies ]

[ FEATURES ]
- Direct booking
- Commission system
- Branded profiles
- API integration

[ CTA – Become Partner ]
```

---

# ✅ PAGE 10 — MARKETPLACE

```
[ HERO: Buy & Send Memorial Items ]

[ CARD GRID ]
[] Flowers
[] Candles
[] Crosses
[] Decorations
[] Cleaning kits
[] Monument engraving

[ FILTERS ]
[ ADD TO CART ]
[ CTA – Checkout ]
```

---

# ✅ PAGE 11 — AI & TECHNOLOGY

```
[ HERO: AI & Sacred Place Preservation ]

[ AI FEATURES ]
✔ Grave recognition
✔ Damage detection
✔ Predictive care
✔ Text generation
✔ Memorial pages

[ ETHICS STATEMENT – Catholic aligned ]
```

---

# ✅ PAGE 12 — FOR INDIVIDUALS & FAMILIES

```
[ USER PATH ]
- Step-by-step care services
- Remote ordering
- Memorial pages

[ CTA – Create memorial ]
```

---

# ✅ PAGE 13 — FOR CITIES & MUNICIPALITIES

```
[ MANAGEMENT DASHBOARD ]
- Cemetery management
- Area reports
- Bulk care
- Compliance

[ CTA – Request demo ]
```

---

# ✅ PAGE 14 — DIGITAL MEMORIALS

```
[ CREATE PAGE FOR DECEASED ]
- Photos
- Biography (AI-generated)
- Prayer wall
- Tribute messages
```

---

# ✅ PAGE 15 — PRICING

```
ECONOMY – €999
STANDARD – €1999
VIP – €2999 + E-COMMERCE

COMPARISON TABLE
[ CTA – Select Plan ]
```

---

# ✅ PAGE 16 — BLOG / EDUCATION

```
[ Articles ]
- Grave care
- Catholic burial
- Monument history
- AI in remembrance
```

---

# ✅ PAGE 17 — CERTIFICATIONS & COMPLIANCE

```
✔ GDPR
✔ Environmental rules
✔ Heritage protection
✔ Church-approved

[ DOWNLOAD CERTIFICATES ]
```

---

# ✅ PAGE 18 — PARTNERS

```
[ MAP + LOGOS ]
Churches
Cities
Funeral agencies
Florists

[ APPLY FORM ]
```

---

# ✅ PAGE 19 — SUPPORT & FAQ

```
[ CATEGORIES ]
- Orders
- Payments
- Services
- Churches
- AI
- Marketplace

[ LIVE CHAT – ChatGPT ]
```

---

# ✅ PAGE 20 — CONTACT

```
[ FORM + MAP + PHONE + WHATSAPP ]

- Region selection
- Service type
- Prayer request (optional)
```

---

# ✅ ADMIN / PRIVATE ZONE (Hidden UX)

```
- User dashboard
- Technician panel
- Church admin panel
- Marketplace seller panel
- Super admin / AI control
```

---

## ✅ NEXT STEP OPTIONS

Reply with one of the following and I will proceed immediately:

1. **"Convert all 20 pages into full content briefs"**
    
2. **"Create Figma-ready components list"**
    
3. **"Create database & API architecture"**
    
4. **"Create full technical specification"**
    
5. **"Now design the MOBILE APP UX structure"**
    

If you approve this structure, your next reply can simply be:

✅ **"Proceed to content briefs for all 20 pages"**

I am ready.
---
