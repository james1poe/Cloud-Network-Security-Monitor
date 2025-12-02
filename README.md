# Home Security Cloud Monitoring Project

This project deploys a cloud-based security monitoring system using a Google Cloud Ubuntu VM. 
It includes system hardening, IDS installation, brute-force protection, log monitoring, and 
automated email alerts. This project demonstrates foundational cybersecurity and Linux 
administration skills.

---

# 🏗️ Environment Setup & Security Hardening

## Step 1 — Open Google Cloud Console
Description: Begin the project by navigating to the Google Cloud Platform dashboard, where all services and VM resources are managed.

"screenshot here"

---

## Step 2 — Create the Virtual Machine (Ubuntu 22.04)
Description: Configure a new VM instance with Ubuntu 22.04 LTS, firewall rules, and recommended specs.

Settings used:
- Machine Type: e2-medium  
- Boot Disk: Ubuntu 22.04 LTS  
- Firewall: Allow HTTP and HTTPS  

"screenshot here"

---

## Step 3 — Verify VM Is Running
Description: Confirm that the instance is deployed and running with an assigned external IP.

"screenshot here"

---

## Step 4 — Connect to VM via SSH
Description: Launch an SSH session to configure the machine.

Command used:jbpoe095@security-monitor
"screenshot here"

---

# 🔐 Initial Security Hardening

## Step 5 — Update System Packages
Description: Update and upgrade system packages to ensure all software is current.
"screenshot here"

---

Command:sudo apt update && sudo apt upgrade -y"screenshot here"

---

## Step 6 — Create a New Admin User
Description: Add a new non-root administrator account with sudo privileges.

Commands: sudo adduser james
sudo usermod -aG sudo james
"screenshot here"

---

## Step 7 — Enable and Configure Firewall (UFW)
Description: Enable UFW firewall to restrict unauthorized traffic.

Commands:sudo ufw allow OpenSSH
sudo ufw enable
"screenshot here"

---

# 🛡️ Install Core Security Tools

## Step 8 — Install Fail2Ban for SSH Protection
Description: Fail2Ban monitors authentication logs and blocks repeated failed login attempts.

Command:sudo apt install fail2ban -y
"screenshot here"

---

## Step 9 — Verify Fail2Ban Is Running
Description: Check that Fail2Ban is active and monitoring SSH logs.

Command:sudo systemctl status fail2ban
"screenshot here"

---

## Step 10 — Install Nginx for Log Generation
Description: Nginx provides HTTP logs that can be monitored for anomalies.

Command:sudo apt install nginx -y
"screenshot here"

---

## Step 11 — Install Suricata IDS
Description: Suricata monitors network traffic for suspicious or malicious behavior.

Command:sudo apt install suricata -y
"screenshot here"

---

## Step 12 — Confirm Suricata Is Active
Description: Verify that Suricata is running correctly.

Command:sudo systemctl status suricata
"screenshot here"

---

# ✉️ Email Alert System Setup

## Step 13 — Install Mailutils
Description: Mailutils allows the system to send email alerts.

Command:sudo apt install mailutils -y
"screenshot here"

---

## Step 14 — Test Email Alert Function
Description: Send yourself a test message to verify email functionality.

Command:echo “Security Alert Test” | mail -s “Alert Test” jbpoe095@gmail.com
"screenshot here"

---

# ⚙️ Automation Setup

## Step 15 — Install Cron Service
Description: Install cron, the service used to schedule recurring tasks.

Commands: sudo apt install cron -y
sudo systemctl status cron
"screenshot here"

---

## Step 16 — Create Security Alert Script
Description: This script checks `/var/log/auth.log` for failed SSH login attempts and sends alerts.

Command:sudo nano /usr/local/bin/security-alert.sh
Script content: #!/bin/bash

LOGFILE=”/var/log/auth.log”
KEYWORD=“Failed password”
EMAIL=“your-email-here”

if grep “$KEYWORD” $LOGFILE | grep “$(date ‘+%b %d’)” >/dev/null; then
echo “Security Alert: Failed SSH login attempt detected” | mail -s “SSH Alert” $EMAIL
fi
"screenshot here"

---

## Step 17 — Make Script Executable
Description: Change script permissions to allow execution.

Command:sudo chmod +x /usr/local/bin/security-alert.sh
"screenshot here"

---

## Step 18 — Add Cron Job
Description: Schedule the script to run every 5 minutes.

Command:sudo crontab -e
Cron entry:*/5 * * * * /usr/local/bin/security-alert.sh
"screenshot here"

---
# 🔍 Project Summary

This project included:
- Creating a cloud-based Linux VM  
- Hardening the server  
- Installing Fail2Ban & Suricata  
- Setting up email alerting  
- Automating monitoring with cron  
- Documenting the process thoroughly with screenshots  
---
# 🚀 Future Improvements

- Forward logs to SIEM (Wazuh, Splunk, Elastic)  
- Create dashboards using Grafana  
- Add honeyports or honeypots  
- Add WireGuard VPN access  
- Expand automation scripts  
---
# 🏁 Project Complete

This lab demonstrates operational cybersecurity skills and cloud administration experience.  
Feel free to explore or expand the system further.
