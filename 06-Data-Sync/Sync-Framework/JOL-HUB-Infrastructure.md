27.12.2025
https://www.kimi.com/chat/19a6f1bb-3102-89fa-8000-0976fe976bbe

# **ENTERPRISE DEPLOYMENT RUNBOOK: JOL-HUB Infrastructure**

**For:** Windows 11 (Docs) + Ubuntu 24.04 LTS (Dev) + AlmaLinux 9.6 (Local Prod)  
**Scope:** 100 000+ Catholic parish/funeral/cemetery sites  
**Compliance:** GDPR, EU Digital Services Act, Roman Canon Law (CIC 1300‑1307), WCAG 2.2 AA  
**Mindset:** Paranoid, air‑gap‑friendly, student‑debuggable

---

## **LAYER 0 – MASTER PLANNING (Do This First)**

### **0.1. Draw a Network Diagram (on paper)**
```
[Internet]
   │
[Cloudflare] ← DDoS, WAF, DNS for jol-hub.com
   │
[Bitrix24 Cloud] ← journeyoflife.bitrix24.ru
   │
[Your Router/Firewall] ← Port‑forward 51820 (WireGuard), 443 (HTTPS)
   │
┌──┴──────┬──────────────┐
│         │              │
[AlmaLinux 192.168.8.51]  [Ubuntu Dev WS]  [Windows 11 Docs WS]
   │                         │                    │
1C‑Bitrix + PG + Redis   PhpStorm/PyCharm      Obsidian
Nginx + WireGuard          Docker + VPN         VPN Client
```

### **0.2. Create a Master Password Vault (Never in Git)**
- **Tool:** Bitwarden CLI or 1Password CLI (both have Ubuntu/Windows binaries)
- **Why:** One source of truth for 200+ secrets (DB passes, API keys, SSL certs)
- **Action:**
  ```bash
  # On Ubuntu (do this first)
  sudo apt install unzip
  curl -fsS https://bitwarden.com/download/cli/ | bunzip2 > bw
  chmod +x bw
  sudo mv bw /usr/local/bin/
  bw login --apikey  # store creds in ~/.bw-session
  ```
  Save your vault password in Bitwarden itself—**never in a file**.

---

## **LAYER 1 – WINDOWS 11 (Documentation & Light VPN)**

### **1.1. Install Obsidian & Vault**
- **Why:** Human‑readable memory of every config, error, and decision
- **Steps:**
  1. Download Obsidian from `obsidian.md` → Install.
  2. Create vault at `C:\Users\YourName\obsidian-vaults\jol-hub`.
  3. **Enable Git** (so vault is versioned):
     - Install Git for Windows: `git-scm.com/download/win`
     - In Obsidian Settings → Community Plugins → Turn on Safe Mode OFF → Browse → Install **Obsidian Git**.
     - Configure plugin:  
       - Vault backup interval: 30 min
       - Auto pull updates: ON
       - Commit message: `vault: auto‑sync`
  4. **Folder structure inside vault:**
     ```
     jol-hub/
     ├── 00-Master-Keys/           (encrypted notes with vault refs, not raw secrets)
     ├── 01-Network-Diagrams/
     ├── 02-Compliance-GDPR/
     ├── 03-Compliance-Canonical/
     ├── 04-AlmaLinux-Server/
     ├── 05-Ubuntu-Dev/
     ├── 06-Bitrix24/
     ├── 07-Deployments/
     └── diary.md                  (daily log, student writes here)
     ```

### **1.2. Install WireGuard Client (VPN to AlmaLinux)**
- **Why:** All traffic to 192.168.8.51 must be encrypted, even on LAN
- **Steps:**
  1. Download WireGuard from `wireguard.com/install/`.
  2. Open PowerShell as **Administrator**:
     ```powershell
     # Generate Windows keypair
     wg genkey | Out-File -FilePath $env:USERPROFILE\wg-client.key
     wg pubkey < $env:USERPROFILE\wg-client.key | Out-File -FilePath $env:USERPROFILE\wg-client.pub
     ```
  3. Save the public key to Obsidian (`00-Master-Keys/wireguard-client-public.md`).

