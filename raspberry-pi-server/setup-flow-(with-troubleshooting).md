# Raspberry Pi Setup (with troubleshooting flow)

## Overview

This document describes the design flow i had while creating the NMS server. This document also contains the troubleshooting of the issues i encounted while seting up the server

## 1. Install Required Dependencies

### 1.1 Update system

```
sudo apt update && sudo apt upgrade -y
```
---
### 1.2 Install required base packages

```
sudo apt install -y \
apache2 mariadb-server mariadb-client \
php php-cli php-common php-curl php-gd php-mbstring php-mysql \
php-snmp php-xml php-zip php-bcmath php-gmp php-intl \
snmp snmpd rrdtool git unzip curl composer \
python3 python3-pip python3-venv
```
---
### 1.3 Enable Apache modules

```
sudo a2enmod rewrite
sudo a2enmod php*
sudo systemctl restart apache2
```
---
### 1.4 Enable and start services

```
sudo systemctl enable apache2 mariadb snmpd
sudo systemctl start apache2 mariadb snmpd
```
---
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
- On newer Debian / Raspberry Pi OS versions, mysql_secure_installation is not installed separately or MariaDB is configured secure-by-default using Unix socket authentication. What to do instead (correct approach):
* Confirm MariaDB is installed and running
```
systemctl status mariadb
```
You should see active (running).
* Log in to MariaDB as root (no password needed)
```
sudo mariadb
```
or
```
sudo mysql
```
If this works → your DB is may be using secure socket authentication (good).
* Manually apply the important security checks
Inside the MariaDB prompt, run:
```
SELECT user, host, plugin FROM mysql.user;
```
You should see:
```
root@localhost
plugin = unix_socket
```
if plugin = mysql_native_password, then
Root login uses a password-based authentication
Instead of OS-level (sudo/unix socket) authentication. 
This is perfectly acceptable and works fine with LibreNMS.
* For my case plugin = mysql_native_password, so i continues to 1.6

---
### 1.6 Create LibreNMS database

```
sudo mysql -u root -p
```
Inside MariaDB:
```
CREATE DATABASE librenms CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'librenms'@'localhost' IDENTIFIED BY 'StrongPassword';
GRANT ALL PRIVILEGES ON librenms.* TO 'librenms'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```
---
## 2. Install LibreNMS

### 2.1 Create LibreNMS user
```
sudo useradd librenms -d /opt/librenms -M -r -s /bin/bash
```
---
### 2.2 Clone LibreNMS
```
cd /opt
sudo git clone https://github.com/librenms/librenms.git
```
---
### 2.3 Set permissions
```
sudo chown -R librenms:librenms /opt/librenms
sudo chmod 771 /opt/librenms
sudo setfacl -R -m u:www-data:rwx /opt/librenms
sudo setfacl -dR -m u:www-data:rwx /opt/librenms
```
---
### 2.4 Install PHP dependencies (Composer)
```
sudo su - librenms
./scripts/composer_wrapper.php install --no-dev
exit
```
---
### 2.5 Set PHP timezone
```
sudo nano /etc/php/*/apache2/php.ini
```
Find & edit:
```
date.timezone = Asia/Kolkata
```
Restart Apache:
```
sudo systemctl restart apache2
```
---
### 2.6 Configure Apache Virtual Host
```
sudo nano /etc/apache2/sites-available/librenms.conf
```







## Summary

This configuration transforms the Raspberry Pi into a low-power, always-on network monitoring appliance capable of reliably monitoring ISP infrastructure devices using SNMP.

