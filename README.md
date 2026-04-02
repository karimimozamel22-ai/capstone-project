# 🚀 DevOps Capstone Project

## 📋 Table of Contents
- [Overview](#overview)
- [Infrastructure](#infrastructure)
- [Ansible](#ansible)
- [DNS](#dns)
- [Web Servers & Load Balancer](#web-servers--load-balancer)
- [Database](#database)
- [Monitoring](#monitoring)
- [Docker](#docker)
- [Kubernetes](#kubernetes)
- [Security](#security)
- [How to Run](#how-to-run)
- [Repository Structure](#repository-structure)

---

## Overview

This capstone project demonstrates a complete, production-grade DevOps infrastructure built from scratch using industry-standard tools and best practices. The project was completed over 3 sprints across 3 weeks and covers the full DevOps lifecycle — from infrastructure provisioning and configuration management, to containerization and container orchestration.

### Key Goals
- Automate all server configurations using **Ansible**
- Deploy a highly available web application behind a **Load Balancer**
- Store and manage data using **MariaDB**
- Monitor the entire infrastructure using **Nagios**
- Containerize applications using **Docker**
- Orchestrate containers at scale using **Kubernetes**
- Store all code and configurations in **GitHub**

---

## Infrastructure

All servers were provisioned using VirtualBox and run on CentOS/Rocky Linux (ARM64).

| VM Name | Role | CPU | RAM | Storage | OS |
|---|---|---|---|---|---|
| prdx-dns101 | DNS Server | 1 | 1G | 5G | CentOS 9 |
| prdx-db101 | Database Server (MariaDB) | 1 | 1G | 5G | CentOS 9 |
| prdx-webserver101 | Apache Web Server 1 | 1 | 1G | 5G | CentOS 7 |
| prdx-webserver102 | Apache Web Server 2 | 1 | 1G | 5G | CentOS 7 |
| prdx-webserver103 | Apache Web Server 3 | 1 | 1G | 5G | CentOS 7 |
| prdx-haproxy101 | Load Balancer (HAProxy) | 1 | 1G | 5G | CentOS 7 |
| prdx-nagios101 | Monitoring Server (Nagios) | 1 | 1G | 5G | CentOS 7 |
| prdx-ansible101 | Ansible Control Node | 1 | 1G | 5G | CentOS 9 |
| prdx-dprimary101 | Docker Host | 4 | 4G | 15G | CentOS 9 |
| prdx-kube101 | Kubernetes Cluster (Kind) | 1 | 3G | 5G | CentOS 9 |

---

## Ansible

**Server:** `prdx-ansible101`

Ansible is used as the central configuration management tool for the entire infrastructure. All servers are managed from the Ansible control node without requiring passwords, using SSH key-based authentication.

### What Ansible Manages
- Common package installation on all servers
- Security hardening across all servers
- Apache web server deployment
- User management and sudo privileges
- DNS client configuration

### Common Packages Installed on All Servers
- `bind-utils` — DNS lookup tools
- `man` and `man-pages` — Manual pages
- `mlocate` — File search utility
- `sysstat` — System performance tools

### How to Use
```bash
# Test connectivity to all servers
ansible all -m ping

# Install common packages on all servers
ansible-playbook playbooks/packages.yml

# Apply security settings to all servers
ansible-playbook playbooks/security.yml

# Deploy Apache web server
ansible-playbook playbooks/apache.yml

# Manage users
ansible-playbook playbooks/user.yml
```

### Verify Ansible is Working
```bash
# All servers should return "pong"
ansible all -m ping

# Check DNS config on all servers
ansible all -m shell -a "cat /etc/resolv.conf"
```

---

## DNS

**Server:** `prdx-dns101`

A central DNS server handles name resolution for all servers in the environment. Every server is registered and resolvable by hostname, eliminating the need for manual `/etc/hosts` entries.

### Verify DNS is Working
```bash
# Look up any server by name
nslookup prdx-webserver101 prdx-dns101
nslookup prdx-db101 prdx-dns101
nslookup prdx-haproxy101 prdx-dns101

# Check all servers are using the DNS server
ansible all -m shell -a "cat /etc/resolv.conf"
```

---

## Web Servers & Load Balancer

**Web Servers:** `prdx-webserver101`, `prdx-webserver102`, `prdx-webserver103`
**Load Balancer:** `prdx-haproxy101`

Three Apache web servers sit behind an HAProxy load balancer. Traffic is distributed evenly across all three servers using round-robin load balancing. Each web server serves the same content but displays a unique identifier so you can verify the load balancer is distributing requests.

### HAProxy Configuration
- **Frontend:** Listens on port `80`
- **Backend:** Round-robin across all 3 web servers
- **Health checks:** Enabled on all backend servers
- **Stats page:** Available at `http://prdx-haproxy101:8404/stats`
- **Stats login:** `root` / `StrongPass123!`

### Backend Servers
| Server | IP Address | Port |
|---|---|---|
| prdx-webserver101 | 192.168.1.147 | 80 |
| prdx-webserver102 | 192.168.1.148 | 80 |
| prdx-webserver103 | 192.168.1.149 | 80 |

### Verify Load Balancer is Working
```bash
# Open browser and go to the load balancer IP
http://192.168.1.x

# Refresh the page multiple times
# You should see the request being served by different web servers
```

---

## Database

**Server:** `prdx-db101`

MariaDB is used as the relational database management system. A sample group database was created containing team member information including names, states, and assigned servers.

### Sample Database: group_db

```sql
-- Connect to MariaDB
mysql -u root -p

-- Show all databases
SHOW DATABASES;

-- Use the group database
USE group_db;

-- View all members
SELECT * FROM members;
```

### Sample Data

| id | name | state | server_name |
|---|---|---|---|
| 1 | Ahmad | Massachusetts | server1 |
| 2 | Roman | AFG | server2 |
| 3 | Ali | Texas | server3 |
| 4 | Sara | California | server4 |
| 5 | John | New York | server5 |

### Restore the Database
```bash
mysql -u root -p group_db < database/group_db.sql
```

---

## Monitoring

**Server:** `prdx-nagios101`

Nagios provides full infrastructure monitoring with a web-based GUI. All servers and critical services are monitored in real time. Alerts are triggered when any service goes down.

### What is Monitored
- All servers: CPU usage, RAM usage, and disk space
- Apache service on all 3 web servers
- DNS service on the DNS server
- Network connectivity (ping) for all servers
- MariaDB on the database server

### Access Nagios Dashboard
```
http://prdx-nagios101/nagios
```

### Verify Monitoring is Working
1. Open the Nagios dashboard in your browser
2. All hosts and services should show green
3. To test alerting: shut down a service on any server and watch Nagios catch it

```bash
# Example: stop Apache on webserver1
sudo systemctl stop httpd

# Check Nagios dashboard — Apache should turn red within 1-2 minutes

# Bring it back up
sudo systemctl start httpd
```

---

## Docker

**Server:** `prdx-dprimary101`

A custom Docker image is built that runs an Apache web application with PHP and MySQL support. The image downloads and deploys a sample web application automatically on startup.

### What the Dockerfile Does
- Installs `httpd`, `php`, `mysql`, `php-mysql`
- Downloads and extracts the lab web application from AWS S3
- Configures Apache to serve the application
- Sets correct file permissions

### Build and Run
```bash
# Build the Docker image
docker build -t my-web-app .

# Run the container
docker run -d -p 80:80 my-web-app

# Verify it is running
docker ps

# View container logs
docker logs <container-id>
```

### Access the Application
```
http://prdx-dprimary101
```

---

## Kubernetes

**Server:** `prdx-kube101`

A production-grade Kubernetes cluster is provisioned using Kind (Kubernetes in Docker) on CentOS 9 ARM64. The cluster includes a browser-based dashboard for non-technical users and an NGINX ingress controller for HTTP routing.

### Cluster Details
| Component | Details |
|---|---|
| Cluster type | Kind (Kubernetes in Docker) |
| Architecture | ARM64 |
| Kubernetes version | v1.27.3 |
| Monitoring | Kubernetes Dashboard v2.7.0 |
| Ingress | NGINX Ingress Controller v1.9.6 |

### Create the Cluster
```bash
kind create cluster --name my-cluster --config kind-config.yaml
```

### Install the Kubernetes Dashboard
```bash
# Deploy the dashboard
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml

# Create admin user
kubectl apply -f dashboard-admin-user.yaml

# Get login token
kubectl -n kubernetes-dashboard create token admin-user --duration=24h
```

### Install NGINX Ingress Controller
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.9.6/deploy/static/provider/kind/deploy.yaml
```

### Deploy Sample Application
```bash
# Deploy nginx sample app
kubectl apply -f sample-app.yaml

# Add local DNS
echo "127.0.0.1 hello.local" | sudo tee -a /etc/hosts

# Test the app
curl http://hello.local
```

### Access the Dashboard
```bash
# Start proxy on the server
kubectl proxy --port=8001 &

# Open SSH tunnel from your laptop
ssh -N -L 9090:127.0.0.1:8001 root@SERVER_IP

# Open in browser
http://localhost:9090/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
```

### Verify Everything is Running
```bash
# Check all pods
kubectl get pods -A

# Check nodes
kubectl get nodes

# Check ingress rules
kubectl get ingress

# Check services
kubectl get svc -A
```

---

## Security

The following security measures are applied to all servers via Ansible:

| Security Measure | Status |
|---|---|
| Root SSH login disabled | Applied to all servers |
| Firewall enabled | Applied to all servers |
| SELinux disabled | Applied to all servers |
| SSH key-based authentication | Ansible user on all servers |
| Kubernetes Dashboard token auth | Enabled |
| HAProxy stats password protected | Enabled |

---

## How to Run

### Prerequisites
- VirtualBox installed
- All VMs provisioned and running
- Ansible installed on `prdx-ansible101`
- SSH keys distributed to all servers

### Step 1 — Configure All Servers with Ansible
```bash
ssh ansible@prdx-ansible101
cd ansible-project101
ansible all -m ping
ansible-playbook playbooks/packages.yml
ansible-playbook playbooks/security.yml
ansible-playbook playbooks/apache.yml
```

### Step 2 — Verify Web + Load Balancer
```bash
# Open browser and refresh several times to see load balancing
http://prdx-haproxy101
```

### Step 3 — Verify Database
```bash
ssh root@prdx-db101
mysql -u root -p
USE group_db;
SELECT * FROM members;
```

### Step 4 — Verify Monitoring
```bash
# Open browser — all hosts and services should be green
http://prdx-nagios101/nagios
```

### Step 5 — Run Docker App
```bash
ssh ansible@prdx-dprimary101
cd docker-lab
docker build -t my-web-app .
docker run -d -p 80:80 my-web-app
```

### Step 6 — Access Kubernetes Dashboard
```bash
ssh root@prdx-kube101
kubectl proxy --port=8001 &
kubectl -n kubernetes-dashboard create token admin-user --duration=24h
# Open SSH tunnel from laptop then access dashboard in browser
```

---

## Repository Structure

```
capstone-project/
├── README.md
├── ansible/
│   └── ansible-project101/
│       ├── ansible.cfg
│       ├── inventory
│       ├── README.md
│       └── playbooks/
│           ├── apache.yml
│           ├── packages.yml
│           ├── security.yml
│           └── user.yml
├── kubernetes/
│   ├── kind-config.yaml
│   ├── dashboard-admin-user.yaml
│   ├── sample-app.yaml
│   └── README.md
├── monitoring/
│   └── nagios/
│       └── nagios-config/
├── load-balancing/
│   └── haproxy.cfg
├── database/
│   ├── my.cnf
│   ├── my.cnf.d/
│   └── group_db.sql
└── docker/
    └── Dockerfile
```

---

## Project Timeline

| Sprint | Duration | Deliverables |
|---|---|---|
| Sprint 1 | Week 1 | Create servers in VirtualBox, build Ansible VM, configure DNS |
| Sprint 2 | Week 2 | Continue building environment, reach 70% completion |
| Sprint 3 | Week 3 | Complete full environment, prepare presentation |
| Presentation | Day 21 | Live demo of full infrastructure |

---

## Quick Reference Commands

| Task | Command |
|---|---|
| Test Ansible connectivity | `ansible all -m ping` |
| Check all Kubernetes pods | `kubectl get pods -A` |
| Check Kubernetes nodes | `kubectl get nodes` |
| View MariaDB databases | `mysql -u root -p -e "SHOW DATABASES;"` |
| Check HAProxy stats | `http://prdx-haproxy101:8404/stats` |
| Check Docker containers | `docker ps` |
| Get Kubernetes dashboard token | `kubectl -n kubernetes-dashboard create token admin-user --duration=24h` |
| Restart kubectl proxy | `kubectl proxy --port=8001 &` |

---

## Author

**Mozamel Karimi**
GitHub: [@karimimozamel22-ai](https://github.com/karimimozamel22-ai)
Repository: [capstone-project](https://github.com/karimimozamel22-ai/capstone-project)
