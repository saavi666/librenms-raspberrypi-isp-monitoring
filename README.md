# LibreNMS Raspberry Pi ISP Monitoring Setup

## Overview

This project documents the deployment of a **LibreNMS network monitoring system** running on a **Raspberry Pi 5**, designed for monitoring ISP infrastructure including Huawei switches using SNMP.

The setup focuses on:

* Lightweight monitoring hardware
* Safe dual-NMS SNMP configuration
* VLAN-based management networks
* Telegram alert integration

This repository contains the **final working configuration** after deployment and validation.

---

## Hardware Used

* Raspberry Pi 5 (4GB RAM)
* MicroSD Storage
* Huawei S6700 Series 10G Switch
* ISP Access Network (OLT + Access Switches)

---

## Software Stack

* Raspberry Pi OS (Debian based)
* LibreNMS
* MariaDB
* Apache2
* SNMP (v2c)
* Telegram Bot API

---

## Network Architecture (Summary)

* Raspberry Pi acts as the Network Management System (NMS)
* Management traffic routed through VLAN-based infrastructure
* Huawei switch monitored using a dedicated read-only SNMP community
* Existing ISP monitoring configuration remains untouched

---

## Project Structure

```
architecture/
raspberry-pi/
librenms/
huawei-switch/
alerts/
```

Each section explains the final configuration used in production.

---

## Features Implemented

* LibreNMS deployment on Raspberry Pi
* Headless optimized server configuration
* SNMP monitoring for Huawei VRP devices
* Dual SNMP manager support (ISP NMS + LibreNMS)
* Telegram alert notifications

---

NB : The seperate VLAN configurations done in the raspberry pi and huawei switch are in a seperate repository "github.com/saavi666/raspberrypi-vpn-network-management". That repo also include the setup of Wireguard VPN using piVPN for remote access to the LAN for monitoring and management purposes.