---

## **LAYER 2 – UBUNTU 24.04 LTS (Development Workstation)**

### **2.1. OS Hardening (First Boot)**
```bash
# Update everything
sudo apt update && sudo apt full-upgrade -y

# Install basic tools
sudo apt install -y curl wget git vim ufw fail2ban

# Set timezone to Lithuania (or your timezone)
sudo timedatectl set-timezone Europe/Vilnius

# Disable password SSH (key only)
sudo sed -i 's/#PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config
sudo systemctl restart sshd

# Enable UFW (firewall) – allow only what we need
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 22/tcp  # SSH
sudo ufw allow 51820/udp # WireGuard (we'll set this up later)
sudo ufw allow 8000:9000/tcp # Dev ports (Next.js, Django)
sudo ufw enable
sudo ufw status verbose

# Install fail2ban (blocks brute-force attacks)
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```
**Verification:** `sudo ufw status` shows rules; `sudo fail2ban-client status` shows active jails.

### **2.2. Install Docker & Docker Compose**
```bash
# Add Docker repo
sudo apt install -y ca-certificates gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Add to sources
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list

# Install
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Add user to docker group (so no sudo needed)
sudo usermod -aG docker $USER
newgrp docker  # activate now

# Test
docker run hello-world
```
**Verification:** `docker ps` shows no errors.

### **2.3. Install Node.js 22 LTS & Python 3.12**
```bash
# Node.js via NodeSource
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt install -y nodejs

# Python 3.12 (Ubuntu 24 has it)
sudo apt install -y python3.12 python3.12-venv python3-pip

# Verify
node -v  # v22.x.x
npm -v   # 10.x.x
python3.12 -V  # 3.12.x
```

### **2.4. Install JetBrains IDEs (PhpStorm, PyCharm, DataGrip)**
1. **Download:** `jetbrains.com/toolbox-app/` (install Toolbox first).
2. **Install via Toolbox:**
   - Open Toolbox → Install PhpStorm, PyCharm Professional, DataGrip.
3. **License:** Use JetBrains Account or license server. **Save license key in Bitwarden**, not in plain text.

### **2.5. Git Configuration (Signed Commits)**
```bash
# Set identity
git config --global user.name "Your Full Name"
git config --global user.email "you@jol-hub.com"

# GPG signing (required for compliance)
sudo apt install -y gnupg
gpg --full-generate-key
# Choose (1) RSA, 4096 bits, no expiry
# Enter your name/email (same as git)
# Set a strong passphrase, save in Bitwarden

# List key and copy the LONG ID
gpg --list-secret-keys --keyid-format LONG
# Output: sec   rsa4096/ABCD1234EFGH5678 2025-12-27 [SC]
# Key ID is: ABCD1234EFGH5678

# Tell Git to use this key
git config --global user.signingkey ABCD1234EFGH5678
git config --global commit.gpgsign true

# Export public key to GitHub
gpg --armor --export ABCD1234EFGH5678 | pbcopy  # or copy manually
# Go to github.com/settings/keys → New GPG Key → Paste
```

### **2.6. SSH Keys for GitHub & AlmaLinux**
```bash
# Generate ED25519 key (faster than RSA)
ssh-keygen -t ed25519 -C "you@jol-hub.com" -f ~/.ssh/id_jol_hub
# Press Enter for no passphrase (or set one, store in Bitwarden)

# Add to SSH agent
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_jol_hub

# Copy public key to clipboard
cat ~/.ssh/id_jol_hub.pub | xclip -selection clipboard
# Add to github.com/settings/keys → New SSH Key

# Also, copy to AlmaLinux (we'll do this after server setup)
```

### **2.7. Install WireGuard (Ubuntu as VPN Client)**
```bash
sudo apt install -y wireguard
# Generate Ubuntu keypair
wg genkey | sudo tee /etc/wireguard/private.key
sudo chmod 600 /etc/wireguard/private.key
sudo cat /etc/wireguard/private.key | wg pubkey | sudo tee /etc/wireguard/public.key

# Save public key to Obsidian
```

