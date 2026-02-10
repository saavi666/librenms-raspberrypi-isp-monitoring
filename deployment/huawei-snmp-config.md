# Huawei Switch SNMP Deployment Reference

## Overview

This document contains the CLI configuration used to allow LibreNMS to monitor a Huawei switch that is already monitored by an existing ISP Network Management System.

The configuration is non-destructive and does not modify existing SNMP settings.

---

## Objective

* Add LibreNMS as a second SNMP manager.
* Maintain existing ISP monitoring configuration.
* Provide read-only access only.
* Restrict SNMP access to the LibreNMS server IP.

---

## Enter System View

Purpose: Enter configuration mode.

```
system-view
```

Expected Result:

```
[Huawei]
```

---

## Create Access Control List

Purpose: Allow SNMP access only from LibreNMS server.

```
acl number 2999
 rule permit source 10.0.0.10 0
 quit
```

Result:
ACL created allowing only LibreNMS server IP.

---

## Create Read-Only SNMP Community

Purpose: Add a separate SNMP community for LibreNMS without affecting ISP configuration.

```
snmp-agent community read librenmsRO acl 2999
```

Result:
LibreNMS can poll the device using read-only SNMP access.

Existing ISP SNMP community remains unchanged.

---

## Save Configuration

Purpose: Ensure configuration persists after reboot.

```
save
```

Expected Result:

```
Configuration is saved successfully.
```

---

## SNMP Verification from LibreNMS Server

Purpose: Confirm SNMP communication before adding device to LibreNMS.

```
snmpwalk -v2c -c librenmsRO <SWITCH_IP> 1.3.6.1.2.1.1
```

Expected Result:

Device system information returned successfully.

---

## Operational Result

After configuration:

* ISP monitoring continues unchanged.
* LibreNMS gains read-only monitoring access.
* Both monitoring systems operate independently.

---

## Summary

Using a dedicated SNMP community with ACL restriction allows safe integration of additional monitoring systems into production Huawei network environments without operational risk.
