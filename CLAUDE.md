# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

This repo (`therapiestation/x4_warehouse_fleets`) is a **fork** of Manuel Bleichner's (mbleichner) Warehouse Fleets mod; the fork point is commit `a96167e`. Everything after that was made with Claude's help, which the README disclaimer states — keep that disclaimer (and the note that the repo will be removed on the original author's request) intact. Don't change `author` in `content.xml`.

An X4: Foundations mod ("Warehouse Fleets", extension id `WarehouseFleets`) written in Egosoft's XML scripting languages (Mission Director `md/`, AI scripts `aiscripts/`, library diffs `libraries/`) plus one Lua UI patch (`ui/`). There is no build system, linter or test suite — the game is the only runtime. Dependencies: SirNukes Mod Support APIs (options menu via `md.Simple_Menu_API`) and kuertee's UI Extensions (Lua callbacks).

## Develop / verify

- Syntax check after edits: `xmllint --noout md/warehousefleets.xml` (and any other touched XML).
- Deploy for testing: the repo must be inside `<X4 install>/extensions/`. On this machine `~/.local/share/Steam/steamapps/common/X4 Foundations/extensions/x4_warehouse_fleets` is a **symlink to this repo**, so the game always runs the current working tree (including uncommitted changes) — restart the game / load a save after edits. MD changes to cue versions/patches apply on load.
- Savegame inspection: `zcat` a save into the scratchpad and grep it. Ship orders appear as `<order order="WarehouseFleet|Assist|TradePerform|...">` with `<trade buyer=… seller=… partner=<ship> ware=…>`; list-typed order params (e.g. `targets`) are stored as numeric ids resolved via `<ref id="…">` blocks near the end of the file.
- Runtime debugging: the scheduler writes `debug_to_file` logs into a `WarehouseFleets` directory (one file per ship idcode, plus `scheduler`, `config`, `garbage-collector`); these only appear when the game is started with script log files enabled. Each candidate transport is logged with its type, importance, rating, amounts and cargo usage — the first place to look when a ware "isn't transported".
- Game data reference: vanilla/DLC files live in the `.cat/.dat` archives of the install (e.g. `libraries/wares.xml` for ware volumes, `libraries/scriptproperties.xml` for valid script properties). Savegames are gzipped XML in `~/.config/EgoSoft/X4/<id>/save/`.
- Release (from `release.md`): bump `version` in `content.xml`, zip the folder without `.git`, upload to Nexus.

## Architecture

**The scheduler lives in MD, not in the aiscript.** `aiscripts/order.warehousefleet.xml` defines the `WarehouseFleet` default order (all fleet parameters) but its loop only signals `player.entity` with `'RequestScheduling'` and then waits/docks. This design avoids a base-game bug where evaluating modded behaviours undocks ships. Subordinates use vanilla `Assist`; `aiscripts/order.assist.xml` is a diff that lets Assist run the WarehouseFleet script, and the scheduler always reads parameters from `$ship.defaultorder.$baseorder` for Assist ships.

**`md/warehousefleets.xml`** — main logic:
- `Config` cue: global defaults in `global.$WarehouseFleets.*` (edited through `md/warehousefleets_menu.xml`). Never reset; migrations are `<patch sinceversion>` blocks.
- `WarehouseFleets` main cue holds shared state: `$deliveries` (in-flight/recent transports), `$reservations` (`'$ware:station:incoming|outgoing'` → amount), `$backpressure` (anti ping-pong penalty for reverse flows over 20 min). Bumping its `version` + patch resets all runtime state.
- `Scheduler` is an instantiated cue per request, split into chained sub-cues with small delays (spreads load across frames; `SchedulerKillswitch` aborts after 30s): prepare → collect targets (`AddTradeTargets`, sectors expand to stations + build storages) → ware lists → backpressure/reservations (throttled) → push transports (warehouse → targets) → pull transports (targets → warehouse) → `AssembleTrip` for each → `FindMiningTrip` → pick best by `$rating` → create trade orders (or a `WarehouseFleetMine` order) and register deliveries.
- `FindPossibleTransports` is the core: matches provider sell offers with receiver buy offers and classifies each pair as `supply` (receiver in Supply Targets, Prio: Supply), `build` (player build storage, Prio: Build), `trade` (NPC partner; must be profitable and pass the warehouse offer's faction restriction, Prio: Trade), or — between own stations — by the ware's role at the non-warehouse end: `restock` (drone/ammo supplies offer), `consumer` (production resource), `producer` (product), all Prio: Own Stations (`prioStations`); anything else (warehouse↔warehouse) is `distribution`, balanced by fill ratio via `cargo.{ware}.target` (Prio: Distribute). Amount and `importance` are per type; `rating = importance × volume`.
- `minCargoUsage` is a volume threshold relative to the ship's capacity. Transports below it survive only if they complete a build/restock request or can piggyback on a viable trip for the same provider→receiver relation; `AssembleTrip` combines up to 3 wares per relation.
- A trip always starts/ends at the home warehouse; a push trip may be paired with a pull trip back, otherwise a `WarehouseFleetReturn` order is appended.

**Other pieces:** `order.warehousefleet.mining.xml` is a one-shot mining order (vanilla mining loops forever). `libraries/assignments.xml` + `ui/warehousefleet_map_uix.lua` + `ui.xml` implement the optional "subordinates of warehouse" grouping (custom `warehousefleet` assignment; the station never actually commands the ships). `UpdateFleetOrganization` cue applies that setting.

## Conventions / gotchas

- Many comments are in German; keep the existing style when editing nearby code.
- Aiscript params that change need a `<patch sinceversion>` and a bumped aiscript `version` so existing saves migrate.
- Remove list elements by index in reverse loops (`remove_value name="$list.{$idx}"`); `remove_from_list` by value proved unreliable.
- Table keys are built as `'$' + ...` strings.
- Verify properties against `scriptproperties.xml` before using them; unknown properties fail silently (use `@` to guard optional ones).