---

## **LAYER 3 – ALMALINUX 9.6 SERVER (192.168.8.51)**

### **3.1. OS Hardening (Console Access Required)**
```bash
# Login as root (or sudo user)
# Update system
sudo dnf update -y

# Install basics
sudo dnf install -y epel-release
sudo dnf install -y git vim curl wget fail2ban firewalld

# Set hostname
sudo hostnamectl set-hostname alma-jol-hub-01.local

# Set timezone
sudo timedatectl set-timezone Europe/Vilnius

# Disable password SSH
sudo sed -i 's/#PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config
sudo systemctl restart sshd

# Enable firewalld
sudo systemctl enable firewalld
sudo systemctl start firewalld

# Allow only necessary ports
sudo firewall-cmd --permanent --add-service=ssh
sudo firewall-cmd --permanent --add-port=51820/udp  # WireGuard
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --reload
sudo firewall-cmd --list-all

# SELinux: Enforcing (paranoid mode)
sudo setenforce 1
sudo sed -i 's/SELINUX=permissive/SELINUX=enforcing/' /etc/selinux/config
```
**Verification:** `sestatus` shows "Enforcing"; `firewall-cmd --list-all` shows rules.

### **3.2. Install PostgreSQL 16**
```bash
# Add PostgreSQL repo
sudo dnf install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-9-x86_64/pgdg-redhat-repo-latest.noarch.rpm
sudo dnf -qy module disable postgresql

# Install server
sudo dnf install -y postgresql16-server postgresql16-contrib

# Initialize DB
sudo /usr/pgsql-16/bin/postgresql-16-setup initdb

# Enable and start
sudo systemctl enable postgresql-16
sudo systemctl start postgresql-16

# Harden PostgreSQL
sudo -u postgres psql
# Inside psql:
\password postgres
# Enter strong password, save in Bitwarden

# Edit pg_hba.conf to require scram-sha-256
sudo vim /var/lib/pgsql/16/data/pg_hba.conf
# Change all "trust" or "md5" to "scram-sha-256"

# Edit postgresql.conf for listening
sudo vim /var/lib/pgsql/16/data/postgresql.conf
# Set:
listen_addresses = 'localhost,192.168.8.51'
ssl = on

# Restart
sudo systemctl restart postgresql-16

# Create diocese database (example)
sudo -u postgres createdb jol_diocese_vilnius
```

### **3.3. Install Redis 7**
```bash
sudo dnf install -y redis
sudo systemctl enable redis
sudo systemctl start redis

# Secure: set password
sudo vim /etc/redis.conf
# Find #requirepass foobared → Change to:
requirepass YourStrongRedisPass  # Save in Bitwarden

# Listen only on LAN IP
bind 127.0.0.1 192.168.8.51

# Restart
sudo systemctl restart redis
```

### **3.4. Install 1C‑Bitrix: Website Management (Boxed)**
**Note:** 1C‑Bitrix is a PHP application. We'll use Nginx + PHP-FPM.
```bash
# Install PHP 8.2 (Bitrix recommends 8.1+)
sudo dnf install -y https://rpms.remirepo.net/enterprise/remi-release-9.rpm
sudo dnf module enable -y php:remi-8.2
sudo dnf install -y php php-fpm php-pgsql php-redis php-curl php-gd php-xml php-mbstring php-zip

# Install Nginx
sudo dnf install -y nginx
sudo systemctl enable nginx

# Create data directory for Bitrix
sudo mkdir -p /var/www/bitrix
sudo chown -R nginx:nginx /var/www/bitrix

# Download Bitrix boxed
cd /tmp
wget https://www.1c-bitrix.ru/download/box_bitrix_server.tar.gz  # replace with actual URL
sudo tar -xzf box_bitrix_server.tar.gz -C /var/www/bitrix/

# Set permissions
sudo chown -R nginx:nginx /var/www/bitrix
sudo chmod -R 755 /var/www/bitrix

# Configure PHP-FPM pool for Bitrix
sudo cp /etc/php-fpm.d/www.conf /etc/php-fpm.d/bitrix.conf
sudo vim /etc/php-fpm.d/bitrix.conf
# Change:
user = nginx
group = nginx
listen = /run/php-fpm/bitrix.sock
listen.owner = nginx
listen.group = nginx

# Start PHP-FPM
sudo systemctl enable php-fpm
sudo systemctl start php-fpm
```

