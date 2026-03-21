# Ansible Automation Project

## Overview

This project demonstrates infrastructure automation using Ansible.  
It is designed to manage and configure multiple servers from a central control node (master) in a consistent and automated way.

All configurations are handled through Ansible playbooks and stored in this GitHub repository.

---

## Features

- Passwordless SSH access between master and client servers  
- Automated creation of the `ansible` user with sudo privileges  
- Security hardening:
  - Root login disabled
  - Firewall enabled
  - SELinux disabled  
- Automated installation of common packages across all servers  
- Centralized configuration management using Ansible  
- Fully version-controlled in GitHub  

---

## Architecture

- **Ansible Control Node (Master):**  
  Runs Ansible playbooks and manages all client servers.

- **Client Servers:**  
  Configured remotely by Ansible through playbooks.

---

## Project Structure
ansible-project101/
├── inventory/
├── playbooks/
│   ├── user.yml
│   ├── security.yml
│   ├── packages.yml
├── README.md

## Author

- Name: Ahmad Karimi  
- GitHub: https://github.com/karimimozamel22-ai
