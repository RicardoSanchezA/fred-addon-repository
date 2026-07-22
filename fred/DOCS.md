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

### Auth model

- **Sidebar / ingress:** Supervisor authenticates the HA user, proxies the
  browser to the add-on, and injects `X-Ingress-Path`. The engine treats a
  non-empty `X-Ingress-Path` as sufficient for `/ui/v1/*` only.
- **Integration API (`/api/v1/*`):** always requires the backend bearer token.
  Ingress headers never authorize configuration or integration commands.
- **Standalone dogfood:** open the engine `/ui/` directly and paste the bearer
  token when prompted (stored in `sessionStorage` for the tab only).

This matches the usual Home Assistant add-on tradeoff: the container network
is assumed not to be reachable by arbitrary LAN clients who could forge
`X-Ingress-Path`. **That network boundary is load-bearing, not incidental.**
Do not publish host port 8099 (no `ports:` mapping) without a reverse-proxy
auth layer: forging `X-Ingress-Path` from outside Supervisor ingress bypasses
the bearer for `/ui/v1/*` commands.

Supervisor options stay intentionally thin (`observer_mode`, `max_occupants`):
only settings that must be known before the integration pushes configuration
belong here. Structural topology and lighting targets are configured in the
FrED integration, not in the add-on options form.

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
