 Infrastructure Deployment and Security 
 ## Disclaimer

This project is intended for educational and lab purposes only.
All testing was conducted in an isolated virtual environment.
No real-world systems were targeted or harmed.
This project demonstrates the deployment and securing of a virtual infrastructure using virtualization tools. It includes:

Deployment and configuration of pfSense as a firewall with WAN, LAN, and DMZ interfaces.
Setup of OpenVPN for secure remote access.
Configuration of Suricata as an IDS/IPS.
Deployment of a hardened Ubuntu Server 22.04 LTS VM with SSH key authentication, iptables firewall, and rsyslog.
Containerized deployment of a Flask web application using Docker and Docker Compose.
Real-time monitoring with Prometheus and Grafana.

The entire setup is designed to run in a virtual environment (e.g., VirtualBox or VMware) for testing and learning purposes.
Architecture Overview

pfSense VM: Acts as the main firewall/router with three interfaces:
WAN: Connected to external network (NAT).
LAN: Internal private network.
DMZ: For public-facing servers.

Ubuntu Server VM: Hardened Linux server in the LAN.
Web App Containers: Flask app, Prometheus, and Grafana running via Docker Compose (typically on a host or separate VM).

Traffic is controlled via pfSense rules, with Suricata monitoring for intrusions.
Prerequisites

Virtualization software: VirtualBox or VMware.
ISO files:
pfSense latest ISO.
Ubuntu Server 22.04 LTS ISO.

Host machine with sufficient resources (at least 8GB RAM recommended for running multiple VMs).
Basic knowledge of networking and Linux.

Setup Instructions
1. Deploy pfSense Firewall

Create a new VM:
Name: pfSense
Type: FreeBSD (64-bit)
RAM: At least 2GB
Disk: At least 10GB

Network adapters:
Adapter 1: NAT (WAN)
Adapter 2: Host-only (LAN)
Adapter 3: Host-only (DMZ) – optional for public servers

Mount pfSense ISO and install:
Follow the installer wizard.
Set strong admin password.
Assign interfaces: WAN, LAN, DMZ.

Access web GUI at https://<LAN_IP>  complete setup wizard.
Configure firewall rules:
Block vulnerable protocols on LAN.
Block DMZ → LAN traffic.

Set up OpenVPN remote access:
Use OpenVPN Wizard.
Create CA and server certificates.
Add users and export client configs.
Allow UDP 1194 on WAN.

Install and configure Suricata IDS/IPS:
Install package via System > Package Manager.
Enable on WAN/LAN interfaces in IPS mode.
Update rules and enable logging.


2. Deploy Hardened Ubuntu Server

Create VM for Ubuntu Server 22.04 LTS.
Install with:
Enable OpenSSH during installation.
Set strong user password.

Post-install:Bashsudo apt update && sudo apt upgrade -y
Set up SSH key-based authentication:
On client: ssh-keygen -t ed25519
Copy key: ssh-copy-id user@<server_ip>

Configure iptables firewall:Bashsudo iptables -P INPUT DROP
sudo iptables -P FORWARD DROP
sudo iptables -P OUTPUT ACCEPT
sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
sudo iptables -A INPUT -i lo -j ACCEPT
sudo apt install iptables-persistent -y
sudo netfilter-persistent save
Enable rsyslog for logging.

3. Deploy Containerized Web App with Monitoring
Project Structure
text.
├── app.py
├── requirements.txt
├── Dockerfile
├── prometheus.yml
└── docker-compose.yml
Sample Files
app.py
Pythonfrom flask import Flask
from prometheus_client import Counter, generate_metrics

app = Flask(__name__)

REQUESTS = Counter('flask_app_requests_total', 'Total HTTP requests to the Flask app')

@app.route('/')
def home():
    REQUESTS.inc()
    return "Hello from secure Flask app!"

@app.route('/metrics')
def metrics():
    return generate_metrics()

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
requirements.txt
textFlask
prometheus_client
Dockerfile
dockerfileFROM python:3.9-slim

