
---

---

### **Paranoid Compliance Configuration: pve-church-01 - The Sacred Host**
**Document Version:** 1.0 | **Server Role:** Primary Host for ~895 Catholic Church Websites | **Guiding Principle:** *"A chain is only as strong as its weakest link. This server is a forged steel chain for the Body of Christ."*

This protocol details the creation of `pve-church-01`, the foundational server hosting all Lithuanian basilicas, cathedrals, and parish websites. We will proceed with the precision of a Swiss watchmaker and the paranoia of a Cold War intelligence officer. Every step is intentional; every command is logged.

---

## **Phase 1: Physical & Firmware Foundation - The Immutable Rock**

Before installing any software, we establish a secure, auditable hardware baseline.

### **Step 1.1: Hardware Inventory & BIOS Sanctification**
*   **1.1.1. Physical Audit:**
    Upon receiving the Supermicro AS-2025HS-TNR, create a **Hardware Manifest**. Document every serial number (chassis, motherboard, CPUs, RAM modules, drives). Photograph the components and seals. Store this in a cryptographically signed PDF (`hardware-manifest-pve-church-01.pdf.asc`).

*   **1.1.2. BIOS/UEFI Hardening (Access via IPMI or Console):**
    This is your first and most critical line of defense. Configure the following **without exception**.
    ```bash
    # These are conceptual steps performed via the BIOS setup utility
    1.  SECURITY TAB:
        - Set Supervisor Password: (Use a 20+ character passphrase, stored in Vault).
        - Secure Boot: [Enabled] -> Mode: [Custom]
        - TPM State: [Enabled] -> TPM 2.0 UEFI Spec Version.
        - Intel Platform Trust Technology (PTT): [Enabled].
    2.  ADVANCED TAB:
        - CPU Configuration:
            * SVM Mode (Virtualization): [Enabled].
            * SEV-ES (Secure Encrypted Virtualization): [Enabled] *for future confidential VMs*.
            * IOMMU: [Enabled] *for PCIe passthrough security*.
        - Memory Configuration:
            * Memory RAS: [Enabled] *for ECC error correction*.
    3.  BOOT TAB:
        - Boot Mode Select: [UEFI Only] *Disable Legacy/CSM completely*.
        - Fixed Boot Order: 1. Virtual CD-ROM, 2. Your Boot NVMe Drive.
    4.  SAVE & EXIT:
        - Reset the system. On reboot, enter the BIOS again to verify all settings held.
    ```

---

## **Phase 2: Operating System Installation - The Hardened Base**

We install AlmaLinux 9.6 not manually, but via a reproducible, automated Kickstart file to eliminate human error.

### **Step 2.1: Create the Paranoid Kickstart File**
Create a file named `ks-alma9-6-paranoid.cfg` on your provisioning server or USB drive. This file is your blueprint.

```bash
# Kickstart File for pve-church-01 - AlmaLinux 9.6
# --- SYSTEM CONFIGURATION ---
lang en_US.UTF-8
keyboard us
timezone Europe/Vilnius --isUtc
rootpw --iscrypted $6$YourPrecomputedSHA512Hash$...  # Generate with: mkpasswd --method=SHA-512
# --- NETWORK CONFIGURATION ---
network \
  --bootproto=static \
  --device=link \
  --ip=10.10.10.11 \          # Static IP in Management VLAN
  --netmask=255.255.255.0 \
  --gateway=10.10.10.1 \
  --nameserver=1.1.1.1,8.8.8.8 \
  --hostname=pve-church-01.lt.jol \
  --activate
# --- SECURITY & FIREWALL ---
firewall --enabled --service=ssh
selinux --enforcing            # NON-NEGOTIABLE
# --- DISK PARTITIONING - LUKS ENCRYPTED ---
ignoredisk --only-use=nvme0n1  # Explicitly use only the first NVMe for OS
clearpart --all --initlabel
part /boot/efi --fstype="efi" --size=512
part /boot --fstype="ext4" --size=1024
# **CRITICAL: Encrypted Root Partition**
part pv.01 --size=102400 --grow
encrypt --cipher=aes-xts-plain64 --luks-version=luks2 pv.01
volgroup vg_root pv.01
logvol /      --fstype="xfs" --name=lv_root --vgname=vg_root --size=40960
logvol /var   --fstype="xfs" --name=lv_var  --vgname=vg_root --size=20480
logvol /var/log --fstype="xfs" --name=lv_log --vgname=vg_root --size=10240
logvol /var/log/audit --fstype="xfs" --name=lv_audit --vgname=vg_root --size=4096
# --- PACKAGE SELECTION ---
%packages
@^minimal-environment
kexec-tools
vim-enhanced
-aic94xx-firmware
-alsa-*
-wdaemon
%end
# --- POST-SCRIPT FOR INITIAL HARDENING ---
%post
#!/bin/bash
# Set restrictive umask
echo "umask 027" >> /etc/bashrc
echo "umask 027" >> /etc/profile
# Lock root account from direct SSH (force sudo)
passwd -l root
# Install and enable AIDE for file integrity
dnf -y install aide
aide --init
mv /var/lib/aide/aide.db.new.gz /var/lib/aide/aide.db.gz
# Schedule daily integrity check at 3 AM
echo "0 3 * * * /usr/sbin/aide --check" | crontab -
%end
```

