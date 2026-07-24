# Changelog

## 0.11.10

- Fix reset seeding occupants defaulting to `away` and degrading on first interior motion (`lps-core`). Reset/enable empty tracks now default to `unknown` and preserve `away` only when already set; `handle_departure_timeout` is the single path entering `away`.
- Glow section & map UI now derive light "on" state directly from `lit_areas` (`fred-protocol` / `fred-server`), eliminating stale yellow light indicators caused by HA light entity echo lag.

## 0.11.9


- Harden protocol-bump migration: the integration re-reads and adopts live
  Supervisor discovery when the stored config-entry record predates a bump, now
  covered by setup-path regression tests. No protocol or schema change.
- First release deployed via the image-pull path (private GHCR image + host
  registry auth) instead of a local source build.

## 0.11.8

- Fix FrED integration setup failing after the v4 protocol bump. A config entry
  keeps the discovery record's protocol version from when it was created, and
  Supervisor never re-notifies on a changed record, so setup rejected the stale
  v3 discovery and never started. The integration now re-reads the live
  Supervisor discovery and adopts it when the stored one predates a protocol
  bump. Engine and add-on are version-only bumps.

## 0.11.7

- Replace the enabled/disabled model with explicit engine modes: `off`
  (observe-only), `dumb_home` (simple motion lighting), and `smart_home` (full
  tracking). An impossible presence model now degrades to `dumb_home` instead of
  switching the house off, so lighting keeps working while a notification and
  `binary_sensor.fred_degraded` explain why.
- Reset now takes the number of people home, seeding tracking from ground truth
  instead of a guess.
- Add engine mode controls, reset, and an activity feed to the Home Console; the
  Lovelace dashboard becomes a thin pointer to it.
- Bump the FrED protocol to v4 (engine and integration release together).

## 0.11.6

- Stop a clear that a seated track already accounts for from being read as
  evidence that some other body moved into that area, which could relocate a
  still occupant's placement and disable the engine on their next motion.

## 0.11.5

- Preserve automation through contradicted closed-door placements by relocating
  bodies onto credible live motion instead of disabling.
- Infer clears per track, and back-date clear-derived evidence to when the PIR
  last saw motion so clears no longer outrank fresher detects.

## 0.11.4

- Fix the Home Console highlighting the wrong guest room: the `cuarto_visitas`
  and `cuarto_visitas__2` region ids were on each other's rectangles.
- Record occupant-count-floor retentions and weaken a placement once a
  clear-wave shows the body moved on (Phases 1-2). No body is ever deleted.
- Adopt a re-minted backend token on `401` instead of deadlocking when the
  add-on rotates its credential (C7).

## 0.11.3

- **Interior-door split observability**: log the full watch lifecycle -- armed,
  matured, held-untrusted, expired, cancelled -- plus the split outcome. A door
  close that never becomes a split was previously indistinguishable in the logs
  from a door FrED never saw.
- A credible closed-door split may now reclaim a quiet occupant-count-floor
  retained track at `max_occupants`, correcting placement while preserving the
  body count. Ambiguous pending-spawn motion still cannot.
- Mark an area stale when it loses sensor observability (`Active -> Unknown`)
  instead of leaving it active indefinitely.

## 0.11.2

- Coordinated release: engine fix for areas that lose sensor observability
  (an area whose motion entities go unavailable is now marked stale rather
  than staying active indefinitely), and an integration guard rejecting
  non-binary long-lived evidence entities.
- No add-on behaviour change; the version moves so the cross-repo contract
  stays consistent.

## 0.11.1

- Add a Supervisor **watchdog** so a hung engine is detected and restarted.
- New unauthenticated `GET /live` liveness endpoint (returns only
  `{"status":"ok"}`; exposes no state). Reached over the internal container
  network — no host port is published and the ingress trust boundary is
  unchanged.

## 0.11.0

- Configuration schema **5**: remove unused `door_evidence_seconds` option (never
  consumed by the engine).
- **Upgrade order (load-bearing):** update the FrED Home Assistant integration
  first, then this add-on. Integration-first keeps lighting decisions running
  while the engine still holds a schema-4 config; add-on-first would quarantine
  configuration and leave the home without FrED lighting until Core catches up.

## 0.10.9

- Remediation: document load-bearing ingress/network boundary for UI auth.
- Coordinate with freds-crib / integration 0.10.9 (timer, recovery, durability fixes).

## 0.10.8

- Fix Home Console map rendering by moving the SVG stylesheet out of
  `<defs>`.
- Reflow the Home Console desktop dashboard into a map plus three side
  columns: Presence, Lighting, and Controls/Activity.

## 0.10.7

- Fix the Home Console under Home Assistant ingress by keeping the layout asset
  URL relative and aligning `/ui/v1/state` snapshot cursors with the SSE event
  stream.

## 0.10.6

