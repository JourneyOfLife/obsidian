


---

Absolutely. As a **paranoid compliance-driven architect with 30+ years of experience**, I will now provide a **professionally precise, technically accurate, and pedagogically clear** explanation of how to handle **1,200 websites for Lithuania** under a single domain (`https://gyvenimo-kelias.lt/`) with **1,200 subdomains**—and **why you do *not* need 1,200 public IP addresses**.

This is a **critical misconception** that must be corrected early—especially before scaling to 400,000 sites across Europe.

---

# 🛠️ **JOL Infrastructure Strategy: Hosting 1,200 Lithuanian Subdomains Without 1,200 IPs**

> **Version**: January 13, 2026  
> **Audience**: Students, DevOps engineers, compliance officers  
> **Compliance Alignment**: GDPR, SOC2, ISO 27001  
> **Domain**: `https://gyvenimo-kelias.lt/` (Lithuania master domain)

---

## ❌ Myth: “Each Website Needs Its Own Public IP Address”

> **This is false in modern web architecture (post-1999).**

Since the adoption of **HTTP/1.1** and **Server Name Indication (SNI)** in TLS, **a single IP address can host thousands of domains or subdomains**—as long as:
- The web server supports **name-based virtual hosting**
- TLS certificates support **Subject Alternative Names (SANs)** or use **wildcard + SNI**

> ✅ **You only need *one* public IP per VM (or load balancer), not per website.**

---

## ✅ Reality: Use **Name-Based Virtual Hosting** + **Wildcard DNS** + **Automated TLS**

### 🔹 Step-by-Step Architecture for Lithuania (1,200 Subdomains)

---

### ▶️ **Step 1: Understand the Domain Structure**

You have:
- **Primary domain**: `gyvenimo-kelias.lt`
- **Subdomains**: e.g.,  
  - `vilnius-cathedral.gyvenimo-kelias.lt`  
  - `kaunas-diocese.gyvenimo-kelias.lt`  
  - `klaipeda-funeral.gyvenimo-kelias.lt`  
  - ... up to ~1,200

> All resolve to the **same infrastructure** in Lithuania.

---

### ▶️ **Step 2: Assign Only *One* Public IP to the VM (or Load Balancer)**

#### Option A: Single VM (for MVP / Phase 1)
- VM name: `jol-lt-web-01`
- Internal IP: `192.168.10.101` (on `vmbr0` — public VLAN)
- **Public IP**: `212.55.XX.XX` ← assigned by your ISP or cloud provider (e.g., Google Cloud External IP)
- **Only 1 public IP needed**

#### Option B: Scalable Setup (Recommended for Production)
- Use a **reverse proxy/load balancer** (e.g., NGINX, Traefik, HAProxy) on a dedicated VM
- Backend: multiple Bitrix VMs (e.g., `jol-lt-bitrix-01`, `jol-lt-bitrix-02`, ...)
- **Still only 1 public IP** → all traffic enters through the load balancer

> ✅ **Result**: 1,200+ websites → 1 public IP.

---

### ▶️ **Step 3: Configure DNS — Wildcard Record**

In your DNS provider (e.g., Cloudflare, AWS Route 53, or local registrar):

```dns
*.gyvenimo-kelias.lt.    IN    A    212.55.XX.XX
gyvenimo-kelias.lt.      IN    A    212.55.XX.XX
```

> This means:  
> Any subdomain (e.g., `anything.gyvenimo-kelias.lt`) resolves to the **same IP**.

> 💡 **Student Tip**:  
> `*` = wildcard → matches any single label (but not nested like `a.b.gyvenimo-kelias.lt` unless you add `*.*` — which you don’t need here).

---

### ▶️ **Step 4: Web Server Configuration — Name-Based Virtual Hosts**

On your AlmaLinux 9.7 VM (running Apache or NGINX + PHP-FPM for Bitrix):

#### Example: NGINX config snippet
```nginx
server {
    listen 443 ssl http2;
    server_name ~^(?<subdomain>.+)\.gyvenimo-kelias\.lt$;

    ssl_certificate /etc/letsencrypt/live/gyvenimo-kelias.lt/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/gyvenimo-kelias.lt/privkey.pem;

    root /var/www/sites/$subdomain;
    index index.php;

    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_pass unix:/run/php/php82-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }
}
```

> ✅ This dynamically serves content based on the **subdomain name** — no IP change needed.

> ⚠️ For **1C-Bitrix24**, you’ll likely use a **multisite mode** or **separate document roots per site**, but still on the same VM/IP.

---

### ▶️ **Step 5: TLS/SSL Certificates — Use Wildcard + ACME**

You **cannot** issue 1,200 individual certs manually.

Instead:

#### ✅ Use **Let’s Encrypt Wildcard Certificate** via DNS-01 Challenge