### **Step 2.2: Automated Installation & First Boot**
1.  Boot the server from the AlmaLinux 9.6 ISO.
2.  At the boot menu, press `Tab` to edit the entry and append: `inst.ks=https://your-provision-server/ks-alma9-6-paranoid.cfg`
3.  The installation will proceed unattended. **When prompted, enter the LUKS encryption passphrase.** This passphrase must be stored in your HashiCorp Vault and known by at least two senior administrators.
4.  Upon reboot, the system will ask for the LUKS passphrase to unlock the root volume.

---

## **Phase 3: Post-Installation Hardening - The Fortified Walls**

Once the base OS is up, we apply a curated set of security configurations.

### **Step 3.1: Apply CIS Benchmarks Automatically**
We use OpenSCAP, the industry-standard compliance scanner.
```bash
# Install the security guide and scan
dnf install -y scap-security-guide
# Evaluate against the Level 2 Server benchmark for strict compliance
oscap xccdf eval \
  --profile xccdf_org.ssgproject.content_profile_cis_server_l2 \
  --results /var/log/oscap/initial_scan-results.xml \
  --report /var/log/oscap/initial_scan-report.html \
  /usr/share/xml/scap/ssg/content/ssg-almalinux9-ds.xml

# Review the HTML report. It will list failures. Remediate them.
# Example: It may report "SSH root login not disabled." Fix:
echo "PermitRootLogin no" >> /etc/ssh/sshd_config.d/99-paranoid.conf
systemctl restart sshd
```

### **Step 3.2: Configure the Auditing Subsystem (`auditd`)**
Configure `auditd` to log all privileged actions, forming your GDPR and canonical law audit trail.
```bash
# Create a custom rule file for our specific monitoring
cat > /etc/audit/rules.d/99-jol-church.rules << 'EOF'
# Monitor all writes to web content
-w /var/www/html -p wa -k jol_web_content
# Monitor changes to user accounts
-w /etc/passwd -p wa -k jol_identity
-w /etc/shadow -p wa -k jol_identity
# Monitor use of privileged commands (like sudo)
-a always,exit -F arch=b64 -S execve -C uid!=euid -F euid=0 -k jol_privileged_exec
-a always,exit -F arch=b64 -S execve -C gid!=egid -F egid=0 -k jol_privileged_exec
# Monitor access to financial/donor data (conceptual)
-w /opt/jol/databases/donations.sql -p rwa -k jol_financial_data
EOF

# Load the new rules and enable the service
augenrules --load
systemctl enable --now auditd
```

---

## **Phase 4: Proxmox VE Installation & Cluster Formation**

We transform this hardened AlmaLinux host into a Proxmox virtualization node.

