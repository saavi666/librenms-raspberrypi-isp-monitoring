# LibreNMS Deployment Reference

## Overview

This document contains the operational commands and configuration references used for running LibreNMS on the Raspberry Pi monitoring server.

Only the final working configuration is documented.

---

## LibreNMS Directory

Purpose: LibreNMS is installed under the standard directory.

```bash
/opt/librenms
```

All LibreNMS commands are executed from this location.

---

## Verify Web Interface

Purpose: Confirm Apache and LibreNMS web interface are operational.

```bash
curl http://localhost
```

Expected Result:

HTML redirect response to `/login`.

---

## SNMP Verification (Local)

Purpose: Confirm SNMP daemon is functioning before device polling.

```bash
snmpwalk -v2c -c librenmsRO localhost 1.3.6.1.2.1.1
```

Expected Result:

System description and uptime information returned.

---

## Device SNMP Verification (Remote)

Purpose: Verify SNMP connectivity to network devices before adding to LibreNMS.

```bash
snmpwalk -v2c -c librenmsRO <DEVICE_IP> 1.3.6.1.2.1.1
```

Expected Result:

Device system information returned successfully.

---

## LibreNMS Services

Purpose: Ensure LibreNMS background services are active.

```bash
systemctl status librenms.service
```

Expected Result:

Services active and running.

---

## Device Onboarding Reference

Devices are added through the LibreNMS web interface.

Required parameters:

* Device IP address
* SNMP Version: v2c
* SNMP Community: librenmsRO

After addition, LibreNMS automatically performs:

* OS detection
* Interface discovery
* Sensor discovery
* Periodic polling

---

## Polling Verification

Purpose: Confirm device polling is operational.

LibreNMS web interface should show:

* Device uptime increasing
* Interface statistics updating
* CPU and memory graphs populating

---

## Alert Transport Reference

Telegram transport is configured within LibreNMS alert settings.

Verification is performed using the built-in transport test option in the LibreNMS interface.

---

## Summary

LibreNMS operates as a continuous polling system where SNMP data is collected from infrastructure devices and processed into monitoring metrics and alerts.
