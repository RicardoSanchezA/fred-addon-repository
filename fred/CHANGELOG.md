# Changelog

## 0.16.0

- Add daylight awareness: a solar clock, lux ingest, and occupancy
  eligibility so automatic lighting can stay off during the day and ramp
  toward Bright or the selected profile at dawn and dusk.
- Protocol 5 / configuration schema 8. The engine still loads persisted
  schemas 6 and 7; new integration payloads write schema 8. Upgrade the
  engine/add-on first, then restart Core so the integration can re-adopt
  protocol 5 discovery and reapply configuration.
- Off and disconnected clock paths advance solar deadlines without
  dispatching lights. Remotes record Glow manual-on / manual-dark and
  project lighting only when the engine is allowed to dispatch.

## 0.15.0

- Preserve a person's observed walk across motion events, so a continuing
  walker or a valid return wins over a sitter in the destination room. Clear
  inference now rejects unsupported quiet-rival moves, stale or unavailable
  destinations, and routes that cross a shut interior door.
- Classify circulation halls from the per-area configuration rather than
  hard-coded room IDs. The integration migrates the existing `pasillo` and
  `pasillo__2` halls, so the degree-three second hallway uses the intended
  destination rule instead of falling back to neighbour-edge inference.

## 0.14.0

- Replace the clear-wave repair model with an uncertain-state engine. A track
  whose evidence cannot be narrowed now carries a set of candidate areas and
  keeps every one of them lit, instead of the engine dropping to simple
  lighting when it could not decide.
- Reconcile the detect and clear chains by **interval compatibility** rather
  than by order. A latched PIR emits one Detect and then extends silently, so a
  room an occupant walks back into clears last while its rising edge stays put;
  comparing orders reported those ordinary walks as contradictions.
- Add the D6 source-motion gate: nobody is moved out of a room whose trusted
  PIR reported nothing during the walk. Candidate authors are chosen by
  evidence that a body actually walked the route, not by topological
  reachability, and the route now respects shut doors.
- Place an occupant on a clear when motion supports exactly one destination,
  and widen instead of guessing when several are equally supported or a room is
  already occupied.
- Replace auto-degrade with a resync/quarantine ladder: a contradiction rebuilds
  presence from live Home Assistant state, repeated contradictions in one room
  distrust that room's sensor, and `dumb_home` is reached only when the house
  can no longer be modelled. A completed departure now resets the ladder.
- Correct the PIR clear-delay default from 15s to a measured 22s, and bound the
  occupant-count floor so a missed departure cannot hold a room lit
  indefinitely.

## 0.13.1

- Defer an ambiguous interior-door close when the observed open interval already
  explains the two sustained motion readings as one out-and-back passage. The
  watch consumes when either side clears, still routes fresh motion on both
  sides to normal split handling, and retains its 90-second fail-safe.
- Preserve an active deferred watch through short door-contact open/close
  chatter, require the full hold deadline on both timer and event paths, and
  keep clear-wave relocation from traversing a shut interior door.
- Add recorder-derived regression fixtures for the 2026-08-02 and 2026-08-03
  Sala/Pasillo incidents. No protocol, configuration-schema, or integration
  change: this is an engine/add-on-only deployment.

## 0.13.0

- Add optional authoritative presence assertions from configured point-in-time
  sources, including the macOS Mobile App Frontmost App sensor.
- Configuration schema 7 is written by the integration. The engine accepts
  persisted and applied schema 6 or 7 during this rolling release, while health
  advertises both versions.
- Upgrade the FrED engine/add-on before the integration. An integration that
  sends schema 7 to an older schema-6-only engine keeps its last good runtime
  configuration and reports an upgrade-required configuration push failure.

## 0.12.4

- A motion clear no longer relocates an occupant simply because another active
  room is topologically reachable. Clear-driven relocation now needs a
  corroborated, strictly time-ordered trail away from the cleared area, and it
  retains the occupant when the cleared area still has active evidence. This
  prevents a person sitting still in Cuarto from being pulled into an older,
  unrelated Cocina detection just because Cuarto's PIR cleared.
- The door-aware route restriction from 0.12.3 now composes with this evidence
  gate, so a clear inference must both respect closed doors and be supported by
  an observed walk.

Over a 250.9 h replay of recorded history, clear-driven relocations fall from
3,102 to 482 and non-door degrades fall from 534 to 354; door-related degrades
remain 15. Protocol 4 and configuration schema 6 are unchanged, so there is no
configuration re-apply, quarantine, or engine-mode reset on upgrade.

## 0.12.3

- A clear can no longer be read as a departure across a shut interior door (`lps-core`). Deciding where a cleared area's occupant went searched the house as if every door were open, so a Sala clear could move a track through a door that had just been shut. On 2026-07-30 that happened twice: the first left Sala empty, so the next motion there read as an extra body and degraded the engine at 21:10:57; the second put a body in Despensa while its occupant was in the shower. Both the motion-path and clear-wave routes now refuse to cross a shut door.
- The engine is now told which interior doors are shut (`fred-server`), on every door transition and at boot. `unknown` and `unavailable` count as not-shut, matching how the rest of the runtime reads a door, so a flaky sensor fails open rather than stranding an occupant.
- Both 2026-07-30 incidents are now replayed from real recorder data in CI, through the real runtime, rather than reconstructed by hand.

