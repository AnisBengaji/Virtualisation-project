# 🛡️ Virtual Infrastructure Deployment and Security Project

<div align="center">

![Project Banner](https://img.shields.io/badge/Security-Infrastructure-blue?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Active-success?style=for-the-badge)
![License](https://img.shields.io/badge/License-Educational-orange?style=for-the-badge)

</div>

## 📋 Table of Contents

- [Overview](#-overview)
- [Architecture](#-architecture)
- [Prerequisites](#-prerequisites)
- [Installation Guide](#-installation-guide)
  - [1. pfSense Firewall](#1-pfsense-firewall)
  - [2. Ubuntu Server](#2-ubuntu-server)
  - [3. Wazuh Deployment](#3-wazuh-deployment)
  - [4. Containerized Web App](#4-containerized-web-app)
- [Verification & Testing](#-verification--testing)
- [Optional Features](#-optional-features)
- [Troubleshooting](#-troubleshooting)
- [Contributing](#-contributing)

---

## 🎯 Overview

This project demonstrates a comprehensive virtual infrastructure deployment with enterprise-grade security practices. Designed for educational purposes, it showcases:

### Key Features

✅ **Network Security** - pfSense firewall with multi-zone architecture  
✅ **Intrusion Detection** - Suricata IDS/IPS monitoring  
✅ **VPN Access** - OpenVPN for secure remote connectivity  
✅ **System Hardening** - Ubuntu Server with SSH keys & iptables  
✅ **Containerization** - Docker-based Flask application  
✅ **Monitoring** - Prometheus & Grafana for metrics  
✅ **SIEM Integration** - Wazuh for centralized security monitoring  

### ⚠️ Disclaimer

> **This project is intended for educational and lab purposes only.**  
> All testing must be conducted in an isolated virtual environment.  
> No real-world systems should be targeted.

---

## 🏗️ Architecture

```
                                    ┌─────────────────┐
                                    │   Internet      │
                                    └────────┬────────┘
                                             │
                                    ┌────────▼────────┐
                                    │   pfSense FW    │
                                    │  (WAN/LAN/DMZ)  │
                                    └────────┬────────┘
                                             │
                    ┌────────────────────────┼────────────────────────┐
                    │                        │                        │
           ┌────────▼────────┐     ┌────────▼────────┐     ┌────────▼────────┐
           │  Ubuntu Server  │     │   Kali Linux    │     │   Windows 10    │
           │  (Wazuh Server) │     │  (Wazuh Agent)  │     │  (Wazuh Agent)  │
           └─────────────────┘     └─────────────────┘     └─────────────────┘
                    │
           ┌────────▼────────┐
           │ Docker Services │
           │ Flask/Prom/Graf │
           └─────────────────┘
```

### Component Breakdown

| Component | Purpose | Network Zone |
|-----------|---------|--------------|
| **pfSense** | Firewall/Router with Suricata IDS | WAN/LAN/DMZ |
| **Ubuntu Server** | Wazuh Manager & hardened services | LAN |
| **Kali Linux** | Security testing & validation | LAN |
| **Windows 10** | Endpoint monitoring | LAN |
| **Docker Stack** | Flask app, Prometheus, Grafana | LAN |

---

## 💻 Prerequisites

### Hardware Requirements

- **RAM**: Minimum 16GB (8GB for basic setup)
- **Storage**: 100GB+ free disk space
- **CPU**: Multi-core processor (4+ cores recommended)

### Software Requirements

<table>
<tr>
<td width="50%">

**Virtualization Platform**
- VirtualBox 7.0+ or
- VMware Workstation/Player

**ISO Images Required**
- [pfSense Latest](https://www.pfsense.org/download/)
- [Ubuntu Server 22.04 LTS](https://ubuntu.com/download/server)
- [Kali Linux](https://www.kali.org/get-kali/)
- [Windows 10](https://www.microsoft.com/software-download/windows10)

</td>
<td width="50%">

**Knowledge Prerequisites**
- Basic Linux command line
- Networking fundamentals (TCP/IP, subnetting)
- Firewall concepts
- Docker basics (helpful)

**Tools on Host Machine**
- SSH client
- Web browser
- Text editor

</td>
</tr>
</table>

---

## 🚀 Installation Guide

## 1. pfSense Firewall

<div align="center">
<img src="https://www.pfsense.org/img/pfsense-logo.svg" alt="pfSense" width="200"/>
</div>

### 1.1 Virtual Machine Setup

**VM Configuration:**
```
Name: pfSense-Gateway
OS Type: FreeBSD (64-bit)
RAM: 2048 MB (minimum)
Disk: 10 GB
```

**Network Adapters:**
| Adapter | Type | Purpose |
|---------|------|---------|
| Adapter 1 | NAT | WAN (Internet) |
| Adapter 2 | Host-only | LAN (Internal) |
| Adapter 3 | Host-only | DMZ (Optional) |

### 1.2 Installation Steps

1. **Mount pfSense ISO** and boot the VM
2. **Follow installation wizard:**
   - Accept defaults
   - Select `Install pfSense`
   - Choose disk and confirm
   - Reboot after installation

3. **Initial Configuration:**
   ```bash
   # Assign interfaces when prompted:
   # WAN -> em0 (or first adapter)
   # LAN -> em1 (or second adapter)
   # DMZ -> em2 (optional)
   ```

4. **Set LAN IP** (example: `192.168.1.1/24`)

### 1.3 Web GUI Configuration

Access: `https://192.168.1.1`  
Default credentials: `admin` / `pfsense`

**Steps:**
1. Complete setup wizard
2. Change admin password (⚠️ **CRITICAL**)
3. Configure time zone and hostname

### 1.4 Firewall Rules

Navigate to **Firewall → Rules**

**LAN Rules:**
```
✅ Allow LAN to WAN (established connections)
❌ Block vulnerable protocols (Telnet, FTP)
✅ Allow HTTPS to web services
```

**DMZ Rules:**
```
❌ Block ALL DMZ → LAN traffic
✅ Allow DMZ → WAN (outbound only)
```

### 1.5 OpenVPN Setup

<div align="center">
<img src="https://openvpn.net/wp-content/uploads/openvpn-logo.svg" alt="OpenVPN" width="150"/>
</div>

**VPN → OpenVPN → Wizards**

1. Select "Local User Access"
2. Create Certificate Authority (CA)
3. Create Server Certificate
4. Configure:
   - Port: `1194`
   - Protocol: `UDP`
   - Tunnel Network: `10.8.0.0/24`

5. **Add firewall rule** on WAN:
   ```
   Protocol: UDP
   Port: 1194
   Source: Any
   Destination: WAN address
   ```

6. **Export client configs** under VPN → OpenVPN → Client Export

### 1.6 Suricata IDS/IPS

<div align="center">
<img src="https://suricata.io/wp-content/uploads/2021/03/Suricata_Logo_Horizontal_Color.svg" alt="Suricata" width="200"/>
</div>

**System → Package Manager → Available Packages**

1. Install **Suricata**
2. Navigate to **Services → Suricata**
3. **Add interface:**
   - Interface: `WAN` and `LAN`
   - Enable IPS mode
   - Block offenders: ✅

4. **Update Rules:**
   - Go to Updates tab
   - Enable ET Open rules
   - Update rules

5. **Enable EVE JSON logging:**
   ```
   Services → Suricata → Interface → EVE Output
   ✅ Enable EVE JSON log
   Format: JSON
   ```

---

## 2. Ubuntu Server

<div align="center">
<img src="https://assets.ubuntu.com/v1/29985a98-ubuntu-logo32.png" alt="Ubuntu" width="150"/>
</div>

### 2.1 VM Creation

```
Name: Ubuntu-Wazuh-Server
OS: Ubuntu 22.04 LTS (64-bit)
RAM: 4096 MB
Disk: 40 GB
Network: Host-only (LAN)
```

### 2.2 Base Installation

1. Install Ubuntu Server with **OpenSSH** enabled
2. Create user with strong password
3. Complete installation and reboot

### 2.3 System Hardening

**Update system:**
```bash
sudo apt update && sudo apt upgrade -y
```

**SSH Key Authentication:**
```bash
# On client machine:
ssh-keygen -t ed25519 -C "your_email@example.com"

# Copy to server:
ssh-copy-id username@192.168.1.10

# Test connection:
ssh username@192.168.1.10
```

**Disable password authentication:**
```bash
sudo nano /etc/ssh/sshd_config

# Change these lines:
PasswordAuthentication no
PermitRootLogin no

sudo systemctl restart sshd
```

### 2.4 iptables Firewall

```bash
# Set default policies
sudo iptables -P INPUT DROP
sudo iptables -P FORWARD DROP
sudo iptables -P OUTPUT ACCEPT

# Allow established connections
sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

# Allow SSH
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Allow loopback
sudo iptables -A INPUT -i lo -j ACCEPT

# Allow Wazuh ports
sudo iptables -A INPUT -p tcp --dport 1514 -j ACCEPT  # Agent communication
sudo iptables -A INPUT -p tcp --dport 1515 -j ACCEPT  # Agent enrollment
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT   # Wazuh dashboard

# Save rules
sudo apt install iptables-persistent -y
sudo netfilter-persistent save
```

### 2.5 Enable Logging

```bash
# Enable rsyslog
sudo systemctl enable rsyslog
sudo systemctl start rsyslog

# Configure log rotation
sudo nano /etc/logrotate.d/rsyslog
```

---

## 3. Wazuh Deployment

<div align="center">
<img src="https://wazuh.com/uploads/2022/05/Logo-blogpost.png" alt="Wazuh" width="200"/>
</div>

### 3.1 Install Wazuh Server (Ubuntu)

**Add Wazuh Repository:**
```bash
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | gpg --dearmor -o /usr/share/keyrings/wazuh.gpg

echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | \
sudo tee /etc/apt/sources.list.d/wazuh.list

sudo apt update
```

**Install All-in-One (Manager + Dashboard):**
```bash
curl -sO https://packages.wazuh.com/4.x/wazuh-install.sh
sudo bash ./wazuh-install.sh -a
```

> 📝 **Save the credentials** displayed after installation!

**Access Dashboard:**
```
URL: https://<UBUNTU_IP>
Default: admin / <generated_password>
```

### 3.2 Install Wazuh Agents

#### 🐧 Ubuntu/Kali Linux

```bash
# Add repository (same as server)
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | gpg --dearmor -o /usr/share/keyrings/wazuh.gpg
echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | \
sudo tee /etc/apt/sources.list.d/wazuh.list

# Install agent
sudo apt update
sudo apt install wazuh-agent

# Configure
sudo nano /var/ossec/etc/ossec.conf
```

**Edit configuration:**
```xml
<client>
  <server>
    <address>192.168.1.10</address>  <!-- Wazuh server IP -->
    <port>1514</port>
    <protocol>tcp</protocol>
  </server>
</client>
```

**Start agent:**
```bash
sudo systemctl daemon-reload
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
```

#### 🪟 Windows 10

1. **Download installer:**
   ```
   https://packages.wazuh.com/4.x/windows/wazuh-agent-4.7.0-1.msi
   ```

2. **Install via PowerShell (Admin):**
   ```powershell
   msiexec.exe /i wazuh-agent-4.7.0-1.msi /q WAZUH_MANAGER="192.168.1.10" WAZUH_AGENT_NAME="Windows10-PC"
   ```

3. **Start service:**
   ```powershell
   NET START WazuhSvc
   ```

#### 🔥 pfSense (FreeBSD)

```bash
# Update packages
pkg update

# Install Wazuh agent
pkg install wazuh-agent

# Copy timezone
cp /etc/localtime /var/ossec/etc/localtime

# Rename config
mv /var/ossec/etc/ossec.conf.sample /var/ossec/etc/ossec.conf

# Edit configuration
ee /var/ossec/etc/ossec.conf
```

**Add to ossec.conf:**
```xml
<client>
  <server>
    <address>192.168.1.10</address>
    <port>1514</port>
    <protocol>tcp</protocol>
  </server>
</client>

<localfile>
  <location>/var/log/suricata/eve.json</location>
  <log_format>json</log_format>
</localfile>
```

**Enable and start:**
```bash
sysrc wazuh_agent_enable="YES"
service wazuh-agent start
```

### 3.3 Verify Agent Connection

**In Wazuh Dashboard:**
1. Navigate to **Server Management → Endpoints Summary**
2. Verify all agents show **Active** status
3. Check last keep-alive timestamp

**Via CLI:**
```bash
sudo /var/ossec/bin/agent_control -l
```

### 3.4 Suricata Integration

Suricata alerts from pfSense are automatically forwarded via the Wazuh agent.

**Verify in dashboard:**
1. Go to **Threat Hunting**
2. Filter: `rule.groups:suricata`
3. View network-based alerts

---

## 4. Containerized Web App

<div align="center">
<img src="https://www.docker.com/wp-content/uploads/2022/03/Moby-logo.png" alt="Docker" width="80"/>
<img src="https://flask.palletsprojects.com/en/2.3.x/_images/flask-horizontal.png" alt="Flask" width="120"/>
<img src="https://prometheus.io/assets/prometheus_logo_grey.svg" alt="Prometheus" width="100"/>
<img src="https://grafana.com/static/img/menu/grafana2.svg" alt="Grafana" width="100"/>
</div>

### 4.1 Project Structure

```
webapp-monitor/
├── app.py
├── requirements.txt
├── Dockerfile
├── prometheus.yml
└── docker-compose.yml
```

### 4.2 Application Files

**`app.py`**
```python
from flask import Flask
from prometheus_client import Counter, generate_latest

app = Flask(__name__)

REQUESTS = Counter('flask_app_requests_total', 'Total HTTP requests to the Flask app')

@app.route('/')
def home():
    REQUESTS.inc()
    return "Hello from secure Flask app!"

@app.route('/metrics')
def metrics():
    return generate_latest()

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

**`requirements.txt`**
```
Flask==3.0.0
prometheus_client==0.19.0
```

**`Dockerfile`**
```dockerfile
FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

EXPOSE 5000

CMD ["python", "app.py"]
```

**`prometheus.yml`**
```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'flask-app'
    static_configs:
      - targets: ['flask-app:5000']
```

**`docker-compose.yml`**
```yaml
version: '3.8'

services:
  flask-app:
    build: .
    container_name: flask-app
    ports:
      - "5000:5000"
    restart: unless-stopped

  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus
    ports:
      - "9090:9090"
    restart: unless-stopped

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    volumes:
      - grafana_data:/var/lib/grafana
    depends_on:
      - prometheus
    restart: unless-stopped

volumes:
  prometheus_data:
  grafana_data:
```

### 4.3 Deploy Stack

```bash
# Navigate to project directory
cd webapp-monitor

# Build and start services
docker compose up -d

# View logs
docker compose logs -f

# Check status
docker compose ps
```

### 4.4 Access Services

| Service | URL | Credentials |
|---------|-----|-------------|
| **Flask App** | http://localhost:5000 | N/A |
| **Prometheus** | http://localhost:9090 | N/A |
| **Grafana** | http://localhost:3000 | admin / admin |

### 4.5 Configure Grafana

1. **Login to Grafana** (http://localhost:3000)
2. **Add Prometheus data source:**
   - Settings → Data Sources → Add data source
   - Select Prometheus
   - URL: `http://prometheus:9090`
   - Click "Save & Test"

3. **Create Dashboard:**
   - Dashboards → New Dashboard → Add visualization
   - Query: `flask_app_requests_total`
   - Set visualization type (Graph, Gauge, etc.)
   - Save dashboard

---

## ✅ Verification & Testing

### Network Tests

**From Kali VM:**
```bash
# Port scan pfSense
nmap -sV 192.168.1.1

# Check for open ports on Ubuntu
nmap -p 22,443,1514,1515 192.168.1.10

# Test web app
curl http://192.168.1.10:5000
```

### Security Checks

- [ ] **SSH**: Only key-based authentication works
- [ ] **Firewall**: Unauthorized ports blocked
- [ ] **Suricata**: Alerts generated from nmap scan
- [ ] **Wazuh**: All agents showing active
- [ ] **VPN**: OpenVPN client connects successfully

### Monitoring Validation

1. **Grafana Dashboard**: Shows real-time metrics
2. **Prometheus**: Targets show "UP" status
3. **Wazuh**: Suricata alerts visible in Threat Hunting
4. **pfSense**: Traffic logs show blocked/allowed connections

---

## 🎨 Optional Features

### Expose Services with Ngrok

<div align="center">
<img src="https://ngrok.com/static/img/ngrok-black.svg" alt="Ngrok" width="150"/>
</div>

**For temporary external access:**

```bash
# Install ngrok
snap install ngrok

# Authenticate
ngrok config add-authtoken <YOUR_TOKEN>

# Expose Flask app
ngrok http 5000

# Expose Grafana
ngrok http 3000
```

> ⚠️ **Security Note**: Only use ngrok in controlled lab environments!

---

## 🔧 Troubleshooting

### Common Issues

<details>
<summary><b>Wazuh Agent Not Connecting</b></summary>

**Check connectivity:**
```bash
# On agent
sudo systemctl status wazuh-agent
sudo tail -f /var/ossec/logs/ossec.log

# Test connection to manager
telnet 192.168.1.10 1514
```

**Verify firewall:**
```bash
# On Wazuh server
sudo iptables -L -n | grep 1514
```
</details>

<details>
<summary><b>Suricata Not Logging</b></summary>

**Check Suricata status:**
```bash
# On pfSense
service suricata status
tail -f /var/log/suricata/suricata_*/eve.json
```

**Verify log path in Wazuh:**
```xml
<localfile>
  <location>/var/log/suricata/eve.json</location>
  <log_format>json</log_format>
</localfile>
```
</details>

<details>
<summary><b>Docker Containers Not Starting</b></summary>

```bash
# Check Docker service
sudo systemctl status docker

# View container logs
docker compose logs flask-app

# Rebuild containers
docker compose down
docker compose build --no-cache
docker compose up -d
```
</details>

### Log Locations

| Component | Log Path |
|-----------|----------|
| pfSense | `/var/log/system.log` |
| Suricata | `/var/log/suricata/eve.json` |
| Wazuh Server | `/var/ossec/logs/ossec.log` |
| Wazuh Agent | `/var/ossec/logs/ossec.log` |
| Ubuntu Auth | `/var/log/auth.log` |

---

## 📚 Additional Resources

- [pfSense Documentation](https://docs.netgate.com/pfsense/en/latest/)
- [Wazuh Documentation](https://documentation.wazuh.com/)
- [Suricata User Guide](https://suricata.readthedocs.io/)
- [Docker Compose Reference](https://docs.docker.com/compose/)
- [Prometheus Documentation](https://prometheus.io/docs/)
- [Grafana Tutorials](https://grafana.com/tutorials/)

---

## 🤝 Contributing

This is an educational project. Contributions, suggestions, and improvements are welcome!

1. Fork the repository
2. Create a feature branch
3. Commit your changes
4. Push to the branch
5. Open a Pull Request

---

## 📄 License

This project is released for educational purposes only. Use at your own risk.

---



---

<div align="center">

**⭐ If you found this helpful, please star the repository! ⭐**

Made with ❤️ for the cybersecurity community

</div>
