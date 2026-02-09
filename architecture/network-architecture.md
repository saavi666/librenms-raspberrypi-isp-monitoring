# Network Architecture

## Overview

This deployment uses a **Raspberry Pi 5** as a lightweight Network Management System (NMS) running LibreNMS to monitor ISP infrastructure devices.

The monitoring system is designed to operate alongside an existing ISP monitoring platform without modifying or interfering with its configuration.

The architecture separates monitoring traffic from user traffic using VLAN-based management networks.

---

## Design Goals

The architecture was designed with the following objectives:

* Deploy a low-power monitoring system using commodity hardware.
* Monitor infrastructure devices without affecting existing ISP monitoring.
* Maintain safe read-only SNMP access.
* Keep management traffic isolated from subscriber traffic.
* Allow future expansion to additional switches and network devices.

---

## Network Components

### Raspberry Pi (LibreNMS Server)

* Runs LibreNMS, MariaDB, Apache, and SNMP services.
* Operates as the Network Management System.
* Configured in headless mode for reduced resource usage.
* Connected to infrastructure VLAN for management access.

### Huawei 10G Switch

* Core infrastructure switch running Huawei VRP.
* Already monitored by ISP Network Management System.
* Provides SNMP access to LibreNMS using a separate read-only community.
* Existing ISP SNMP configuration remains unchanged.

---

## Management Network Layout

The Raspberry Pi has multiple logical interfaces:

* **Infrastructure VLAN (VLAN 3000)**
  Used for monitoring infrastructure devices such as switches and OLTs.
  LibreNMS communicates with network devices using this interface.

* **Local LAN Network**
  Used for administrative access to the LibreNMS web interface.

Monitoring traffic is sourced from the infrastructure VLAN interface to keep management paths consistent with network topology.

---

## Dual SNMP Manager Design

The Huawei switch is monitored by two independent SNMP managers:

1. Existing ISP Network Management System
2. LibreNMS running on Raspberry Pi

This is achieved by:

* Creating a separate read-only SNMP community for LibreNMS.
* Restricting access using an ACL bound to the LibreNMS server IP.
* Leaving the ISP SNMP configuration untouched.

This approach ensures safe parallel monitoring without operational risk.

---

## Monitoring Flow

```
Network Device (Huawei Switch)
            ↓
       SNMP Agent
            ↓
LibreNMS (Raspberry Pi)
            ↓
 Monitoring Database
            ↓
 Alert Engine
            ↓
 Telegram Notifications
```

---

## Summary

This architecture provides a lightweight and reliable monitoring solution suitable for small ISP environments, lab deployments, or distributed infrastructure monitoring where low power consumption and safe integration with existing systems are required.
