# Home Security Cloud Monitoring Project

This project deploys a cloud-based security monitoring system using a Google Cloud Ubuntu VM. 
It includes system hardening, IDS installation, brute-force protection, log monitoring, and 
automated email alerts. This project demonstrates foundational cybersecurity and Linux 
administration skills.

---

# 🏗️ Environment Setup & Security Hardening

## Step 1 — Open Google Cloud Console
Begin the project by navigating to the Google Cloud Platform dashboard, where all services and VM resources are managed.
<img width="1303" height="691" alt="01_cloud_project_dashboard" src="https://github.com/user-attachments/assets/2a97ee83-125b-42bc-baac-b0e5f5abd7c4" />


---

## Step 2 — Create the Virtual Machine (Ubuntu 22.04)
Configure a new VM instance with Ubuntu 22.04 LTS, firewall rules, and recommended specs.

Settings used:
- Machine Type: e2-medium  
- Boot Disk: Ubuntu 22.04 LTS  
- Firewall: Allow HTTP and HTTPS  

<img width="1298" height="685" alt="02_vm_instance_page" src="https://github.com/user-attachments/assets/20243eef-13d6-4222-ad3e-a949fcc76dea" />


---

## Step 3 — Verify VM Is Running
Confirm that the instance is deployed and running with an assigned external IP.

<img width="1313" height="684" alt="03_vm_details_page" src="https://github.com/user-attachments/assets/eb674dc5-a636-4efe-ab7d-0b6de17d534b" />


---

## Step 4 — Connect to VM via SSH
Launch an SSH session to configure the machine.

<img width="1111" height="673" alt="Screenshot 2025-12-02 at 6 52 31 PM" src="https://github.com/user-attachments/assets/24ed8b5a-0c21-4a71-a894-9dbdd4a32590" />



---

# 🔐 Initial Security Hardening

## Step 5 — Update System Packages
Update and upgrade system packages to ensure all software is current.

Command:

sudo apt update && sudo apt upgrade -y
<img width="894" height="575" alt="screenshots:06_system_update" src="https://github.com/user-attachments/assets/17a37193-848f-403a-a6da-bb8ef6c4b5d0" />

---

## Step 6 — Create a New Admin User
Add a new non-root administrator account with sudo privileges.

Commands: 
sudo adduser james
sudo usermod -aG sudo james
<img width="889" height="150" alt="User Setup" src="https://github.com/user-attachments/assets/ba8052b6-63df-4a82-9efd-052d3d5e2764" />


---

## Step 7 — Enable and Configure Firewall (UFW)
Enable UFW firewall to restrict unauthorized traffic.

Commands:

sudo ufw allow OpenSSH
sudo ufw enable
<img width="899" height="94" alt="Screenshot 2025-12-02 at 6 47 28 PM" src="https://github.com/user-attachments/assets/69f32472-2a00-4cca-9c73-a76da3825491" />


---

# 🛡️ Install Core Security Tools

## Step 8 — Install Fail2Ban for SSH Protection
Fail2Ban monitors authentication logs and blocks repeated failed login attempts.

Command:

sudo apt install fail2ban -y
<img width="900" height="580" alt="screenshots:09_fail2ban_install" src="https://github.com/user-attachments/assets/9555b1de-694a-482a-aee3-87f425e9eb3b" />


---

## Step 9 — Verify Fail2Ban Is Running
Check that Fail2Ban is active and monitoring SSH logs.

Command:

sudo systemctl status fail2ban
<img width="899" height="335" alt="screenshots:10_fail2ban_running" src="https://github.com/user-attachments/assets/ae5d1ebe-ccc1-49a4-a2bd-ca77e15699ac" />


---

## Step 10 — Install Nginx for Log Generation
Nginx provides HTTP logs that can be monitored for anomalies.

Command:

sudo apt install nginx -y
<img width="900" height="574" alt="Nginx Install" src="https://github.com/user-attachments/assets/b9220066-887d-4246-9ed4-51f631b1017b" />


---

## Step 11 — Install Suricata IDS
Suricata monitors network traffic for suspicious or malicious behavior.

Command:

sudo apt install suricata -y
<img width="796" height="680" alt="17_suricata_install" src="https://github.com/user-attachments/assets/c5f087d4-ed65-4dfc-ad16-5a4d201594f0" />


---

## Step 12 — Confirm Suricata Is Active
Verify that Suricata is running correctly.

Command:

sudo systemctl status suricata

<img width="808" height="322" alt="18_suricata_running" src="https://github.com/user-attachments/assets/e9863f2e-fdd0-475e-986c-8e952b5d7fc4" />

---

# ✉️ Email Alert System Setup

## Step 13 — Install Mailutils
Mailutils allows the system to send email alerts.

Command:

sudo apt install mailutils -y
<img width="898" height="123" alt="Mailutils install" src="https://github.com/user-attachments/assets/2738c67f-5425-46f6-951f-83c0e3cffeee" />


---

## Step 14 — Test Email Alert Function
Send yourself a test message to verify email functionality.

Command:

echo “Security Alert Test” | mail -s “Alert Test” jbpoe095@gmail.com

<img width="809" height="555" alt="Email Test Command Sent" src="https://github.com/user-attachments/assets/01923619-e9ad-417b-8f35-f80cea7d45a3" />

---

# ⚙️ Automation Setup

## Step 15 — Install Cron Service
Install cron, the service used to schedule recurring tasks.

Commands: 

sudo apt install cron -y

sudo systemctl status cron
<img width="892" height="272" alt="Cron install" src="https://github.com/user-attachments/assets/b4576521-021e-465d-b6cc-7a0d9b7a0549" />

---

## Step 16 — Create Security Alert Script
This script checks `/var/log/auth.log` for failed SSH login attempts and sends alerts.

Command:

sudo nano /usr/local/bin/security-alert.sh

Script content: #!/bin/bash

LOGFILE=”/var/log/auth.log”
KEYWORD=“Failed password”
EMAIL=“your-email-here”

if grep “$KEYWORD” $LOGFILE | grep “$(date ‘+%b %d’)” >/dev/null; then
echo “Security Alert: Failed SSH login attempt detected” | mail -s “SSH Alert” $EMAIL
fi

<img width="893" height="578" alt="Screenshot 2025-12-02 at 6 43 04 PM" src="https://github.com/user-attachments/assets/8f17a592-e2f1-4eff-bd5f-350f865633e7" />

---
## Step 17 — Make Script Executable
Change script permissions to allow execution.

Command:

sudo chmod +x /usr/local/bin/security-alert.sh

---

## Step 18 — Add Cron Job
Schedule the script to run every 5 minutes.

Command:

sudo crontab -e
Cron entry:*/5 * * * * /usr/local/bin/security-alert.sh
<img width="891" height="567" alt="Cron Job visible in Nano" src="https://github.com/user-attachments/assets/9289ec50-0c81-4e92-943f-236d618e7791" />


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
