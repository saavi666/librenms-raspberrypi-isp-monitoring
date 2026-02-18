# Huawei Switch SNMP Configuration (Dual NMS)

## Overview

This document describes a safe method to allow a Huawei switch to be monitored by multiple Network Management Systems (NMS) simultaneously.

The configuration allows LibreNMS to monitor the switch while keeping the existing ISP monitoring configuration completely unchanged.

The approach is non-destructive and follows standard operational practices used in ISP environments.

---

## Existing Environment

The Huawei switch already has SNMP enabled and is monitored by an existing ISP Network Management System.

Key characteristics:

* SNMP agent already active.
* Existing SNMP community in use by ISP monitoring.
* SNMP traps configured for ISP NMS.
* Production traffic running through the switch.

Because of this, modifying or replacing existing SNMP configuration is avoided.

---

## Design Approach

Instead of modifying existing SNMP settings, a separate read-only SNMP community is created specifically for LibreNMS.

The configuration principles are:

* Do not modify existing SNMP communities.
* Do not remove existing trap configurations.
* Provide read-only access only.
* Restrict access using an ACL bound to the LibreNMS server IP.

This ensures both monitoring systems operate independently.

---

## Configuration Components

### Access Control List (ACL)

An ACL is created to allow SNMP access only from the LibreNMS server.

Example concept:

```
acl number 2999
 rule permit source <LIBRENMS_IP> 0
```

This prevents unauthorized SNMP access from other devices.

---

### Read-Only SNMP Community

A new read-only community is added and bound to the ACL.

Example concept:

```
snmp-agent community read librenmsRO acl 2999
```

This allows LibreNMS to poll the device while keeping existing SNMP communities intact.

---

## Verification

SNMP connectivity should be verified from the LibreNMS server before adding the device to LibreNMS.

Example verification:

```
snmpwalk -v2c -c librenmsRO <SWITCH_IP> 1.3.6.1.2.1.1
```

Successful output confirms:

* SNMP communication is working
* ACL configuration is correct
* Device information is readable

---

## Operational Result

After configuration:

* ISP NMS continues monitoring normally.
* LibreNMS receives read-only SNMP access.
* Both monitoring systems operate independently.
* No service interruption occurs.

This method allows safe integration of additional monitoring systems into existing ISP infrastructure.

---

## Summary

Using a dedicated read-only community with ACL restriction provides a safe and scalable way to introduce additional monitoring platforms without impacting production monitoring environments.
