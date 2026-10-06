# FrED Engine

FrED Engine is the native inference backend for the FrED Home Assistant
integration. Install and start the add-on, then add FrED from **Settings >
Devices & services**. Supervisor discovery supplies the private endpoint and
credential automatically.

The add-on stores its instance identity, API credential, accepted
configuration, and durable command state under `/data`, which Home Assistant
includes in add-on backups. Since 0.17.0 it also keeps a rolling log in
`/data/logs/` and a history of published states in `/data/state-history/`,
each capped at 64 MiB. Backups leave out the contents of both directories,
and of `/data/feedback-bundles/`, where every LPS Next feedback report
archives a triage bundle (up to 64 MiB each, 1 GiB in all; a full store
rejects new bundles visibly and never evicts). Copy these files off the box
yourself if you need them; bundles are exported over the engine's bearer API.
Backups also leave out `/data/comparison-history/`, the rolling record of
production and LPS Next side by side that the engine keeps while LPS Next
dogfooding is switched on in the Home Console (at most 48 hours and 128 MiB).
A report that cites a moment from it copies that moment into
`/data/comparison/`, which backups keep with the reports themselves.
A restore brings these directories back empty, so the log, state history and
comparison history start again from the restore.

To fill a triage bundle, the engine reads its own Supervisor log and Home
Assistant Core's log for the incident window. Reading the Core log needs
Supervisor API access (`hassio_api`) with the `homeassistant` role
(`hassio_role`), the least role that allows it.

That role is not limited to logs. Supervisor allows it every request matching
`/.+/info`, `/core/.+` and `/homeassistant/.+`, which includes controlling Home
Assistant Core: restarting, stopping, updating and changing its options. The
engine itself only registers discovery and reads `/addons/self/info`,
`/addons/self/logs` and `/core/logs`, but code that ran inside the add-on could
use its Supervisor token for anything the role allows. Weigh that before
installing. Removing `hassio_api` and `hassio_role` from the add-on's
`config.yaml` drops the Core log from triage bundles (recorded there as a named
gap) and changes nothing else.

Home Assistant history and logbook come from the separate Core API
(`homeassistant_api`), which the add-on already uses, over a connection
separate from the event stream.

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

### Liveness and the watchdog

Supervisor's watchdog polls `GET /live`, which is **unauthenticated by design**:
Supervisor has no way to present the engine's bearer token, and `/api/v1/health`
is bearer-gated. `/live` returns only `{"status":"ok"}` — no status, version,
instance id, or configuration — and reads no engine state.

It is reached over the internal container network, so **no host port is
published** and the boundary described above stays intact.

The probe is liveness, not readiness: it proves the process is scheduled and
accepting connections. It answers `200` while the engine is waiting for its
first configuration (a healthy state, not a hang), and it deliberately takes no
runtime lock, so a stalled internal lock would still answer `200`.

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