### **3.5. Configure Nginx for Bitrix & Next.js**
```bash
sudo vim /etc/nginx/conf.d/bitrix.conf
```

**Inside `/etc/nginx/conf.d/bitrix.conf`:**
```nginx
server {
    listen 80;
    listen 443 ssl http2;
    server_name alma-jol-hub-01.local 192.168.8.51;

    ssl_certificate /etc/ssl/certs/alma-jol-hub-01.crt;  # We'll create
    ssl_certificate_key /etc/ssl/private/alma-jol-hub-01.key;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;

    root /var/www/bitrix;
    index index.php index.html;

    # Bitrix typical rules
    location / {
        try_files $uri $uri/ @bitrix;
    }

    location @bitrix {
        fastcgi_pass unix:/run/php-fpm/bitrix.sock;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root/bitrix/urlrewrite.php;
    }

    location ~ \.php$ {
        fastcgi_pass unix:/run/php-fpm/bitrix.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

    # Deny access to sensitive files
    location ~ /\. { deny all; }
    location ~ /bitrix/admin { allow 192.168.8.0/24; deny all; }
}
```

### **3.6. Generate Self‑Signed SSL Certificate (for local)**
```bash
sudo mkdir -p /etc/ssl/private
sudo openssl req -x509 -nodes -days 3650 -newkey rsa:4096 \
  -keyout /etc/ssl/private/alma-jol-hub-01.key \
  -out /etc/ssl/certs/alma-jol-hub-01.crt \
  -subj "/C=LT/ST=Vilnius/L=Vilnius/O=JOL-HUB/CN=alma-jol-hub-01.local"

sudo chmod 600 /etc/ssl/private/alma-jol-hub-01.key
```

### **3.7. Start Nginx**
```bash
sudo nginx -t  # Test config
sudo systemctl enable nginx
sudo systemctl start nginx

# Check status
sudo systemctl status nginx
```

**Verification:** Browse to `https://192.168.8.51` – should show Bitrix setup wizard (ignore SSL warning for now).

### **3.8. Install WireGuard (AlmaLinux as VPN Server)**
```bash
sudo dnf install -y wireguard-tools

# Generate server keys
wg genkey | sudo tee /etc/wireguard/server-private.key
sudo chmod 600 /etc/wireguard/server-private.key
sudo cat /etc/wireguard/server-private.key | wg pubkey | sudo tee /etc/wireguard/server-public.key

# Create config
sudo vim /etc/wireguard/wg0.conf
```

**Inside `/etc/wireguard/wg0.conf`:**
```ini
[Interface]
Address = 10.8.0.1/24
ListenPort = 51820
PrivateKey = <contents of server-private.key>
PostUp = firewall-cmd --zone=public --add-port 51820/udp && firewall-cmd --zone=public --add-masquerade
PostDown = firewall-cmd --zone=public --remove-port 51820/udp && firewall-cmd --zone=public --remove-masquerade
```

**Enable IP forwarding:**
```bash
sudo vim /etc/sysctl.conf
# Add:
net.ipv4.ip_forward = 1

sudo sysctl -p
```

**Start WireGuard:**
```bash
sudo systemctl enable wg-quick@wg0
sudo systemctl start wg-quick@wg0
```

**Verification:** `sudo wg show` shows interface.

---

## **LAYER 4 – CONNECTING ALL MACHINES**

### **4.1. Copy SSH Public Key from Ubuntu to AlmaLinux**
```bash
# On Ubuntu terminal
ssh-copy-id -i ~/.ssh/id_jol_hub.pub root@192.168.8.51
# Or if you have a sudo user:
ssh-copy-id -i ~/.ssh/id_jol_hub.pub your-user@192.168.8.51
```

