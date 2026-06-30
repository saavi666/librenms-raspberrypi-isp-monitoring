# LibreNMS Setup

## Overview

This document contains the commands and configuration steps for using the Raspberry Pi as a dedicated LibreNMS monitoring server.

---

## System Update

```bash
sudo apt update
sudo apt full-upgrade -y
```

## Install Required Packages

```bash
sudo apt install acl ca-certificates curl fping git lsb-release mariadb-client mariadb-server mtr-tiny nginx-full nmap php-cli php-curl php-fpm php-gd php-gmp php-mbstring php-mysql php-snmp php-xml php-zip python3-command-runner python3-dotenv python3-pip python3-psutil python3-pymysql python3-redis python3-setuptools python3-systemd rrdtool snmp snmpd unzip wget whois
```

## Add librenms user

```bash
sudo useradd librenms -d /opt/librenms -M -r -s "$(which bash)"
```

## Download LibreNMS

```bash
cd /opt
sudo git clone https://github.com/librenms/librenms.git
```

## Set permissions

```bash
sudo chown -R librenms:librenms /opt/librenms
sudo chmod 771 /opt/librenms
sudo setfacl -d -m g::rwx /opt/librenms/rrd /opt/librenms/logs /opt/librenms/bootstrap/cache/ /opt/librenms/storage/
sudo setfacl -R -m g::rwx /opt/librenms/rrd /opt/librenms/logs /opt/librenms/bootstrap/cache/ /opt/librenms/storage/
```

## Install PHP dependencies

Change to the LibreNMS user:
```bash
sudo su - librenms
```
Then run the composer wrapper script and exit back to the previous user:
```bash
./scripts/composer_wrapper.php install --no-dev
exit
```

## Set the PHP timezone

Find timezone from https://php.net/manual/en/timezones.php  
Edit both files :
```bash
sudo nano /etc/php/8.4/fpm/php.ini
```
```bash
sudo nano /etc/php/8.4/cli/php.ini
```
find ```;date.timezone =``` by ```ctrl+f``` and edit :
```
date.timezone = Asia/Kolkata
```

## Set the system timezone

```bash
sudo timedatectl set-timezone Asia/Kolkata
```
---
