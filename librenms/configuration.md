# LibreNMS Configuration

## Overview

LibreNMS is deployed as the central Network Management System (NMS) for monitoring infrastructure devices using SNMP. The configuration described here reflects the final operational state after successful deployment.

The system monitors network devices without modifying existing device configurations used by other monitoring systems.

---

## Device Onboarding

Devices are added to LibreNMS using SNMP v2c read-only access.

Required parameters:

* Device management IP address
* SNMP version: v2c
* Read-only community string

After addition, LibreNMS automatically:

* Detects device operating system
* Discovers interfaces and sensors
* Begins periodic polling
* Generates performance graphs

---

## SNMP Polling Workflow

LibreNMS interacts with devices using standard SNMP polling:

1. LibreNMS sends SNMP requests from the infrastructure VLAN interface.
2. Network devices respond through their SNMP agent.
3. Collected metrics are stored in the monitoring database.
4. Historical data is used for graph generation and alert evaluation.

Polling occurs automatically at defined intervals.

---

## Device Discovery and Polling

LibreNMS performs two primary background operations:

### Discovery

* Detects new interfaces and sensors.
* Updates device metadata.
* Runs periodically.

### Polling

* Collects operational metrics.
* Updates device status.
* Stores performance data.

These processes run automatically as part of LibreNMS services.

---

## Alert Rules

Alert rules are configured to monitor device availability and operational status.

Typical alert conditions include:

* Device unreachable
* Interface down events
* Critical device status changes

Alert rules evaluate device state continuously and trigger notifications when conditions are met.

---

## Alert Transport

Telegram is configured as the primary alert transport.

When an alert condition is triggered:

1. LibreNMS evaluates the alert rule.
2. The alert is passed to the configured transport.
3. A notification is delivered to Telegram via bot API.

This provides immediate operational visibility without requiring access to the LibreNMS interface.

---

## Summary

LibreNMS operates as an automated monitoring system that continuously polls network devices, stores operational metrics, and generates alerts when abnormal conditions are detected. The configuration prioritizes stability, read-only monitoring, and safe coexistence with existing monitoring platforms.