- Expose the FrED Home Console through Home Assistant ingress and a
  **FrED Home** sidebar panel (`/ui/`, streaming enabled for SSE).
- Requires an engine image that trusts Supervisor `X-Ingress-Path` for
  browser-facing `/ui/v1/*` routes (no bearer token in the browser).

## 0.10.5

- Rebuild the engine image for the People-home estimate override control.

## 0.10.4

- Rebuild the engine image for the coordinated FrED configuration UI polish release.

## 0.10.3

- Rebuild the engine image for interior door-close split detection.

## 0.10.2

- Rebuild the engine image for configuration schema 4 and open-connection
  lighting retention support.

## 0.10.1

- Keep the add-on release aligned with the FrED integration's read-only
  aggregate occupancy dashboard polish.

## 0.10.0

- Rebuild the engine image for protocol v3, which reports anonymous occupant
  aggregates for Phase B UI support.

## 0.9.0

- The FrED cutover. The add-on is now **FrED Engine**: slug `fred`
  (a fresh install from Supervisor's perspective -- the old `icu_engine`
  add-on is uninstalled at cutover), image
  `ghcr.io/ricardosancheza/freds-crib`, Supervisor discovery service
  `fred` (pairs with the `fred` Home Assistant integration).
- Engine contract v2/schema 3: `fred_state_update` bus event, `FRED_*`
  environment variables, no `profile_entity` helper references -- no Home
  Assistant helper entity participates in any decision.
- Add `max_occupants`, passed through to the engine as `FRED_MAX_OCCUPANTS`
  for the multi-Glower track scaffold.

## 0.8.0

- The engine now owns per-area lighting profile values. New
  `set_profile_value` and `clear_zone_profile` commands persist per-area
  brightness/color temperature and pin/release per-area profile overrides;
  values are published in `state.profile_values` and applied directly via
  `light.turn_on` (no more `script.apply_light_profile` dependency).
- Zone profile overrides are now explicit pins: setting a zone to the
  current home profile keeps it custom until cleared.
- New dynamic-remote gestures: double press cycles the home profile (or the
  zone's own profile when pinned); press-and-hold toggles following
  home/custom, acknowledged by a best-effort double flash.
- Activity records for profile changes carry a structured `outcome` field:
  `home_profile_changed`, `area_profile_changed`, `area_set_custom`, or
  `area_following_home`.
- Version aligned with the Python integration (0.8.0).

## 0.5.6

- Report the add-on package version on the ICU Engine health endpoint
  (resolved from Supervisor's `/addons/self/info`) so the Home Assistant
  integration can surface it as a diagnostic.

## 0.5.5

- Expose repeated entity-conflict and Home Assistant transport-failure
  diagnostic counters through the ICU Engine health endpoint.

## 0.5.4

- Rebuild the engine image from the current refactored runtime code after the Stage 3b and Stage 4 module-split backlog landed.

## 0.5.3

- Publish presence and activity records so the ICU dashboard can show the latest
  presence transition separately from other runtime activity.

## 0.5.2

- Watch every light's and switch's own state continuously instead of only
  ICU's own dispatched actions, so a light or switch changed by a physical
  control, the frontend, or a foreign automation is no longer invisible to
  reconciliation, profile re-application, or manual-dark enforcement.
- Fix switch-only areas (a smart plug with no separately controlled light):
  they now correctly activate from motion, a button press, or clearing
  manual-dark, rather than only ever being gated on having light entities.
- Add diagnostic logging for successful lighting dispatches, detected entity
  conflicts, and rejected commands (previously only failures and the HA
  WebSocket connection lifecycle were logged).
- Supervisor discovery now refreshes periodically instead of registering
  once at startup, retrying independently on failure.

## 0.5.1

- Areas can now have multiple light entities, dispatched as one batched
  service call instead of always targeting a single (formerly
  group-wrapped) light. Switch brightness gating uses the brightest of an
  area's lights.
- Add conflict detection: a `state_changed` event on a managed light or
  switch caused by something other than ICU itself (a foreign automation,
  the frontend, a physical switch) is now tracked and surfaced to the
  Python integration.
- Fix `/health`'s `supported_commands` list, which never advertised
  `set_switch_threshold` even though it has been dispatchable since 0.5.0.

## 0.5.0

- Add smart-plug/switch entity support: switches follow an area's lighting
  on/off state, with an optional per-area brightness threshold (reading the
  area's real light brightness from Home Assistant) that turns them off
  below a configurable percentage and back on once brightness recovers.

## 0.4.1

- Fix Supervisor discovery: the announced service now matches the Python
  integration's actual domain (`icu`, not `icu.engine`), so Home Assistant
  can route the discovery to it instead of logging an invalid-domain error.

## 0.4.0

- Add the `observer_mode` option: ICU computes and logs every lighting
  decision but never dispatches a real Home Assistant service call.

## 0.1.0

- Initial experimental package with Supervisor discovery.
