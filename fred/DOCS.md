# FrED Engine

FrED Engine is the native inference backend for the FrED Home Assistant
integration. Install and start the add-on, then add FrED from **Settings >
Devices & services**. Supervisor discovery supplies the private endpoint and
credential automatically.

The add-on stores its instance identity, API credential, accepted
configuration, and durable command state under `/data`, which Home Assistant
includes in add-on backups.

## Home Console

The add-on serves the FrED Home Console at `/ui/` and exposes it through
Home Assistant ingress. After the add-on starts, open **FrED Home** in the
sidebar. The browser never receives the backend bearer token: HA session
authentication is the trust boundary, and the engine accepts ingress-proxied
UI requests that carry Supervisor's `X-Ingress-Path` header.

The Lovelace **FrED Engine** dashboard remains available as the detailed
control/debug fallback.

## Options

### `observer_mode`

When enabled, FrED computes and logs every lighting decision exactly as it
normally would, but never calls a real Home Assistant service -- nothing
about your home ever changes. Use this to compare FrED's decisions against
whatever is currently controlling your lights before trusting it with real
control. This is not time-bounded: leave it on for as long as you want to
observe, and turn it off (and restart the add-on) once you're ready for FrED
to actually control lights.

### `max_occupants`

Sets the maximum number of occupants FrED should model for the durable
multi-Glower track scaffold. The default is `2`; accepted values are `1`
through `16`. FrED refuses to start if the environment value fails this
range check, so bare-container runs and add-on runs use the same limit.
