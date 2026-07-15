# ICU Home Assistant Add-on

Supervisor packaging for the proprietary ICU presence engine. The add-on
connects to Home Assistant through the internal WebSocket proxy and publishes
an `icu` discovery record consumed by the ICU custom integration.

The engine API remains on Home Assistant's private add-on network; no host port
is exposed.
