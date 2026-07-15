# Changelog

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
