# Telegram Alert Integration

## Overview

Telegram is used as the primary alert notification method for LibreNMS. This provides immediate notification of network events without requiring continuous access to the LibreNMS web interface.

The configuration uses the official Telegram Bot API and LibreNMS built-in alert transport.

---

## Telegram Bot Creation

A Telegram bot is created using the official BotFather service.

Steps:

1. Open Telegram and search for **@BotFather**.
2. Create a new bot using the `/newbot` command.
3. Assign a bot name and username.
4. Save the generated bot token securely.

The bot token is required for LibreNMS configuration.

---

## Obtaining Chat ID

The Telegram chat ID identifies where alerts are delivered.

Steps:

1. Start a conversation with the created bot.
2. Send a message to the bot.
3. Retrieve the chat ID using a Telegram information bot or API query.

The chat ID is used by LibreNMS to deliver alerts to the correct recipient.

---

## LibreNMS Transport Configuration

Telegram is added as an alert transport inside LibreNMS:

* Navigate to **Settings → Alerting → Alert Transports**
* Add a new transport
* Select **Telegram**
* Enter the bot token
* Enter the chat ID
* Enable the transport

Once saved, LibreNMS can send notifications directly to Telegram.

---

## Alert Rule Integration

Alert rules are configured to use the Telegram transport.

When an alert condition is triggered:

1. LibreNMS evaluates the alert rule.
2. The alert is passed to the Telegram transport.
3. A notification is delivered to the configured chat.

Typical alert conditions include:

* Device down events
* Interface status changes
* Critical device conditions

---

## Verification

Alert delivery can be verified by triggering a test alert or using the built-in test function within LibreNMS alert transport settings.

Successful delivery confirms correct integration.

---

## Summary

Telegram integration provides a lightweight and reliable alerting mechanism for LibreNMS deployments, enabling rapid operational response to network events.
