# Changelog

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
