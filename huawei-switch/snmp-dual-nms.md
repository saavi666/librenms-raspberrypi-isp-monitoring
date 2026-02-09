# Huawei Switch SNMP Configuration (Dual NMS)

This section explains how a Huawei switch can be monitored by multiple SNMP managers safely.

The configuration method ensures:

* Existing ISP monitoring remains unchanged
* A separate read-only SNMP community is created
* Access is restricted using ACLs
* LibreNMS receives monitoring access without risk

Only non-destructive configuration steps are included.
