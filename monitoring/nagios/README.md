# Nagios Monitoring Project

## Overview
This project demonstrates a Nagios Core monitoring setup with:

- Host monitoring
- Service monitoring (CPU, RAM, Disk, DNS)
- Apache and NRPE integration

## Setup
1. Install Nagios Core
2. Configure Apache
3. Configure hosts in `/etc/nagios/objects/`
4. Start services:
   - systemctl start nagios
   - systemctl start httpd

## Access
http://<server-ip>/nagios

## Notes
Sensitive files such as passwords are excluded.
