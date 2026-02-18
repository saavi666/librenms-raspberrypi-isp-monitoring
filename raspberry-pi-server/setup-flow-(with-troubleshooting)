# Raspberry Pi Setup (with troubleshooting flow)

## Overview

This document describes the design flow i had while creating the NMS server. This document also contains the troubleshooting of the issues i encounted while seting up the server

---

## Hardware Platform

* Raspberry Pi 5
* 4 GB RAM
* MicroSD storage
* Gigabit Ethernet connectivity

The device operates as an always-on monitoring appliance.

---

## Operating System

* Raspberry Pi OS (Debian-based)
* Updated to latest stable packages
* SSH enabled for remote administration
* Desktop environment disabled

The system operates entirely in headless mode.

---

## Headless Configuration

The graphical desktop environment is disabled to reduce CPU and memory usage.

System default target:

```
multi-user.target
```

This allows system resources to be dedicated to the NMS.

---

## Network Configuration

The Raspberry Pi uses multiple logical network paths:

### Infrastructure VLAN Interface

* Used for SNMP communication with switches and infrastructure devices.
* LibreNMS polling traffic originates from this interface.

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

