# Home Security Cloud Monitoring Project

This repository demonstrates a cloud-based security monitoring lab using an Ubuntu VM on Google Cloud Platform. It includes host hardening, Fail2Ban for brute-force protection, Suricata IDS for network detection, example logging (Nginx), and an automated alerting example.

---

## Table of Contents
- Overview
- Diagram
- Quickstart
- Step-by-step (with screenshots)
  - Create VM
  - Connect and update
  - Hardening: users & firewall
  - Install core tools: Fail2Ban, Nginx, Suricata
  - Email alert test
  - Automation (cron / systemd)
- Example hardened alert script (reference)
- Testing & validation
- Troubleshooting

---

## Overview
Goal: deploy a single-cloud VM lab demonstrating basic host/network monitoring and alerting:
- Harden the host (non-root admin, UFW, SSH config)
- Protect SSH with Fail2Ban
- Monitor network traffic with Suricata
- Generate logs with Nginx (optional)
- Send alerts via mailutils
- Automate checks with cron or systemd timer

---

## Diagram

```mermaid
flowchart TD
    subgraph Internet
        A1[Attacker 1]
        A2[Botnet]
        A3[Legitimate User]
    end

    subgraph CloudVPC
        G1[SSH Service - sshd]
        G2[App Server]
        G3[Auth Logs]
        G4[Fail2Ban]
        G5[Alert System]
        G6[Banned IP List]
    end

    subgraph AdminSOC
        S1[Admin Console]
        S2[Mobile Notification]
    end

    A1 -- Brute Force --> G1
    A2 -- Distributed Attacks --> G1
    A3 -. SSH Login .-> G1
    G1 -- Log Events --> G3
    G3 -- Suspicious Patterns --> G4
    G4 -- IP Ban --> G6
    G4 -- Triggers Alert --> G5
    G5 -- Notification --> S2
    S2 -- Alert Processed --> S1
    S1 -- Updates Security Controls --> G4
```
---

## Quickstart (TL;DR)
1. Create an Ubuntu 22.04 VM in Google Cloud (e2-medium recommended).
2. SSH in (use key-based auth).
3. Update packages, create non-root admin, enable UFW.
4. Install Fail2Ban, Nginx, Suricata, mailutils, cron.
5. Place the alert script at `/usr/local/bin/security-alert.sh`, make it executable, and schedule via systemd timer or cron.

---

## Step-by-step 