WORKDIR /app

COPY . /app

RUN pip install -r requirements.txt

EXPOSE 5000

CMD ["python", "app.py"]
prometheus.yml
YAMLglobal:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'flask-app'
    static_configs:
      - targets: ['flask-app:5000']
docker-compose.yml
YAMLversion: '3.8'

services:
  flask-app:
    build: .
    container_name: flask-app
    ports:
      - "5000:5000"

  prometheus:
    image: prom/prometheus
    container_name: prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"

  grafana:
    image: grafana/grafana
    container_name: grafana
    ports:
      - "3000:3000"
    depends_on:
      - prometheus
Run the Stack
Bashdocker compose up -d

Flask app: http://localhost:5000
Prometheus: http://localhost:9090
Grafana: http://localhost:3000 (default login) must be changed!

In Grafana:

Add Prometheus as data source (URL: http://prometheus:9090).
Create dashboard querying metric flask_app_requests_total.

Optional: Expose Locally with Ngrok
For temporary public access:

Sign up at ngrok.com and get auth token.
Install ngrok.
Run: ngrok http 5000 (or other ports).

Verification

Test pfSense rules and Suricata alerts (e.g., run nmap scans).
Verify SSH only allows key auth.
Check Grafana dashboard for real-time metrics.

This project emphasizes security best practices in virtual networking, server hardening, and container monitoring.6,8sRapideExecuted codei want you to also add a part of wazuh deployment for all local machines (kali, ubuntu windows10) and integrate it with suricataVirtual Infrastructure Deployment and Security Project
This project demonstrates the deployment and securing of a virtual infrastructure using virtualization tools. It includes:

Deployment and configuration of pfSense as a firewall with WAN, LAN, and DMZ interfaces.
Setup of OpenVPN for secure remote access.
Configuration of Suricata as an IDS/IPS.
Deployment of a hardened Ubuntu Server 22.04 LTS VM with SSH key authentication, iptables firewall, and rsyslog.
Containerized deployment of a Flask web application using Docker and Docker Compose.
Real-time monitoring with Prometheus and Grafana.
Deployment of Wazuh for security monitoring across endpoints, integrated with Suricata for enhanced threat detection.

The entire setup is designed to run in a virtual environment (e.g., VirtualBox or VMware) for testing and learning purposes.
Architecture Overview

pfSense VM: Acts as the main firewall/router with three interfaces:
WAN: Connected to external network (NAT).
LAN: Internal private network.
DMZ: For public-facing servers.

Ubuntu Server VM: Hardened Linux server in the LAN, also hosting Wazuh server.
Kali VM: Used for testing (e.g., nmap scans), with Wazuh agent.
Windows 10 VM: Endpoint with Wazuh agent.
Web App Containers: Flask app, Prometheus, and Grafana running via Docker Compose (typically on a host or separate VM).
Wazuh: Server on Ubuntu for centralized monitoring; agents on Ubuntu, Kali, Windows 10, and pfSense to collect logs, including Suricata alerts.

Traffic is controlled via pfSense rules, with Suricata monitoring for intrusions. Wazuh collects and analyzes logs from all endpoints, including Suricata's network IDS data.
Prerequisites

Virtualization software: VirtualBox or VMware.
ISO files:
pfSense latest ISO.
Ubuntu Server 22.04 LTS ISO.
Kali Linux ISO.
Windows 10 ISO.

Host machine with sufficient resources (at least 16GB RAM recommended for running multiple VMs).
Basic knowledge of networking and Linux.

Setup Instructions
1. Deploy pfSense Firewall

Create a new VM:
Name: pfSense
Type: FreeBSD (64-bit)
RAM: At least 2GB
Disk: At least 10GB

Network adapters:
Adapter 1: NAT (WAN)
Adapter 2: Host-only (LAN)
Adapter 3: Host-only (DMZ) – optional for public servers

Mount pfSense ISO and install:
Follow the installer wizard.
Set strong admin password.
Assign interfaces: WAN, LAN, DMZ.

Access web GUI at https://<LAN_IP> , complete setup wizard.
Configure firewall rules:
Block vulnerable protocols on LAN.
Block DMZ → LAN traffic.

Set up OpenVPN remote access:
Use OpenVPN Wizard.
Create CA and server certificates.
Add users and export client configs.
Allow UDP 1194 on WAN.

Install and configure Suricata IDS/IPS:
Install package via System > Package Manager.
Enable on WAN/LAN interfaces in IPS mode.
Update rules and enable logging in EVE JSON format.


2. Deploy Hardened Ubuntu Server

Create VM for Ubuntu Server 22.04 LTS.
Install with:
Enable OpenSSH during installation.
Set strong user password.

Post-install:Bashsudo apt update && sudo apt upgrade -y
Set up SSH key-based authentication:
On client: ssh-keygen -t ed25519
Copy key: ssh-copy-id user@<server_ip>

Configure iptables firewall:Bashsudo iptables -P INPUT DROP
sudo iptables -P FORWARD DROP
sudo iptables -P OUTPUT ACCEPT
sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
sudo iptables -A INPUT -i lo -j ACCEPT
sudo apt install iptables-persistent -y
sudo netfilter-persistent save
Enable rsyslog for logging.

3. Deployment of Wazuh for Security Monitoring and Suricata Integration
Wazuh is deployed for endpoint detection, log analysis, and compliance monitoring. The Wazuh server is installed on the Ubuntu Server VM, with agents on all local machines (Ubuntu, Kali, Windows 10, and pfSense). Suricata integration allows Wazuh to process network IDS alerts from pfSense.
3.1 Install Wazuh Server on Ubuntu Server VM
Follow the official step-by-step guide:

Add Wazuh repository:Bashcurl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | sudo apt-key add -
echo "deb https://packages.wazuh.com/4.x/apt/ stable main" | sudo tee -a /etc/apt/sources.list.d/wazuh.list
sudo apt update
Install Wazuh manager, Filebeat, and Kibana (for dashboard):Bashsudo apt install wazuh-manager
sudo apt install filebeat
sudo apt install kibana
sudo apt install elasticsearch
Configure and start services:
Set up Elasticsearch, Kibana, and Filebeat as per docs.
Access Wazuh dashboard at https://<UBUNTU_IP>:5601 


3.2 Install Wazuh Agents

On Ubuntu and Kali (Debian-based):
Add repository (same as server).
Install agent:Bashsudo apt install wazuh-agent
Configure /var/ossec/etc/ossec.conf with manager IP:XML<client>
  <server>
    <address><WAZUH_MANAGER_IP></address>
  </server>
</client>
Restart agent: sudo systemctl restart wazuh-agent.

On Windows 10:
Download the Wazuh agent MSI from https://packages.wazuh.com/4.x/windows/wazuh-agent-4.x.x-x.msi.
Run the installer via GUI or CLI:cmdmsiexec.exe /i wazuh-agent-4.x.x-x.msi /q WAZUH_MANAGER="<WAZUH_MANAGER_IP>" WAZUH_REGISTRATION_SERVER="<WAZUH_MANAGER_IP>"
Start the service: net start WazuhSvc.

On pfSense (for Suricata logs):
Update packages: pkg update.
Install: pkg install wazuh-agent.
Configure:
Copy timezone: cp /etc/localtime /var/ossec/etc.
Rename config: cp /var/ossec/etc/ossec.conf{.sample,}.
Edit /var/ossec/etc/ossec.conf: Set manager IP in <address>, change protocol to TCP.
Add local files to monitor, e.g., pfSense logs.

Enable and start: sysrc wazuh_agent_enable="YES" and service wazuh-agent start.

3.3 Integrate Wazuh with Suricata (on pfSense)

Ensure Suricata on pfSense is configured for EVE JSON logging (already set in pfSense setup).
On pfSense Wazuh agent, add to /var/ossec/etc/ossec.conf:XML<localfile>
  <log_format>json</log_format>
  <location>/var/log/suricata/eve.json</location>  
</localfile>
Restart Wazuh agent: service wazuh-agent restart.
Wazuh will parse Suricata logs and generate alerts viewable in the dashboard under Threat Hunting (filter: rule.groups:suricata).

4. Deploy Containerized Web App with Monitoring
Project Structure
text.
├── app.py
├── requirements.txt
├── Dockerfile
├── prometheus.yml
└── docker-compose.yml
Sample Files
app.py
Pythonfrom flask import Flask
from prometheus_client import Counter, generate_metrics

app = Flask(__name__)

REQUESTS = Counter('flask_app_requests_total', 'Total HTTP requests to the Flask app')

@app.route('/')
def home():
    REQUESTS.inc()
    return "Hello from secure Flask app!"

@app.route('/metrics')
def metrics():
    return generate_metrics()

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
requirements.txt
textFlask
prometheus_client
Dockerfile
dockerfileFROM python:3.9-slim

WORKDIR /app

COPY . /app

RUN pip install -r requirements.txt

EXPOSE 5000

CMD ["python", "app.py"]
prometheus.yml
YAMLglobal:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'flask-app'
    static_configs:
      - targets: ['flask-app:5000']
docker-compose.yml
YAMLversion: '3.8'

services:
  flask-app:
    build: .
    container_name: flask-app
    ports:
      - "5000:5000"

  prometheus:
    image: prom/prometheus
    container_name: prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"

  grafana:
    image: grafana/grafana
    container_name: grafana
    ports:
      - "3000:3000"
    depends_on:
      - prometheus
Run the Stack
Bashdocker compose up -d

Flask app: http://localhost:5000
Prometheus: http://localhost:9090
Grafana: http://localhost:3000 

In Grafana:

Add Prometheus as data source (URL: http://prometheus:9090).
Create dashboard querying metric flask_app_requests_total.

Optional: Expose Locally with Ngrok
For temporary public access:

Sign up at ngrok.com and get auth token.
Install ngrok.
Run: ngrok http 5000 (or other ports).

Verification

Test pfSense rules and Suricata alerts (e.g., run nmap scans).
Verify SSH only allows key auth.
Check Grafana dashboard for real-time metrics.
In Wazuh dashboard, verify agents are connected and Suricata alerts are processed (e.g., trigger a scan and check for alerts).

This project emphasizes security best practices in virtual networking, server hardening, container monitoring, and endpoint security with Wazuh.9,6sExpert29 sourcesgive me a ready file to copy paste readmeMarkdown# Virtual Infrastructure Deployment and Security Project

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

## Prerequisites

- Virtualization platform (VirtualBox or VMware).
- ISO images:
  - pfSense (latest)
  - Ubuntu Server 22.04 LTS
  - Kali Linux
  - Windows 10
- Host machine with sufficient resources (≥ 8 GB RAM recommended).
- Basic networking and Linux knowledge.

## Setup Instructions

### 1. Deploy pfSense Firewall

1. Create VM:
   - OS: FreeBSD (64-bit)
   - RAM: ≥ 2 GB
   - Disk: ≥ 10 GB

2. Network adapters:
   - Adapter 1 → NAT (WAN)
   - Adapter 2 → Host-only (LAN)
   - Adapter 3 → Host-only (DMZ, optional)

3. Install pfSense from ISO, assign interfaces, set admin password.

4. Access web GUI: `https://<LAN_IP>`

5. Configure firewall rules:
   - Block vulnerable protocols on LAN.
   - Block all traffic from DMZ to LAN.

6. Set up OpenVPN (VPN → OpenVPN → Wizards):
   - Create CA and server certificate.
   - Configure users and export client configs.
   - Allow UDP 1194 on WAN.

7. Install & configure Suricata:
   - System → Package Manager → Install Suricata.
   - Enable on WAN/LAN in IPS mode.
   - Enable EVE JSON logging.

### 2. Deploy Hardened Ubuntu Server (Wazuh Manager Host)

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
3.1 Install Wazuh Manager on Ubuntu Server
Bashcurl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | gpg --dearmor -o /usr/share/keyrings/wazuh.gpg
echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | tee /etc/apt/sources.list.d/wazuh.list
sudo apt update
sudo apt install wazuh-manager
# Install the full Wazuh stack (Filebeat, OpenSearch, Wazuh Dashboard) using the assisted installer:
curl -sO https://packages.wazuh.com/4.x/wazuh-install.sh && sudo bash ./wazuh-install.sh -a
Access dashboard at https://<UBUNTU_IP> (default credentials)
3.2 Install Wazuh Agents
On Ubuntu/Kali:
Bashsudo apt install wazuh-agent
Edit /var/ossec/etc/ossec.conf → set manager IP, then:
Bashsudo systemctl restart wazuh-agent
On Windows 10:

Download MSI from https://packages.wazuh.com/4.x/windows/wazuh-agent-latest.msi
Install with:cmdmsiexec.exe /i wazuh-agent-latest.msi /q WAZUH_MANAGER="<UBUNTU_IP>"

On pfSense (for Suricata logs):
Shellpkg update
pkg install wazuh-agent
cp /etc/localtime /var/ossec/etc/localtime
mv /var/ossec/etc/ossec.conf.sample /var/ossec/etc/ossec.conf
Edit /var/ossec/etc/ossec.conf → set manager IP and add:
XML<localfile>
  <location>/var/log/suricata/eve.json</location>
  <log_format>json</log_format>
</localfile>
Enable and start:
Shellsysrc wazuh_agent_enable="YES"
service wazuh-agent start
Suricata alerts will now appear in the Wazuh dashboard.
4. Deploy Containerized Web App with Monitoring
Project Structure
text.
├── app.py
├── requirements.txt
├── Dockerfile
├── prometheus.yml
└── docker-compose.yml
Key Files
app.py
Pythonfrom flask import Flask
from prometheus_client import Counter, generate_metrics

app = Flask(__name__)
REQUESTS = Counter('flask_app_requests_total', 'Total HTTP requests')

@app.route('/')
def home():
    REQUESTS.inc()
    return "Hello from secure Flask app!"

@app.route('/metrics')
def metrics():
    return generate_metrics()

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
requirements.txt
textFlask
prometheus_client
Dockerfile
dockerfileFROM python:3.9-slim
WORKDIR /app
COPY . /app
RUN pip install -r requirements.txt
EXPOSE 5000
CMD ["python", "app.py"]
prometheus.yml
YAMLglobal:
  scrape_interval: 15s
scrape_configs:
  - job_name: 'flask-app'
    static_configs:
      - targets: ['flask-app:5000']
docker-compose.yml
YAMLversion: '3.8'
services:
  flask-app:
    build: .
    ports:
      - "5000:5000"
  prometheus:
    image: prom/prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"
  grafana:
    image: grafana/grafana
    ports:
      - "3000:3000"
    depends_on:
      - prometheus
Run:
Bashdocker compose up -d
Access:

App: http://localhost:5000
Prometheus: http://localhost:9090
Grafana: http://localhost:3000 (admin/admin)

In Grafana → Add Prometheus data source → Create dashboard with metric flask_app_requests_total.
Optional: Temporary Public Exposure with Ngrok
Bash# Install ngrok and authenticate
ngrok http 5000   # or 3000, 9090 as needed
Verification

Test firewall rules and Suricata (run nmap scans from Kali → check alerts).
Confirm SSH key-only access.
View real-time metrics in Grafana.
Check Wazuh dashboard for agent connectivity and Suricata alerts.

This project implements security best practices across networking, server hardening, containerization, application monitoring, and centralized endpoint/security event management.
