# Raspberry Pi Setup (with troubleshooting flow)

## Overview

This document describes the design flow i had while creating the NMS server. This document also contains the troubleshooting of the issues i encounted while seting up the server

---
## 1. Install Required Dependencies

### 1.1 Update system

```
sudo apt update && sudo apt upgrade -y
```

### 1.2 Install required base packages

```
sudo apt install -y \
apache2 mariadb-server mariadb-client \
php php-cli php-common php-curl php-gd php-mbstring php-mysql \
php-snmp php-xml php-zip php-bcmath php-gmp php-intl \
snmp snmpd rrdtool git unzip curl composer \
python3 python3-pip python3-venv
```

### 1.3 Enable Apache modules

```
sudo a2enmod rewrite
sudo a2enmod php*
sudo systemctl restart apache2
```

### 1.4 Enable and start services

```
sudo systemctl enable apache2 mariadb snmpd
sudo systemctl start apache2 mariadb snmpd
```

### 1.5 Secure MariaDB

```
sudo mysql_secure_installation
```
Recommended answers:

```
Switch to unix_socket auth → No
Set root password → Yes
Remove anonymous users → Yes
Disallow remote root login → Yes
Remove test database → Yes
Reload privilege tables → Yes
```

### Local LAN Interface

* Used for administrative access to the LibreNMS web interface.
* Not used for infrastructure monitoring traffic.

This separation ensures predictable routing and management isolation.

---

## Enabled Services

The following services are required for normal operation:

* apache2
* mariadb
* snmpd
* ssh
* cron
* NetworkManager

These services form the core LibreNMS runtime environment.

---

## Disabled Services

Non-essential desktop and consumer services are disabled to reduce system load:

* graphical desktop manager
* avahi/mDNS services
* bluetooth services
* unused cloud initialization services

This reduces background resource usage and improves long-term stability.

---

## LibreNMS Runtime Model

LibreNMS runs as a service-based application:

* Polling performed automatically by LibreNMS services.
* Device discovery and polling executed periodically.
* Alert evaluation performed within LibreNMS alert engine.

The Raspberry Pi operates as a dedicated monitoring node with no additional workloads.

---

## Summary

This configuration transforms the Raspberry Pi into a low-power, always-on network monitoring appliance capable of reliably monitoring ISP infrastructure devices using SNMP.