Over a 250 h replay of recorded history, relocations that crossed a shut door fall from 128 to zero and total degrades fall from 558 to 549.

Known residual: three door-split degrades that 0.12.2 did not produce. They are a sustained-both-sides shape the transit classifier never defers, which the bogus relocation this release removes had been masking. Net degrades still fall.

No configuration change: protocol 4 and schema 6 are unchanged, so there is no re-apply, quarantine, or engine-mode reset on upgrade.

## 0.12.2

- A door split that finds the evidence already consistent now says so instead of stopping (`lps-core`). `soft_reset_placements` returned one "no" for three different answers — the evidence is impossible, repair failed, and there was nothing to repair — and both callers turned all three into a degrade. On 2026-07-30 every lit area lay on one ordered walk ending where a body was already seated, so nothing needed placing, and FrED stopped anyway.
- Lit rooms are now counted as walks rather than as rooms. One occupant in transit lights every room they pass through and holds them all for a clear delay, which previously read as a crowd and tripped the "more rooms than people" guard. The count is an exact minimum path cover, which matters: a greedy approximation over-counts on this house's topology and that over-count is a false stop, not a safe one.
- Clear-wave repair can now route through a lit room when somebody else's body explains why it is lit — a second occupant standing in it, or in an open-plan neighbour whose motion its sensor can see. A room occupied by someone else never clears, so any route through it could never corroborate, which is what left Sala permanently blocking the route out of Cuarto. One such room per route, and never a route corroborated only by them.

No configuration change: protocol 4 and schema 6 are unchanged, so there is no re-apply, quarantine, or engine-mode reset on upgrade.

## 0.12.1

- A closing interior door is no longer read as *only* a split (`fred-server`). When a door shuts and the far side lights up a fraction of a second later, that is somebody pulling it closed behind them, not two people straddling it — but the near side is still lit for its clear delay, so at `max_occupants` every repair path declined in turn and the engine degraded. The verdict is now deferred instead of guessed: it resolves when either side goes quiet (transit, no split), when both sides carry a post-close detect (a real split), or at a 90s fail-safe. Live 2026-07-29 13:04 regression, Sala↔Pasillo-Cuartos.
- An area whose motion sensor goes `unavailable` now counts as going quiet for that decision. Such an area reads `Unknown` rather than `Clear`, which previously reached nothing at all — so the very walk this fix targets, where a Pasillo-Cuartos sensor was unavailable throughout, would have been delayed 90s rather than spared.
- Door closes reported twice within two seconds (one physical close, with a bounce open between) are now a single watch anchored to the first close, so a bounce cannot restart the deferral clock or discard the motion evidence recorded before it.

No configuration change: protocol 4 and schema 6 are unchanged, so there is no re-apply, quarantine, or engine-mode reset on upgrade.

## 0.12.0

- **Configuration schema 6.** Expect a one-time `configuration.json` quarantine and a `waiting_for_config` window while the integration reapplies config; the engine mode resets to its default across that quarantine, so re-set it if it was anything but `smart_home`.
- `device_tracker` entities may now be area keep-alive evidence (`long_lived_entities`). Classification is domain-aware -- `home` is active and every other state, including `not_home` and named zones, is softened by a new `device_tracker_grace_seconds` (default 300s) rather than the appliance hold. Grace no longer re-arms on repeated readings in any domain, which also fixes a television stuck `unavailable` holding its area lit forever.
- New `person_home_entities` (restricted to `person.*`) and `person_home_grace_seconds` decide how many people are home. That count keeps a departure that retires the last placement in `unknown` rather than `away`, so the next interior motion locates whoever stayed instead of degrading the engine; it lets tracker absence conclude an empty house behind a completed-departure guard and a no-recent-motion guard; and it biases spawn-vs-relocate alongside, never replacing, the manual People-home claim. It is never written into the occupant floor.
- Concluding the house is empty now withdraws every area's keep-alive support, whatever the domain and whether the reading is live or held by grace.
- New diagnostics: a tracker-derived people count with its located/unlocated split, and a tracker-conflict sensor covering a sustained body excess, a tracker holding the house open for the full unlocated window, and an absence a guard refused.

## 0.11.11

- Clear-wave repair now requires a corroborated trail before relocating a floor-retained occupant (`lps-core`). Bare topological reachability could teleport a retained body across the house onto any uniquely-plausible area, emptying the area it left so its next real motion read as an extra occupant and degraded the engine at `max_occupants`. The route is now searched for one whose intermediate areas actually went quiet, in travel order.
- Interior door watches whose sustained PIR trail has been superseded are consumed instead of resolved (`fred-server`). Both sides staying lit through their clear delay after the occupant had already moved on produced split evidence with no active track on either side, which disabled the engine. The guard sits on the shared resolution path, covering both the timer and state-change routes.
- Accept summaries in decision logs now carry their reason, so a timer-sourced relocation is distinguishable from an ordinary walk.

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
