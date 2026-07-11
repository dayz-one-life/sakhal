# Changelog

All notable changes to this project are documented here.
The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
### Changed
### Deprecated
### Removed
### Fixed
### Security

## [1.5.0] - 2026-07-10

### Added
- Server broadcast messages: `db/messages.xml` (ported from the chernarus server, adapted for Sakhal) — a new-player onboarding rotation (one-life ruleset + Discord invite) plus the restart countdown. Auto-loaded from `db/`.

### Changed
- Permadeath ruleset: enabled `disableRespawnInUnconsciousness` in `cfggameplay.json` — players can no longer respawn (suicide out) while unconscious, so going down commits you to the outcome. Matches the chernarus hardcore ruleset.
- Climate: switched to a **constant year-round cold** in `cfggameplay.json` — `environmentMinTemps`/`environmentMaxTemps` flattened to −10 °C min / −4 °C max every month, replacing Sakhal's default seasonal curve. Relative to the chernarus −6/−2 baseline: max is 2° colder, and min is 4° colder to reflect Sakhal's higher elevation.
- Base building: enabled **free-form placement** in `cfggameplay.json` — disabled all 14 `BaseBuildingData` placement/collision checks (10 `HologramData` + 3 `ConstructionData` + Sakhal's `disableColdAreaPlacementCheck`), so structures can be placed anywhere, including cold zones. Matches the chernarus server's build-anywhere ruleset. `disallowedTypesInUnderground` is unchanged.
- Timers / anti-hop: adopted the chernarus hardcore timer settings in `db/globals.xml` — removed server-hop and relog penalties (`TimeHopping`/`TimePenalty` = 0), shortened `TimeLogin` (15→5), and effectively disabled idle mode (`IdleModeCountdown` 60→3600, `IdleModeStartup` 1→0). The other 25 globals are unchanged.

## [1.4.1] - 2026-07-10

### Changed
- Infected density: increased the dynamic spawn counts (`dmin`/`dmax`) in `env/zombie_territories.xml` by 2 for every zone where the value was greater than 0 (781 attributes across 417 zones), raising the number of roaming infected. Zones with `dmin`/`dmax` of 0 (static-only spawns) and all `smin`/`smax` values are unchanged.

## [1.4.0] - 2026-07-10

### Added
- Custom spawn loadout: `custom/loadout.json` (ported from the chernarus server) referenced via `spawnGearPresetFiles` in `cfggameplay.json`. New characters spawn with a t-shirt, canvas pants, athletic shoes, a bandage, and a steak knife. Note: this is a light/temperate loadout with no cold-weather layers — players may chill quickly on Sakhal until it is warmed up for the map.

## [1.3.0] - 2026-07-10

### Changed
- Loot economy: halved the `nominal` and `min` values (round half up, floored at 0) for all `db/types.xml` entries — 1,902 of 1,955 — to roughly halve overall loot availability. Excludes the 35 `deloot="1"` (dynamic event loot) and 18 `Underground` items, which keep their original values.

## [1.2.0] - 2026-07-10

### Added
- Automated deployment: `.github/workflows/deploy.yml` uploads the mission config to the DayZ (Nitrado) server over FTP whenever a GitHub release is published, syncing only changed files. Requires the `FTP_SERVER`, `FTP_USERNAME`, `FTP_PASSWORD`, and `FTP_DIRECTORY` repository secrets. Until the `FTP_SERVER` secret is set, the deploy job skips gracefully (ends green with a notice) instead of failing, so releases stay clean before the Nitrado server is provisioned.

## [1.1.1] - 2026-07-10

### Fixed
- Solo maintainer mode: back-merge PRs (`main`→`develop`) are no longer blocked by the contribution CHANGELOG/CLAUDE.md gate. The solo `gh-pr-create` check now parses `--head` and exempts `head == productionBranch`, mirroring the merge handler. Backported from the workflow template (`dbd-net/project-template` #4).

## [1.1.0] - 2026-07-10

### Added
- Default DayZ Sakhal server configuration: baseline mission files from the official `dayzOffline.sakhal` set — economy core, spawnable/random presets, map group definitions, animal territory files, gameplay/weather/environment config, `init.c`, and `config.cpp`.
- Workflow initialization: stamped `dayz-one-life/sakhal` as the canonical repo and enabled solo maintainer mode in `.claude/workflow.json`.

## [1.0.0] - 2026-07-10

### Added
- `soloMaintainer` mode: an opt-in `.claude/workflow.json` flag that enables a `solo` guard role holding the union of contributor + maintainer permissions, so one person can run the full workflow (feature work, contribution merge, release, back-merge) from a single clone without swapping git remotes. Protected branches stay PR-only and contribution merges into `develop` still require `--squash` + a posted review (a `COMMENTED` review counts). Off by default.
