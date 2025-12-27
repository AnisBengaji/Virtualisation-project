🛡️ Virtual Infrastructure Deployment and Security Project
⚠️ Disclaimer

This project is intended strictly for educational and lab purposes.
All testing was conducted in an isolated virtual environment.
No real-world systems were targeted or harmed.

📌 Project Overview

This project demonstrates the deployment and securing of a virtualized infrastructure using industry-standard security tools and best practices.

It includes:

Deployment and configuration of pfSense as a firewall with WAN, LAN, and DMZ interfaces

Setup of OpenVPN for secure remote access

Configuration of Suricata as an IDS/IPS

Deployment of a hardened Ubuntu Server 22.04 LTS with:

SSH key-based authentication

iptables firewall

rsyslog logging

Containerized deployment of a Flask web application using Docker & Docker Compose

Real-time monitoring using Prometheus and Grafana

Centralized endpoint and security monitoring using Wazuh, integrated with Suricata alerts

The entire setup runs in a virtual lab environment (VirtualBox or VMware) for learning, testing, and security experimentation.

 Architecture Overview

pfSense VM

Acts as the main firewall/router

Interfaces:

WAN (NAT)

LAN (Host-Only)

DMZ (Host-Only, optional)

Ubuntu Server VM

Hardened Linux server

Hosts the Wazuh Manager

Kali Linux VM

Used for security testing (e.g., Nmap scans)

Runs a Wazuh agent

Windows 10 VM

Endpoint system

Runs a Wazuh agent

Docker Containers (host or separate VM)

Flask web application

Prometheus

Grafana

Wazuh

Centralized security manager on Ubuntu Server

Agents on all VMs, including pfSense

Collects endpoint logs and Suricata IDS alerts

Traffic is controlled using pfSense firewall rules, while Suricata inspects network traffic.
Wazuh aggregates and analyzes logs from all endpoints for threat detection.

 Prerequisites

Virtualization software:

VirtualBox or VMware

ISO Images:

pfSense (latest)

Ubuntu Server 22.04 LTS

Kali Linux

Windows 10

Host system with sufficient resources:

≥ 16 GB RAM recommended

Basic knowledge of:

Networking

Linux

Virtualization concepts

⚙️ Setup Instructions
🔥 1. Deploy pfSense Firewall

VM Configuration

OS: FreeBSD (64-bit)

RAM: ≥ 2 GB

Disk: ≥ 10 GB

Network Adapters

Adapter 1 → NAT (WAN)

Adapter 2 → Host-only (LAN)

Adapter 3 → Host-only (DMZ – optional)

Installation Steps

Install pfSense from ISO

Assign WAN, LAN, and DMZ interfaces

Set a strong admin password

Access the web interface:

https://<LAN_IP>


(Default often 192.168.1.1)

Firewall Rules

Block vulnerable protocols on LAN

Block DMZ → LAN traffic entirely

🔐 OpenVPN Configuration

Use VPN → OpenVPN → Wizards

Create:

Certificate Authority

Server certificate

User credentials

Export client configurations

Allow UDP 1194 on WAN

🚨 Suricata IDS/IPS

Install via System → Package Manager

Enable on:

WAN

LAN

Enable IPS mode

Enable EVE JSON logging

🖥️ 2. Deploy Hardened Ubuntu Server (Wazuh Manager)

Installation

Ubuntu Server 22.04 LTS

Enable OpenSSH

System Update
sudo apt update && sudo apt upgrade -y

SSH & Firewall Hardening

SSH key-based authentication

iptables firewall:

sudo iptables -P INPUT DROP
sudo iptables -P FORWARD DROP
sudo iptables -P OUTPUT ACCEPT
sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
sudo iptables -A INPUT -i lo -j ACCEPT
sudo apt install iptables-persistent -y
sudo netfilter-persistent save

🛡️ 3. Wazuh Deployment & Suricata Integration
Wazuh Manager Installation

Use the assisted installer:

curl -sO https://packages.wazuh.com/4.x/wazuh-install.sh
sudo bash ./wazuh-install.sh -a


Access dashboard:

https://<UBUNTU_IP>


Change default credentials immediately

Wazuh Agents

Installed on:

Ubuntu

Kali Linux

Windows 10

pfSense

Suricata Integration

Configure pfSense Wazuh agent to monitor:

/var/log/suricata/eve.json


Suricata alerts appear in the Wazuh Dashboard

🐳 4. Containerized Web App & Monitoring
Stack

Flask application

Prometheus

Grafana

Run Containers
docker compose up -d

Access Services

Flask App: http://localhost:5000

Prometheus: http://localhost:9090

Grafana: http://localhost:3000

Default: admin / admin (change password)

Grafana Setup

Add Prometheus data source

Create dashboard for:

flask_app_requests_total

🌐 Optional: Temporary Public Exposure (Ngrok)
ngrok http 5000


(Can also expose 3000 or 9090)

✅ Verification Checklist

Run Nmap scans from Kali

Verify Suricata & Wazuh alerts

Confirm SSH key-only access

Validate firewall segmentation

Monitor metrics in Grafana

Ensure all Wazuh agents are connected

 Conclusion

This project implements security best practices across:

Network segmentation & firewalling

IDS/IPS monitoring

Endpoint detection & response

Server hardening

Containerized applications

Centralized logging and monitoring

It serves as a complete cybersecurity lab for learning defensive security, SOC workflows, and infrastructure hardening.
