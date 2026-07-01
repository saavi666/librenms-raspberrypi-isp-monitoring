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
save : ```ctrl+x``` then ```Y``` then ```enter```
## Set the system timezone

```bash
sudo timedatectl set-timezone Asia/Kolkata
```

## Configure MariaDB

Open the file :
```bash
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
```
Within the ```[mariadbd]``` section add:
```bash
innodb_file_per_table=1
lower_case_table_names=0
```
Save : ```ctrl+x``` then ```Y``` then ```enter```

Restart mariaDB :
```bash
sudo systemctl enable mariadb
sudo systemctl restart mariadb
```

Start MariaDB client :
```bash
sudo mysql -u root
```
MariaDB config : 

NB:Change the 'password' below to something secure.
```bash
CREATE DATABASE librenms CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'librenms'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON librenms.* TO 'librenms'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```
Change password later :
```bash
ALTER USER 'librenms'@'localhost' IDENTIFIED BY 'newpassword';
FLUSH PRIVILEGES;
EXIT;
```
## Configure PHP-FPM

Copy the default pool configuration :
```bash
sudo cp /etc/php/8.4/fpm/pool.d/www.conf /etc/php/8.4/fpm/pool.d/librenms.conf
```
Edit the new pool :
```bash
sudo nano /etc/php/8.4/fpm/pool.d/librenms.conf
```
Change ```[www]``` to ```[librenms]``` :
```bash
[librenms]
```
Change user and group to ```"librenms"```:
```bash
user = librenms
group = librenms
```
Change ```listen``` to a unique path that must match your webserver's config :
```bash
listen = /run/php-fpm-librenms.sock
```
Restart PHP-FPM :
```bash
sudo systemctl restart php8.4-fpm
```
Verify :
```bash
systemctl status php8.4-fpm
```
NB:If there are no other PHP web applications on this server, rename www.conf (rather than deleting it) to disable it and save resources. This allows you to restore it later if needed. :
```bash
sudo mv /etc/php/8.4/fpm/pool.d/www.conf \
/etc/php/8.4/fpm/pool.d/www.conf.disabled
```

## Configure Web Server

edit :
```bash
sudo nano /etc/nginx/sites-enabled/librenms.vhost
```
Add the following config, edit server_name as required:
```bash
server {
 listen      80;
 server_name librenms.example.com;
 root        /opt/librenms/html;
 index       index.php;

 charset utf-8;
 gzip on;
 gzip_types text/css application/javascript text/javascript application/x-javascript image/svg+xml text/plain text/xsd text/xsl text/xml image/x-icon;
 location / {
  try_files $uri $uri/ /index.php?$query_string;
 }
 location ~ [^/]\.php(/|$) {
  fastcgi_pass unix:/run/php-fpm-librenms.sock;
  fastcgi_split_path_info ^(.+\.php)(/.+)$;
  include fastcgi.conf;
 }
 location ~ /\.(?!well-known).* {
  deny all;
 }
}
```
Change server_name  

Access via any ip
```bash
server_name _;
```
Using your hostname
```bash
server_name librenmspi;
```
browse to ```http://signetpi``` if pi resolves hostnames  

Using specific IP
```bash
server_name 192.168.x.x;
```
Or use multiple configs :
```bash
server_name librenmspi 192.168.x.x;
```
Remove default ngix site and reload :
```bash
sudo rm /etc/nginx/sites-enabled/default
sudo systemctl reload nginx
sudo systemctl restart php8.4-fpm
```
Test the Nginx configuration
```bash
sudo nginx -t
```
Verify :
```bash
systemctl status nginx
systemctl status php8.4-fpm
```
---
