# Telegram Alert Deployment Reference

## Overview

This document describes the configuration used to integrate Telegram alerts with LibreNMS.

Telegram is used as the primary notification mechanism for device and interface alerts.

Only the final working configuration is documented.

---

## Create Telegram Bot

Purpose: Create a bot used by LibreNMS to send notifications.

Steps:

1. Open Telegram.
2. Search for **@BotFather**.
3. Run the command:

```
/newbot
```

4. Assign a bot name and username.
5. Save the generated bot token.

The bot token is required for LibreNMS configuration.

---

## Obtain Chat ID

Purpose: Identify where alerts should be delivered.

Steps:

1. Start a chat with the created bot.
2. Send any message to the bot.
3. Retrieve the chat ID using a Telegram information bot or API request.

The chat ID is required by LibreNMS to send notifications.

---

## Configure Telegram Transport in LibreNMS

Purpose: Add Telegram as an alert delivery method.

LibreNMS Web Interface:

```
Settings → Alerting → Alert Transports
```

Configuration:

* Transport Type: Telegram
* Bot Token: <BOT_TOKEN>
* Chat ID: <CHAT_ID>
* Enabled: Yes

Save configuration after entry.

---

## Alert Rule Assignment

Purpose: Ensure alerts are sent through Telegram transport.

Within LibreNMS alert rule configuration:

* Assign Telegram as the alert transport.
* Apply transport to device or global alert rules.

---

## Verification

Purpose: Confirm alert delivery is operational.

Use the built-in test option in LibreNMS alert transport settings.

Successful delivery confirms correct configuration.

---

## Summary

Telegram alerts provide immediate notification of network events and allow operators to respond quickly to device or interface issues without constant monitoring of the LibreNMS dashboard.