1. Install `certbot` + DNS plugin (e.g., `certbot-dns-cloudflare`)
2. Issue one certificate covering:
   - `gyvenimo-kelias.lt`
   - `*.gyvenimo-kelias.lt`

Command:
```bash
sudo certbot certonly --dns-cloudflare --dns-cloudflare-credentials ~/.secrets/cloudflare.ini \
  -d gyvenimo-kelias.lt -d '*.gyvenimo-kelias.lt'
```

> ✅ One certificate → secures all 1,200+ subdomains.

> 🔒 **Why DNS-01?** Because HTTP-01 challenge fails for wildcard certs.

---

### ▶️ **Step 6: How Does the Web Server Know Which Site to Show?**

Answer: **The `Host` header in the HTTP request.**

When a user visits:
```
https://vilnius-cathedral.gyvenimo-kelias.lt
```

Their browser sends:
```
GET / HTTP/1.1
Host: vilnius-cathedral.gyvenimo-kelias.lt
```

Your web server reads the `Host` header → routes to correct site root or Bitrix context.

> ✅ No IP needed per site. Just proper application logic.

---

### ▶️ **Step 7: Scaling Beyond 1 VM (When Needed)**

If 1,200 sites overload one VM:

1. **Add more Bitrix VMs** (e.g., `jol-lt-bitrix-02`, `jol-lt-bitrix-03`)
2. **Keep the same public IP** on a **load balancer VM**
3. Load balancer uses **subdomain → backend mapping** (e.g., via Consul, static map, or database lookup)
4. Still **only 1 public IP**

> 🌍 For 27 countries: assign **1 public IP per country** (e.g., `.lt`, `.lv`, `.ee`), not per site.

| Country | Public IP | Sites Hosted |
|--------|----------|--------------|
| Lithuania | `212.55.XX.10` | ~1,200 |
| Latvia | `212.55.XX.11` | ~900 |
| Estonia | `212.55.XX.12` | ~700 |
| ... | ... | ... |
| Total | **27 IPs** | **~400,000 sites**

> ✅ **Not 400,000 IPs — just 27.**

---

### ▶️ **Step 8: Where Do Public IPs Come From?**

You get them from:

| Source | How |
|-------|-----|
| **Google Cloud Platform (GCP)** | Reserve **static external IPs** in your VPC (free when attached to VM) |
| **ISP (if self-hosted)** | Request a `/28` or `/27` subnet (16–32 IPs) — enough for 27 countries |
| **Hosting Provider** | Lease IPs (e.g., Hetzner, OVH) — usually 1 free per VM, extras for fee |

> 💡 For JOL: Since you use **hybrid GCP + home servers**, use:
> - **GCP External IPs** for production EU traffic (low latency, DDoS protection)
> - **Home server IPs** only for internal registry/PBS (not public websites)

> ⚠️ Never expose home server directly to internet for 1,200 websites — use GCP or CDN.

---

### ▶️ **Step 9: Optional — Add CDN for Performance & Security**

Use **Cloudflare** or **Google Cloud CDN**:

- Point DNS to Cloudflare
- Cloudflare → your single public IP
- Benefits:
  - DDoS protection
  - Free TLS (even without Let’s Encrypt)
  - Caching reduces VM load
  - Hide your origin IP

> ✅ Recommended for all public-facing JOL sites.

---

## 📌 Summary: Key Takeaways for Students

| Misconception | Truth |
|--------------|------|
| “Each website needs its own IP” | ❌ False since 1999 |
| “1,200 sites = 1,200 IPs” | ❌ You only need **1 IP per country** |
| “Subdomains are hard to manage” | ✅ Use wildcard DNS + wildcard TLS + dynamic vhosts |
| “Must buy many IPs” | ❌ 27 IPs cover all 400K sites in Europe |

---

## ✅ Final Architecture for Lithuania

```
User → https://any-subdomain.gyvenimo-kelias.lt
       ↓
DNS: *.gyvenimo-kelias.lt → 212.55.XX.10 (1 public IP)
       ↓
[Optional: Cloudflare CDN]
       ↓
GCP VM (or PROXMOX VM with public IP)
       ↓
NGINX/Apache → reads "Host" header
       ↓
Serves /var/www/sites/any-subdomain/
       ↓
Bitrix24 multisite or independent install
```

> ✅ Secure. Scalable. Compliant. Efficient.

---

## 🔄 Next Steps

1. Reserve **1 static public IP** in GCP (or from ISP)
2. Set up **wildcard DNS** for `*.gyvenimo-kelias.lt`
3. Deploy **AlmaLinux 9.7 VM** with NGINX + PHP + Bitrix
4. Obtain **wildcard TLS cert** via Let’s Encrypt + DNS-01
5. Automate site provisioning via GitHub Actions → update site registry

---

You now understand **modern, scalable, IP-efficient web hosting** — the way enterprise systems actually work.

Let me know if you’d like the full NGINX config, Bitrix multisite setup guide, or Terraform script for GCP IP reservation.
