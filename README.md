# Virtual Infrastructure Deployment and Security Project

## Disclaimer
This project is intended for educational and lab purposes only.  
All testing was conducted in an isolated virtual environment.  
No real-world systems were targeted or harmed.

This project demonstrates the deployment and securing of a virtual infrastructure using virtualization tools. It covers:

- Deployment and configuration of **pfSense** as a firewall with WAN, LAN, and DMZ interfaces.
- Setup of **OpenVPN** for secure remote access.
- Configuration of **Suricata** as an IDS/IPS.
- Deployment of a hardened **Ubuntu Server 22.04 LTS** VM with SSH key authentication, iptables firewall, and rsyslog.
- Containerized deployment of a **Flask web application** using Docker and Docker Compose.
- Real-time application monitoring with **Prometheus** and **Grafana**.
- Centralized security monitoring with **Wazuh** across endpoints (Ubuntu, Kali, Windows 10) and integration with Suricata alerts.

The entire setup is designed to run in a virtual environment (e.g., VirtualBox or VMware) for educational and testing purposes.

## Architecture Overview

- **pfSense VM**: Firewall/router with WAN (NAT), LAN (host-only), and optional DMZ (host-only).
- **Ubuntu Server VM**: Hardened server hosting the Wazuh manager.
- **Kali Linux VM**: Testing/attacker machine with Wazuh agent.
- **Windows 10 VM**: Endpoint with Wazuh agent.
- **Docker Containers** (on host or separate VM): Flask app, Prometheus, Grafana.
- **Wazuh**: Centralized manager on Ubuntu Server; agents on all VMs including pfSense to forward Suricata logs.

Traffic is controlled via pfSense rules, with Suricata monitoring for intrusions. Wazuh collects and analyzes logs from all endpoints, including Suricata's network IDS data.

## Prerequisites

- Virtualization platform (VirtualBox or VMware).
- ISO images:
  - pfSense (latest)
  - Ubuntu Server 22.04 LTS
  - Kali Linux
  - Windows 10
- Host machine with sufficient resources (≥ 16 GB RAM recommended).
- Basic networking and Linux knowledge.

## Setup Instructions

### 1. Deploy pfSense Firewall

![pfSense Dashboard](https://sanuja.com/blog/wp-content/uploads/2020/12/pfSense_web_interface_advanced_dashboard.jpg)

1. Create VM:
   - OS: FreeBSD (64-bit)
   - RAM: ≥ 2 GB
   - Disk: ≥ 10 GB

2. Network adapters:
   - Adapter 1 → NAT (WAN)
   - Adapter 2 → Host-only (LAN)
   - Adapter 3 → Host-only (DMZ, optional)

3. Install pfSense from ISO, assign interfaces, set admin password.

4. Access web GUI: `https://<LAN_IP>` (default often 192.168.1.1).

5. Configure firewall rules:
   - Block vulnerable protocols on LAN.
   - Block all traffic from DMZ to LAN.

#### OpenVPN Setup

![OpenVPN Wizard in pfSense](https://www.wundertech.net/wp-content/uploads/2022/01/OpenVPN_pfSense4.jpg)

6. Set up OpenVPN (VPN → OpenVPN → Wizards):
   - Create CA and server certificate.
   - Configure users and export client configs.
   - Allow UDP 1194 on WAN.

#### Suricata IDS/IPS

![Suricata Configuration in pfSense](https://forum.netgate.com/assets/uploads/files/1566312969110-suricataalerts.png)

7. Install & configure Suricata:
   - System → Package Manager → Install Suricata.
   - Enable on WAN/LAN in IPS mode.
   - Enable EVE JSON logging.

### 2. Deploy Hardened Ubuntu Server (Wazuh Manager Host)

![Ubuntu Terminal with Firewall Rules](https://www.cyberciti.biz/media/new/faq/2016/01/How-to-list-all-iptables-rules-in-Linux.jpg)

1. Install Ubuntu Server 22.04 LTS with OpenSSH enabled.

2. Update system:
   ```bash
   sudo apt update && sudo apt upgrade -y

Configure SSH key authentication.
Set up iptables firewall:Bashsudo iptables -P INPUT DROP
sudo iptables -P FORWARD DROP
sudo iptables -P OUTPUT ACCEPT
sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
sudo iptables -A INPUT -i lo -j ACCEPT
sudo apt install iptables-persistent -y
sudo netfilter-persistent save

3. Deploy Wazuh for Endpoint Monitoring & Suricata Integration
Wazuh Main Dashboard
Wazuh Agents Management
3.1 Install Wazuh Manager on Ubuntu Server
Use the assisted installer for the full stack:
Bashcurl -sO https://packages.wazuh.com/4.x/wazuh-install.sh && sudo bash ./wazuh-install.sh -a
Access dashboard at https://<UBUNTU_IP> (change default credentials immediately).
3.2 Install Wazuh Agents & Suricata Integration

Install agents on Ubuntu/Kali, Windows 10, and pfSense as detailed previously.
Add Suricata eve.json monitoring to pfSense agent config for integration.

Suricata alerts will appear in the Wazuh dashboard.
4. Deploy Containerized Web App with Monitoring
Docker Compose Running Containers
Project Structure & Key Files
(As in previous versions: app.py, requirements.txt, Dockerfile, prometheus.yml, docker-compose.yml)
Run:
Bashdocker compose up -d
Prometheus Targets Interface

Prometheus: http://localhost:9090

Grafana Dashboard with Metrics

Grafana: http://localhost:3000 (admin/admin – change password!)
Add Prometheus data source and create dashboard for flask_app_requests_total.

Optional: Temporary Public Exposure with Ngrok
Ngrok Tunnel Terminal
Bashngrok http 5000  # or 3000/9090 as needed
Verification

Test firewall rules and Suricata (run nmap scans from Kali → check alerts in Wazuh/Suricata).
Confirm SSH key-only access.
View real-time metrics in Grafana.
Check Wazuh dashboard for agent connectivity and Suricata alerts.

This project implements security best practices across networking, server hardening, containerization, application monitoring, and centralized endpoint/security event management.
