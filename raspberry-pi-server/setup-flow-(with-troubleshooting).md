# Raspberry Pi LibreNMS configuration flow (with troubleshooting)

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
Replace 'StrongPassword' with prefered password for LibreNMS login.

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
**if password asked for librenms user, then exit of the terminal and continue the step from a main user profile using
Paste in nano:
```
<VirtualHost *:80>
  DocumentRoot /opt/librenms/html
  ServerName librenms.local

  <Directory /opt/librenms/html>
    AllowOverride All
    Require all granted
  </Directory>
</VirtualHost>
```
Enable site:
```
sudo a2dissite 000-default.conf
sudo a2ensite librenms.conf
sudo systemctl reload apache2
```
**you could now access the site http://<pi_ip> from the lan. if not, like the case for me, i have a troubleshooting steps after snmp configurations  *****

---
### 2.7 Configure SNMP on Pi
```
sudo cp /opt/librenms/snmpd.conf.example /etc/snmp/snmpd.conf
sudo nano /etc/snmp/snmpd.conf
```
Edit these line:
* SNMP community string
```
com2sec readonly  default  <RANDOMSTRINGGOESHERE>
```
replace <RANDOMSTRINGGOESHERE> with any string which will represent the SNMP community which will be used later.
* System location
```
syslocation Rack, Room, Building, City, Country [Lat, Lon]
```
eg : syslocation Office Rack 1, Kerala, India
* System contact
```
syscontact Your Name <your@email.address>
```
Apply changes
```
sudo systemctl restart snmpd
```
Test:
```
snmpwalk -v2c -c librenmsRO localhost system
```
For my case i got an error <Unknown Object Identifier>, thats beacause there is no MIBs installed by default. you could enable MIBs if you prefer but i didnt, So we use numeric OID:
```
snmpwalk -v2c -c librenmsRO localhost 1.3.6.1.2.1.1
```
if you get an output, that means SNMP is working.

* nest step was to configure librenms from http://<pi_ip> webpage. but in my case the webpage is not loading. so here is the troubleshooting. after we continue to 3rd section "librenms web install'
---
### LibreNMS Web UI troubleshooting
Confirm the Pi’s IP address
```
Confirm the Pi’s IP address
```
Check Apache status
```
sudo systemctl status apache2
```
if failed / inactive, RUN:
```
sudo systemctl restart apache2
```
Check if Apache is listening on port 80
```
ss -tuln | grep :80
```
if the output is 
```
tcp LISTEN 0 511 *:80 *:*
```
then Apache IS listening on port 80 to all interfaces (NOT a VLAN or firewall bind issue)

Test Apache without LibreNMS (critical isolation):
```
sudo a2dissite librenms.conf
sudo a2ensite 000-default.conf
sudo systemctl reload apache2
```
Now test:
```
curl http://localhost
```
expected output : Apache default page HTML output
* for my case it worked with critical isolation so moving on...

Fix LibreNMS vhost
```
sudo a2dissite 000-default.conf
sudo a2ensite librenms.conf
sudo systemctl reload apache2
```
Fix LibreNMS permissions
```
sudo chown -R librenms:librenms /opt/librenms
sudo chmod 771 /opt/librenms
sudo setfacl -R -m u:www-data:rwx /opt/librenms
sudo setfacl -dR -m u:www-data:rwx /opt/librenms
```
test again
```
curl http://localhost
```
* for my case it failed again. Then i used Apache error to find the root cause.
Check Apache error log
```
sudo tail -n 20 /var/log/apache2/error.log
```
if you found something similar:
```
Failed opening required '/opt/librenms/html/../vendor/autoload.php'
```
here is the fix:

Confirm vendor/ is missing (sanity check)
```
ls /opt/librenms/vendor
```
Expected output
```
ls: cannot access '/opt/librenms/vendor': No such file or directory
```
then Switch to librenms user
```
sudo su - librenms
```
Go to LibreNMS directory
```
cd /opt/librenms
```
Verify location
```
pwd
```
expected output:
```
/opt/librenms
```
VERIFY, Run:
```
ls
```
expected output(sammple):
```
LICENSE
README.md
html
scripts
composer.json
composer.lock
```
run Composer (critical fix)
```
./scripts/composer_wrapper.php install --no-dev
```
expected output(sammple):
```
Loading composer repositories with package information
Installing dependencies from lock file
Generating optimized autoload files
```
VERIFY vendor directory
```
ls vendor/autoload.php
```
Expected:
```
vendor/autoload.php
```
exit out of librenms user
```
exit
```
Restart Apache:
```
sudo systemctl restart apache2
```
VERIFY
```
curl http://localhost
```
Expected:
LibreNMS login HTML

then Open LibreNMS in browser
```
http://<pi-ip>/
```
You should see the LibreNMS login page. Now continue...

---
## 3. LibreNMS web installer
### 3.1 Login
* login using the username and password configured in mariaDB
---
### 3.2 Database configuration
```
Host : localhost
Port : 3306
User : librenms
Password : StrongPassword
Database Name : librenms
```
use your credentials in username and password

if you get similar error:
```
SQLSTATE[HY000] [1045] Access denied for user 'librenms'@'localhost' (using password: YES)
```
the follow the steps below

Log in to MariaDB as root
```
sudo mariadb
```
In MariaDB Verify librenms DB user exists
```
SELECT user, host FROM mysql.user WHERE user='librenms';
```
make sure you see librenms user

Re-grant permissions
```
GRANT ALL PRIVILEGES ON librenms.* TO 'librenms'@'localhost';
FLUSH PRIVILEGES;
```
```
EXIT;
```
then i got next error
```
Build Database Starting Update... Error: index.php must be run as the user librenms. Error!
```
This error can fixed by building the database once from CLI as librenms
```
sudo su - librenms
cd /opt/librenms
./lnms migrate
```
migration will not be successfull and will give a error
```
Access denied for user 'librenms'@'localhost' (using password: NO)
```
Modern LibreNMS (Laravel-based) does NOT read DB credentials from the web form Instead, it reads them from a file:
```
/opt/librenms/.env
```
Fix:

Open the LibreNMS environment file
```
cd /opt/librenms
nano .env
```
Replace the commented DB section (remove #)
```
DB_HOST=localhost
DB_PORT=3306
DB_DATABASE=librenms
DB_USERNAME=librenms
DB_PASSWORD=NewStrongPassword
```
save the nano file

Clear config cache
```
./lnms config:clear
```
Run migration
```
./lnms migrate
```
after this step continue in the browser 
```
http://<pi_ip>
```
on next page

Recommended final selection (summary)
```
Update Channel:	Daily
Theme:	Device
Usage Reports:	Enabled
Error Reports:	Enabled
```
Now you could probably see the libreNMS dashboard page. we have successfully completed the installation part now we can continue to add device
```
Devices → All Devices
```
then
```
Hostname: <pi_ip>
SNMP: ON
SNMP Version: v2c
Port: (blank) #default 161
Transport: udp
Port Association Mode: ifIndex
Community: librenmsRO
Force add: OFF
```
After this step i got an error:
```
Could not ping <pi_ip>
```
The following steps undergo the fix of the issue, start with checking the snmp from pi

Check SNMP service is running
```
systemctl status snmpd
```
If its active and running we are fine.

Test SNMP using numeric OID
```
snmpwalk -v2c -c librenmsRO 127.0.0.1 1.3.6.1.2.1.1
```


## Summary

This configuration transforms the Raspberry Pi into a low-power, always-on network monitoring appliance capable of reliably monitoring ISP infrastructure devices using SNMP.