### **4.2. Configure WireGuard Client on Ubuntu**
```bash
sudo vim /etc/wireguard/jol-client.conf
```

**Add:**
```ini
[Interface]
PrivateKey = <Ubuntu private key from /etc/wireguard/private.key>
Address = 10.8.0.2/32
DNS = 1.1.1.1

[Peer]
PublicKey = <AlmaLinux server public key>
Endpoint = 192.168.8.51:51820
AllowedIPs = 10.8.0.0/24, 192.168.8.51/32
PersistentKeepalive = 25
```

**Start client:**
```bash
sudo systemctl enable wg-quick@jol-client
sudo systemctl start wg-quick@jol-client
```

**Verification:** `sudo wg show` shows handshake.

### **4.3. Configure WireGuard Client on Windows**
1. Open WireGuard app → Add Tunnel → Add empty.
2. Name: `jol-client`.
3. Config:
   ```ini
   [Interface]
   PrivateKey = <Windows private key>
   Address = 10.8.0.3/32
   DNS = 1.1.1.1
   
   [Peer]
   PublicKey = <AlmaLinux server public key>
   Endpoint = 192.168.8.51:51820
   AllowedIPs = 10.8.0.0/24, 192.168.8.51/32
   PersistentKeepalive = 25
   ```
4. Save → Activate.

---

## **LAYER 5 – GITHUB & CI/CD SETUP**

### **5.1. Clone Repository on Ubuntu**
```bash
mkdir -p ~/Projects/jol-hub
cd ~/Projects/jol-hub
git clone git@github.com:IterVitae/jol-hub.git .
git checkout -b develop  # all work here
```

### **5.2. Configure GitHub Secrets (in Browser)**
Go to `github.com/IterVitae/jol-hub/settings/secrets/actions`:
- `PG_PASS_ALMA`: PostgreSQL postgres password
- `REDIS_PASS_ALMA`: Redis password
- `BITRIX24_WEBHOOK`: Full webhook URL for journeyoflife.bitrix24.ru
- `SENTRY_DSN`: Project DSN
- `VAULT_ADDR`: Bitwarden/1Password Connect URL (if using)

### **5.3. Create GitHub Actions Runner (Self‑Hosted, on Ubuntu)**
```bash
# On Ubuntu
mkdir actions-runner && cd actions-runner
curl -o actions-runner-linux-x64-2.322.0.tar.gz -L https://github.com/actions/runner/releases/download/v2.322.0/actions-runner-linux-x64-2.322.0.tar.gz
tar xzf ./actions-runner-linux-x64-2.322.0.tar.gz

# Configure
./config.sh --url https://github.com/IterVitae/jol-hub --token YOUR_GITHUB_TOKEN

# Install as service
sudo ./svc.sh install
sudo ./svc.sh start
```

**Verification:** Runner appears in GitHub Settings → Actions → Runners.

---

## **LAYER 6 – JETBRAINS IDE CONFIGURATION**

### **6.1. PhpStorm – Connect to AlmaLinux (Deployment)**
1. **Open PhpStorm** → File → Settings → Build, Execution, Deployment → Deployment.
2. Add **+** → SFTP → Name: `AlmaLinux-Bitrix`.
3. **Connection:**
   - Host: `192.168.8.51`
   - Port: `22`
   - User: `nginx` (or your sudo user)
   - Auth type: Key pair → Select `~/.ssh/id_jol_hub`
4. **Mappings:**
   - Local path: `~/Projects/jol-hub/sites/`
   - Deployment path: `/var/www/bitrix/`
5. **Test Connection** → Should succeed.

### **6.2. DataGrip – Connect to PostgreSQL on AlmaLinux**
1. **Open DataGrip** → New Data Source → PostgreSQL.
2. **Host:** `192.168.8.51`
3. **Port:** `5432`
4. **Database:** `jol_diocese_vilnius`
5. **User:** `postgres`
6. **Password:** Store in DataGrip with "Remember" (encrypted).
7. **SSH:** Use same SSH config to tunnel.
8. **Test Connection** → Should succeed.