### **Step 4.1: Install Proxmox VE**
```bash
# Add the Proxmox VE repository
echo "deb [arch=amd64] http://download.proxmox.com/debian/ceph-quincy bookworm no-subscription" > /etc/apt/sources.list.d/pve-enterprise.list

# Add the GPG key
wget https://enterprise.proxmox.com/debian/proxmox-release-bookworm.gpg -O /etc/apt/trusted.gpg.d/proxmox-release-bookworm.gpg

# Install Proxmox VE kernel and tools
dnf install -y proxmox-ve postfix open-iscsi

# Remove the generic Linux kernel to force boot from Proxmox kernel
dnf remove -y linux-image*

# Reboot into the new Proxmox kernel
reboot
```

### **Step 4.2: Post-Install Network Configuration (CRITICAL)**
After reboot, configure `/etc/network/interfaces` for secure segmentation.
```bash
# File: /etc/network/interfaces
# Management Interface - For Proxmox web UI (https://10.10.10.11:8006) and SSH
auto eno1
iface eno1 inet static
        address 10.10.10.11/24
        gateway 10.10.10.1

# Bridge for Virtual Machines - Connected to second NIC (eno2)
auto vmbr0
iface vmbr0 inet manual
        bridge-ports eno2
        bridge-stp off
        bridge-fd 0
        # Security: Disable MAC address learning limit and enable filtering
        bridge-vlan-aware yes
        bridge-vids 2-4094

# Apply changes
systemctl restart networking
```

### **Step 4.3: Join the Proxmox Cluster**
Assuming `pve-core-01` (10.10.10.15) is your first cluster node, join from `pve-church-01`:
```bash
# On pve-church-01, join the cluster
pvecm add 10.10.10.15
# You will be prompted for the root password of pve-core-01.
# This will sync the cluster configuration.
```

---

## **Phase 5: Workload Deployment - The Virtual Flock**

Now we deploy the church websites using isolated, templated LXC containers.

### **Step 5.1: Create a Secure LXC Template**
First, create an AlmaLinux 9.6 template from which all church containers will be cloned.
1.  In the Proxmox web UI (`https://10.10.10.11:8006`), go to the **local** storage of `pve-church-01`.
2.  Click **"Templates"** -> **"Download"**.
3.  Download the **AlmaLinux 9** standard template.
4.  Once downloaded, convert it to a template.

### **Step 5.2: Deploy the First Church Container via CLI (Automation Ready)**
We'll deploy the container for `vilnius-bazilika`. The process for the other ~894 will be identical and automated.
```bash
# Create the container with a unique ID (e.g., 101)
pct create 101 \
  /var/lib/vz/template/cache/almalinux-9-default_20221110_amd64.tar.xz \
  --storage local-lvm \
  --hostname vilnius-bazilika.gyvenimo-kelias.lt \
  --password 'StrongTempPassword!' \
  --unprivileged 1 \          # CRITICAL: Security isolation
  --features nesting=1 \      # Allows Docker inside the container if needed
  --net0 name=eth0,bridge=vmbr0,ip=dhcp,tag=150 \  # 'tag=150' assigns VLAN 150
  --rootfs local-lvm:8 \      # 8GB root disk
  --memory 1024 --swap 512 \
  --cores 1

# Start the container
pct start 101

# Enter the container to perform initial setup
pct enter 101

# --- INSIDE THE CONTAINER ---
# 1. Set a static IP (or configure DHCP reservation on your router)
dnf install -y vim
vim /etc/NetworkManager/system-connections/eth0.nmconnection
# Edit with: [ipv4] method=manual addresses=192.168.150.10/24 gateway=192.168.150.1

# 2. Install the LAMP stack and 1C-Bitrix prerequisites
dnf install -y nginx mariadb-server php81 php81-php-fpm php81-php-mysqlnd php81-php-gd

# 3. Harden the container (mini version of host hardening)
systemctl enable --now firewalld
firewall-cmd --add-service=http --add-service=https --permanent
firewall-cmd --reload
setsebool -P httpd_can_network_connect 1

# Exit the container
exit
```

