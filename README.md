# Home Security Cloud Monitoring Project

This project creates a permanent cloud-based security monitoring server using a Google Cloud Ubuntu VM. 
It includes SSH hardening, firewall rules, intrusion detection (Suricata), brute-force protection (Fail2Ban), 
web server log monitoring, and automated email alerts based on suspicious activity.

The goal is to build a real-world cybersecurity homelab that demonstrates:
- Linux administration
- Cloud platform management
- Network security monitoring
- Endpoint hardening
- Log analysis
- Automation using cron and bash scripting

---

## Overview of Tools Used
| Tool | Purpose |
|------|---------|
| **UFW** | Basic firewall rules |
| **Fail2Ban** | SSH brute-force protection |
| **Suricata** | Intrusion Detection System |
| **Nginx** | Log source for monitoring |
| **Mailutils** | Sends automated security alerts |
| **Cron** | Runs scheduled monitoring tasks |

---

## 🚀 Project Steps

### 1. Create Google Cloud VM
- Created Ubuntu 22.04 server instance  
- Configured firewall rules  
- Verified VM connectivity via SSH  

### 2. Harden Server
- Updated system packages  
- Created restricted admin user  
- Enabled UFW  
- Installed Fail2Ban  

### 3. Install Monitoring Tools
- Installed Suricata IDS  
- Installed Nginx for log generation  
- Set up Mailutils for alert delivery  

### 4. Security Automation
- Created `/usr/local/bin/security-alert.sh`  
- Script checks for failed SSH logins  
- Added cron job to run every 5 minutes  