### **6.3. PyCharm – Configure Django Interpreter**
1. **Open PyCharm** → Open `~/Projects/jol-hub/drivers/jol-driver.py`.
2. **File → Settings → Project: jol-driver → Python Interpreter**.
3. Add **+** → Add Interpreter → System Interpreter → Select `/usr/bin/python3.12`.
4. **Install requirements:**
   ```bash
   cd ~/Projects/jol-hub/drivers/
   pip install -r requirements.txt  # (create this file: psycopg2, requests, pyyaml)
   ```

---

## **LAYER 7 – PROJECT WORKFLOW (Daily Student Tasks)**

### **7.1. Create a New Parish Site (Example)**
```bash
# On Ubuntu
cd ~/Projects/jol-hub/drivers/
python jol-driver.py --country lt --diocese vilnius --parish "St. Anne" --type parish
```
- This will:
  1. Generate folder `sites/lt/vilnius/st-anne-{uuid}/`.
  2. Create Next.js files.
  3. Push to GitHub.
  4. Create Bitrix24 task.
  5. Generate Obsidian note.

### **7.2. Develop & Test**
```bash
# In the generated site folder
cd sites/lt/vilnius/st-anne-{uuid}/
npm install
npm run dev  # starts on localhost:3000
# Open browser, test all pages
```

### **7.3. Commit & Push (Signed)**
```bash
git add .
git commit -S -m "feat: st-anne site scaffold"
git push origin develop
```
- The `-S` flag signs with GPG key.

### **7.4. CI/CD Pipeline (Automatic)**
- GitHub Actions runs:
  - Lighthouse → must be ≥95
  - axe → 0 violations
  - Build Docker → push to `europe-docker.pkg.dev`
  - Deploy to AlmaLinux via webhook
  - Update Bitrix24 task to "Finished"
  - Write to Obsidian diary

---

## **LAYER 8 – BACKUP & DISASTER RECOVERY**

### **8.1. AlmaLinux Daily Backup (Cron)**
```bash
sudo crontab -e
```
**Add:**
```cron
0 2 * * * /usr/local/bin/backup-jol.sh
```

**Create `/usr/local/bin/backup-jol.sh`:**
```bash
#!/bin/bash
DATE=$(date +%Y%m%d)
BACKUP_DIR="/backup/jol-hub/$DATE"
mkdir -p $BACKUP_DIR

# PostgreSQL
pg_dumpall -U postgres -f $BACKUP_DIR/postgres-all.sql

# Bitrix files
tar -czf $BACKUP_DIR/bitrix-files.tar.gz /var/www/bitrix/

# SSL certs
cp /etc/ssl/certs/*.crt $BACKUP_DIR/
cp /etc/ssl/private/*.key $BACKUP_DIR/

# Encrypt backup with GPG
gpg --symmetric --cipher-algo AES256 --output $BACKUP_DIR.tar.gz.gpg $BACKUP_DIR
# Password stored in Bitwarden

# Upload to Google Cloud (if configured)
# gsutil cp $BACKUP_DIR.tar.gz.gpg gs://jol-hub-backup/

# Clean old backups (>30 days)
find /backup/jol-hub/ -type d -mtime +30 -exec rm -rf {} \;
```

**Make executable:**
```bash
sudo chmod +x /usr/local/bin/backup-jol.sh
```

### **8.2. Ubuntu Workstation Backup**
```bash
# On Ubuntu, use restic to backup ~/Projects to AlmaLinux
sudo apt install -y restic

# Init repo (first time)
restic -r sftp:root@192.168.8.51:/backup/ubuntu-workstation init

# Daily backup (cron)
crontab -e
0 * * * * restic -r sftp:root@192.168.8.51:/backup/ubuntu-workstation backup ~/Projects
```

---

## **LAYER 9 – MONITORING & LOGGING**

### **9.1. Install Prometheus & Grafana (on AlmaLinux)**
```bash
sudo dnf install -y prometheus grafana
sudo systemctl enable prometheus grafana-server
sudo systemctl start prometheus grafana-server

# Open Grafana: http://192.168.8.51:3000 (admin/admin)
# Add data source: Prometheus http://localhost:9090
```