### **Step 5.3: Deploy the Bitrix CMS**
1.  Upload the 1C-Bitrix installation files to the container via `pct push`.
2.  Follow the Bitrix web installer, connecting to a **dedicated MariaDB instance**.
    *   **Paranoid Note:** Do NOT run the database in the same container. Use a **dedicated, secure database server VM** (e.g., `db-church-01`) on the same Proxmox host, accessible only over an internal VLAN (e.g., VLAN 200).

### **Step 5.4: Automation for Scale (The Blueprint for 894 More)**
Create an Ansible playbook, `deploy-church-container.yml`:
```yaml
- name: Deploy and Configure a Church Website Container
  hosts: localhost
  vars_prompt:
    - name: church_name
      prompt: "Enter church name (e.g., vilnius-bazilika)"
      private: no
    - name: container_id
      prompt: "Enter unique Proxmox CT ID"
      private: no
  tasks:
    - name: Create LXC container from template
      community.general.proxmox:
        node: pve-church-01
        vmid: "{{ container_id }}"
        password: "{{ vault_proxmox_password }}"
        hostname: "{{ church_name }}.gyvenimo-kelias.lt"
        storage: local-lvm
        template: almalinux-9-default_20221110_amd64.tar.xz
        cores: 1
        memory: 1024
        netif: '{"net0":"name=eth0,bridge=vmbr0,ip=dhcp,tag=150"}'
        unprivileged: yes
      register: container

    - name: Wait for container to be reachable
      wait_for_connection:
        delay: 30
```

---

## **Phase 6: Monitoring & Immutable Compliance Logging**

The system is useless without oversight.

### **Step 6.1: Deploy the Monitoring Agent**
Install the `node_exporter` on the Proxmox **host** (pve-church-01) to expose hardware and host-level metrics to your central Prometheus (on `pve-core-01`).
```bash
# Download and install node_exporter
useradd --no-create-home --shell /bin/false node_exporter
wget https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-amd64.tar.gz
tar xvf node_exporter-1.6.1.linux-amd64.tar.gz
cp node_exporter-1.6.1.linux-amd64/node_exporter /usr/local/bin/
chown node_exporter:node_exporter /usr/local/bin/node_exporter

# Create a systemd service file for it
cat > /etc/systemd/system/node_exporter.service << 'EOF'
[Unit]
Description=Node Exporter
After=network.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload
systemctl enable --now node_exporter
```

### **Step 6.2: Configure Centralized, Immutable Logging**
Configure the host to forward all logs (including those from its containers) to your central log server (e.g., a Grafana Loki instance).
```bash
# Install the Promtail agent (for Grafana Loki)
wget https://github.com/grafana/loki/releases/download/v2.9.1/promtail-linux-amd64.zip
unzip promtail-linux-amd64.zip
mv promtail-linux-amd64 /usr/local/bin/promtail

# Create a configuration file /etc/promtail/config.yaml
# This config sends logs to loki.jol.lt:3100, applying labels for "pve-church-01"
cat > /etc/promtail/config.yaml << 'EOF'
server:
  http_listen_port: 9080
  grpc_listen_port: 0

positions:
  filename: /var/lib/promtail/positions.yaml

clients:
  - url: http://loki.jol.lt:3100/loki/api/v1/push

scrape_configs:
- job_name: system
  static_configs:
  - targets:
      - localhost
    labels:
      job: varlogs
      host: pve-church-01
      __path__: /var/log/**/*.log
- job_name: containers
  static_configs:
  - targets:
      - localhost
    labels:
      job: containers
      host: pve-church-01
      __path__: /var/lib/lxc/*/rootfs/var/log/**/*.log
EOF

# Create a systemd service for Promtail and start it
systemctl enable --now promtail
```

This completes the paranoid configuration of `pve-church-01`. You now have a server that is:
1.  **Hardware-Secured** from the silicon up.
2.  **OS-Hardened** against common vulnerabilities.
3.  **Properly Virtualized** in a secure cluster.
4.  **Hosting isolated workloads** in unprivileged containers.
5.  **Fully Auditable** and monitored.

The pattern for `pve-funeral-01`, `pve-market-01`, and `pve-core-01` is conceptually identical, with changes to the network VLAN tags, allocated resources, and the specific LXC templates or VM images deployed within them.

