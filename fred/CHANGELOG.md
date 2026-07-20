# Changelog

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