### **9.2. Install Node Exporter (on AlmaLinux)**
```bash
sudo dnf install -y node_exporter
sudo systemctl enable node_exporter
sudo systemctl start node_exporter
# Scrapes metrics at http://192.168.8.51:9100
```

### **9.3. Configure Sentry (Error Tracking)**
```bash
# On Ubuntu, in each Next.js site
npm install @sentry/nextjs
npx @sentry/wizard@latest -i nextjs
# Provide DSN from GitHub Secrets
```

---

## **LAYER 10 – COMPLIANCE CHECKLISTS (Student Must Fill)**

### **10.1. GDPR Compliance Checklist**
- [ ] All forms have "I consent" checkbox (unchecked by default)
- [ ] Data retention policy PDF linked in footer (27 languages)
- [ ] Bitrix24 contact marked as "GDPR Controller" for each parish
- [ ] PostgreSQL `DELETE CASCADE` on parishioner table
- [ ] Sentry configured to NOT log PII (disable breadcrumbs)
- [ ] Obsidian diary entry: "GDPR audit passed on 2025-12-27"

### **10.2. Canonical Law Compliance Checklist**
- [ ] Donation ledger hash includes no PII (email is SHA256-salted)
- [ ] Parish owns database (JOL-HUB = Processor, not Controller)
- [ ] Bitrix24 task type "Canonical Review" assigned to canon lawyer
- [ ] Terms of Service include "Canon 1300-1307" clause
- [ ] Obsidian diary entry: "Canon lawyer signed off on 2025-12-27"

---

## **LAYER 11 – DAILY STUDENT ROUTINE**

### **Morning (15 min)**
1. **Windows:** Open Obsidian → pull latest diary.
2. **Ubuntu:** `git pull origin develop` → check for new tasks.
3. **AlmaLinux:** `sudo dnf update --security` (check for CVEs).

### **Work Session**
1. Create new parish: `python jol-driver.py ...`
2. Develop in PhpStorm/PyCharm.
3. Test locally: `npm run dev`.
4. Commit: `git commit -S -m "..."`.
5. Push → watch GitHub Actions.

### **Evening (10 min)**
1. Write Obsidian diary entry:
   ```markdown
   ## 2025-12-27
   - [x] Created st-anne-{uuid}
   - [x] Lighthouse 96, axe 0
   - [ ] Waiting for bishop's signature (blocked)
   ```
2. Backup Ubuntu: `restic backup ...`.

---

## **SUMMARY: What Goes Where**

| Machine | Installed Software | Purpose |
|---------|-------------------|---------|
| **Windows 11** | Obsidian + Git + WireGuard | Documentation, VPN client, diary |
| **Ubuntu 24.04** | Docker, Node, Python, JetBrains IDEs, WireGuard, GPG | Full development, CI runner, secure tunnel |
| **AlmaLinux 9.6** | Nginx, PHP-FPM, PostgreSQL, Redis, Bitrix, WireGuard, Prometheus | Local production mirror, database, monitoring |
| **Cloud** | Bitrix24, GitHub, Sentry, optional GCP | SaaS coordination, error tracking, backups |

**Golden Rule:** **Never type a secret into a file.** Use Bitwarden CLI to inject at runtime. All communication between machines goes through WireGuard (even on LAN). Every commit is signed. Every diary entry is versioned. Every backup is encrypted.

---

**Final Student Checklist:**
- [ ] All three machines can ping each other via WireGuard IP (`10.8.0.x`).
- [ ] `ssh root@192.168.8.51` works from Ubuntu without password.
- [ ] Git commits show "Verified" badge on GitHub.
- [ ] Bitrix24 task auto-creates when you push a new site.
- [ ] Obsidian vault syncs from Ubuntu pull.
- [ ] `https://192.168.8.51` shows Bitrix setup wizard.
- [ ] `python jol-driver.py --help` shows all options.

If any step fails, **stop immediately**, document error in Obsidian `diary.md`, and only proceed after resolution. **Compliance is not optional.**