### 1 — Open Google Cloud Console
Start in the Google Cloud dashboard to create and manage your VM.
![GCP Project Dashboard](https://github.com/user-attachments/assets/2a97ee83-125b-42bc-baac-b0e5f5abd7c4)
*Figure: GCP console project dashboard*

---

### 2 — Create the Virtual Machine (Ubuntu 22.04)
Create an instance (e2-medium recommended). Ensure firewall rules for required services are set.
![Create VM Instance](https://github.com/user-attachments/assets/20243eef-13d6-4222-ad3e-a949fcc76dea)
*Figure: VM instance creation page*

---

### 3 — Verify VM Is Running (External IP assigned)
Confirm the VM is running and note its external IP for SSH.
![VM Details](https://github.com/user-attachments/assets/eb674dc5-a636-4efe-ab7d-0b6de17d534b)
*Figure: VM details page*

---

### 4 — Connect to VM via SSH
Open an SSH session (use key-based authentication).
![SSH Session](https://github.com/user-attachments/assets/24ed8b5a-0c21-4a71-a894-9dbdd4a32590)
*Figure: SSH session terminal*

---

## 🔐 Initial Security Hardening

### 5 — Update System Packages
Keep the system current:
```bash
sudo apt update && sudo apt upgrade -y
```
![System Update](https://github.com/user-attachments/assets/17a37193-848f-403a-a6da-bb8ef6c4b5d0)
*Figure: System update in terminal*

---

### 6 — Create a New Admin User
Create a non-root admin account and add to sudo group:
```bash
sudo adduser james
sudo usermod -aG sudo james
```
![User Setup](https://github.com/user-attachments/assets/ba8052b6-63df-4a82-9efd-052d3d5e2764)
*Figure: Add user screenshot*

Tip: Upload the admin user's public key to `~james/.ssh/authorized_keys` and disable password auth in `/etc/ssh/sshd_config`:
- PermitRootLogin no
- PasswordAuthentication no
- AllowUsers james

Restart SSH: `sudo systemctl restart sshd`

---

### 7 — Enable and Configure Firewall (UFW)
Allow SSH and any needed service ports, then enable UFW:
```bash
sudo apt install ufw -y
sudo ufw allow OpenSSH
sudo ufw allow 80/tcp   # only if serving HTTP
sudo ufw enable
sudo ufw status verbose
```
![UFW Setup](https://github.com/user-attachments/assets/69f32472-2a00-4cca-9c73-a76da3825491)
*Figure: UFW enable confirmation*

---

## 🛡️ Install Core Security Tools

### 8 — Install Fail2Ban for SSH Protection
```bash
sudo apt install fail2ban -y
```
![Fail2Ban Install](https://github.com/user-attachments/assets/9555b1de-694a-482a-aee3-87f425e9eb3b)
*Figure: Installing Fail2Ban*

Configure a jail (e.g. `/etc/fail2ban/jail.d/ssh.conf`) and restart the service:
```ini
[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
maxretry = 5
bantime = 3600
```
Restart: `sudo systemctl restart fail2ban`

### 9 — Verify Fail2Ban Is Running
```bash
sudo systemctl status fail2ban
```
![Fail2Ban Running](https://github.com/user-attachments/assets/ae5d1ebe-ccc1-49a4-a2bd-ca77e15699ac)
*Figure: Fail2Ban status*

---

### 10 — Install Nginx (for log generation / testing)
```bash
sudo apt install nginx -y
```
![Nginx Install](https://github.com/user-attachments/assets/b9220066-887d-4246-9ed4-51f631b1017b)
*Figure: Nginx install*

Nginx access/error logs provide traffic events useful for IDS/forensics.

---

### 11 — Install Suricata IDS
```bash
sudo apt install suricata -y
```
![Suricata Install](https://github.com/user-attachments/assets/c5f087d4-ed65-4dfc-ad16-5a4d201594f0)
*Figure: Suricata install*

Tune `/etc/suricata/suricata.yaml`, enable EVE JSON output for SIEM forwarding, and set resource limits appropriate for your VM size.

### 12 — Confirm Suricata Is Active
```bash
sudo systemctl status suricata
```
![Suricata Running](https://github.com/user-attachments/assets/e9863f2e-fdd0-475e-986c-8e952b5d7fc4)
*Figure: Suricata status*

---

## ✉️ Email Alert System Setup

### 13 — Install Mailutils (MTA)
```bash
sudo apt install mailutils -y
```
![Mailutils Install](https://github.com/user-attachments/assets/2738c67f-5425-46f6-951f-83c0e3cffeee)
*Figure: Mailutils installation*

Note: For reliable delivery, configure a secure SMTP relay or cloud email provider rather than sending directly from the VM.

### 14 — Test Email Alert Function
Use a quick test to verify outbound mail:
```bash
echo "Security Alert Test" | mail -s "Alert Test" your-email@example.com
```
![Email Test Sent](https://github.com/user-attachments/assets/01923619-e9ad-417b-8f35-f80cea7d45a3)
*Figure: Email test command*

---

## ⚙️ Automation Setup

### 15 — Install Cron Service
```bash
sudo apt install cron -y
sudo systemctl enable --now cron
sudo systemctl status cron
```
![Cron Install](https://github.com/user-attachments/assets/b4576521-021e-465d-b6cc-7a0d9b7a0549)
*Figure: Cron status*

### 16 — Create Security Alert Script
Place a script at `/usr/local/bin/security-alert.sh` that checks `/var/log/auth.log` for failed SSH attempts and sends alerts. (See the "Example hardened alert script" section below.)

Original screenshot (example edit session):
![Security Script Edit](https://github.com/user-attachments/assets/8f17a592-e2f1-4eff-bd5f-350f865633e7)
*Figure: Editing script with nano*

### 17 — Make Script Executable
```bash
sudo chmod +x /usr/local/bin/security-alert.sh
```

### 18 — Add Cron Job (or systemd timer)
Cron example (run every 5 minutes):
```bash
sudo crontab -e
# add:
*/5 * * * * /usr/local/bin/security-alert.sh
```
![Cron Job in Nano](https://github.com/user-attachments/assets/9289ec50-0c81-4e92-943f-236d618e7791)
*Figure: Cron entry in editor*


---

## Example hardened alert script
Place and configure the following as `/usr/local/bin/security-alert.sh` (make executable). This is a hardened example that avoids broken quotes, uses a date fragment matching syslog, and thresholding to avoid email storms. Be sure to set `ALERT_EMAIL` before using.

```bash
#!/usr/bin/env bash
set -euo pipefail
LOGFILE="${LOGFILE:-/var/log/auth.log}"
KEYWORD="${KEYWORD:-Failed password}"
ALERT_EMAIL="${ALERT_EMAIL:-your-email@example.com}"
THRESHOLD="${THRESHOLD:-3}"
TAIL_LINES="${TAIL_LINES:-200}"
HOSTNAME="$(hostname -f)"
DATE_FRAGMENT="$(date '+%b %e')"
count=$(grep -F "$KEYWORD" "$LOGFILE" 2>/dev/null || true | grep -F "$DATE_FRAGMENT" || true | wc -l || true)
count=${count:-0}
if [ "$count" -ge "$THRESHOLD" ] && [ "$count" -gt 0 ]; then
  body="$(printf 'Host: %s\nDetected: %s occurrences of \"%s\" in %s\n\nLast matching lines:\n' "$HOSTNAME" "$count" "$KEYWORD" "$LOGFILE")"
  body+=$(grep -F "$KEYWORD" "$LOGFILE" 2>/dev/null | grep -F "$DATE_FRAGMENT" | tail -n "$TAIL_LINES" || true)
  printf '%s\n' "$body" | mail -s "SSH Alert on ${HOSTNAME}: ${count} ${KEYWORD}" "$ALERT_EMAIL"
  logger -t security-alert "Sent alert: ${count} ${KEYWORD} occurrences; email to ${ALERT_EMAIL}"
fi
```

---

## Testing & validation
- Fail2Ban:
  - `sudo fail2ban-client status sshd`
  - Simulate failed logins (from a test host) and confirm bans.
- Suricata:
  - Use test PCAPs or scanner rules; check `/var/log/suricata/eve.json` (if enabled).
- Email:
  - Check `/var/log/mail.log` and that your SMTP relay/port is reachable.
### Example Brute Force Logging Notification
![IMG_5767](https://github.com/user-attachments/assets/b2587cb9-2e13-4444-9cb8-8d08b51f94cc)



> _Fail2Ban blocks brute force attempts and notifies administrators when an attack is detected._

---

## Troubleshooting
- No mail delivery:
  - Check `journalctl -u postfix` (if using postfix) or `/var/log/mail.log`.
  - Verify outbound port access or configure authenticated relay.
- Cron job not running:
  - Check `/var/log/syslog` for cron entries, or prefer systemd timers.
- Suricata high CPU:
  - Tune detection rules and thread settings in `/etc/suricata/suricata.yaml`.

---
