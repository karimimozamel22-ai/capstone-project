# DevOps Capstone Project

## Overview
This capstone project demonstrates a full production-grade DevOps infrastructure built from scratch using industry-standard tools and best practices. The environment includes automated configuration management, web serving with load balancing, database management, monitoring, containerization, and container orchestration — all provisioned and managed through code.

## Repository Structure
```
capstone-project/
├── ansible/
├── kubernetes/
├── monitoring/
├── load-balancing/
├── database/
└── docker/
```

## Infrastructure Overview

| VM Name | Role | OS |
|---|---|---|
| prdx-dns101 | DNS Server | CentOS 9 |
| prdx-db101 | Database (MariaDB) | CentOS 9 |
| prdx-webserver101 | Web Server 1 (Apache) | CentOS 7 |
| prdx-webserver102 | Web Server 2 (Apache) | CentOS 7 |
| prdx-webserver103 | Web Server 3 (Apache) | CentOS 7 |
| prdx-haproxy101 | Load Balancer (HAProxy) | CentOS 7 |
| prdx-nagios101 | Monitoring (Nagios) | CentOS 7 |
| prdx-ansible101 | Ansible Control Node | CentOS 9 |
| prdx-dprimary101 | Docker Host | CentOS 9 |
| prdx-kube101 | Kubernetes Cluster | CentOS 9 |

## Components
- **Ansible** — Automated configuration management for all servers
- **DNS** — All servers resolve hostnames through central DNS
- **Web + Load Balancer** — 3 Apache servers behind HAProxy round-robin
- **Database** — MariaDB with sample group database
- **Monitoring** — Nagios monitoring all servers and services
- **Docker** — Custom image running Apache web application
- **Kubernetes** — Kind cluster with Dashboard and NGINX Ingress

## GitHub Repository
https://github.com/karimimozamel22-ai/capstone-project
