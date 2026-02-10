# Raspberry Pi Deployment Reference

## Overview

This document contains the commands and configuration steps used to prepare the Raspberry Pi as a dedicated LibreNMS monitoring server.

Only the final operational configuration is included. Troubleshooting steps are intentionally omitted.

---

## System Update

Purpose: Ensure base operating system packages are up to date.

```bash
sudo apt update
sudo apt upgrade -y
```

---

## Enable Headless Operation

Purpose: Disable graphical desktop environment and run the system in CLI mode to reduce CPU and memory usage.

```bash
sudo systemctl set-default multi-user.target
```

Result:
System boots without graphical desktop.

---

## Disable Non-Essential Services

Purpose: Reduce background resource usage and prevent unnecessary services from running on a monitoring server.

```bash
sudo systemctl disable --now cups.service cups.socket cups.path
sudo systemctl disable --now avahi-daemon.service avahi-daemon.socket
sudo systemctl disable --now bluetooth.service
```

Result:
Lower baseline CPU and memory usage.

---

## Required Services for LibreNMS

Purpose: Ensure required services remain enabled for monitoring operation.

```bash
sudo systemctl enable apache2
sudo systemctl enable mariadb
sudo systemctl enable snmpd
sudo systemctl enable ssh
```

Result:
Core services start automatically on boot.

---

## Verify Service Status

Purpose: Confirm monitoring stack is operational.

```bash
systemctl status apache2
systemctl status mariadb
systemctl status snmpd
```

Expected:
Services active and running.

---

## Network Role

The Raspberry Pi operates with separate logical roles:

* Infrastructure VLAN interface used for SNMP polling.
* Local LAN interface used for administrative access.

LibreNMS polling traffic originates from the infrastructure VLAN interface.
