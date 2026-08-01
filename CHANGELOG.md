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

## [3.2.0] - 2026-08-01

### Changed
- Base building: enabled "build anywhere" by setting every `disable*Check` boolean in `cfggameplay.json` `BaseBuildingData` to `true` — all eleven `HologramData` checks (bounding-box, player, roof-clipping, base-viability, geometry-plot, angle, placement-permitted, height, underwater, in-terrain, cold-area) and all three `ConstructionData` checks (roof, colliding, distance). Placement is now free-form: parts can be snapped into terrain, underwater, at any angle, and outside normally-buildable areas. `disallowedTypesInUnderground` is unchanged, so `FenceKit`, `TerritoryFlagKit`, and `WatchtowerKit` remain blocked underground.

## [3.1.0] - 2026-07-30

### Changed
- Loot economy: reset `db/types.xml` and `cfgrandompresets.xml` to plain vanilla Sakhal (the `road-to-badlands-mission-files` upstream base), reverting the 3.0.0 global loot nerf. All 1,985 types now match upstream `nominal`/`min` (1,024 values restored), and all 78 preset containers match upstream `chance` values (75 restored).
- Server timers: `db/globals.xml` now adopts the shared parent-directory copy — `FlagRefreshMaxDuration` drops from `3456000` (40 days) to `604800` (7 days), and `IdleModeCountdown` rises from `3600` (1 hour) to `21600` (6 hours). All other globals, including the custom `LootDamageMin` of `0.25`, are unchanged; `db/messages.xml` already matched its shared copy.

## [3.0.0] - 2026-07-30

### Changed
- Loot economy: halved `nominal` and `min` for **all 1,985** types in `db/types.xml` using a ceiling round (`ceil(n/2)`, so `9` → `5` and `1` stays `1`). Unlike the 2.0.0 pass, **no types are exempt** — the 45 `deloot="1"` (dynamic event loot) and 18 `<usage name="Underground"/>` items were reduced along with everything else, so the exemption convention documented in CLAUDE.md no longer describes the baseline.
- Loot economy: halved the preset-level `chance` on **all 78** containers in `cfgrandompresets.xml` (43 `<cargo>`, 35 `<attachments>`) — e.g. `foodHermit` `0.15` → `0.075`, `hatsFirefighter` `1.00` → `0.50`, `headtorches` `0.03` → `0.015`. Per-item `chance` values inside each preset are unchanged, and the three intentionally-disabled `glasses*` presets stay at `0.00`. Combined with the `db/types.xml` pass, ambient loot is roughly a quarter of the 2.1.1 baseline in both spawn count and container fill rate.
- Mission config: adopted the live server's copy of the mission folder as the new baseline. This reverts several customizations back to vanilla Sakhal values (see Removed).

### Removed
- Hardcore ruleset: the server copy drops the customizations that 2.0.0 established. `cfggameplay.json` sets `disableRespawnInUnconsciousness` back to `false` (permadeath off), `disablePersonalLight` back to `false` (ambient personal light returns, so nights are no longer fully dark), restores vanilla seasonal `environmentMinTemps`/`environmentMaxTemps` in place of the flat −8/0 °C year-round climate, and returns every `HologramData`/`ConstructionData` `disable*Check` flag to `false` (free-form base placement is gone; vanilla collision, roof, and base-viability checks apply again).
- Infected density: `env/zombie_territories.xml` reverts the 2.0.0 zed buff, lowering `dmin`/`dmax` by 1 across all 404 dynamic zones.
- Broadcast messages: `db/messages.xml` collapses the five-message onboarding rotation from 2.1.1 (PR #46) to a single message, `One Life is better with the app. https://dayzonelife.com`, with no `onconnect` delay.

### Fixed
- Loot economy: `LootDamageMin` in `db/globals.xml` is `0.25` (was `0.2`), so spawned loot carries at least 25% wear.

## [2.1.1] - 2026-07-20

### Changed
- Broadcast messages: synced `db/messages.xml` with the Chernarus server's version so both servers broadcast identical onboarding copy. The second message now points players at the website (`dayzonelife.com`, "earn and spend unban tokens, climb the leaderboard") instead of the Discord invite (`discord.gg/gdCdgmjhRe`). Message count, `onconnect` delay, and `repeat` interval are unchanged; the map name in the first message stays "Sakhal". Line endings normalized to CRLF to match the rest of the mission config.

## [2.1.0] - 2026-07-19

### Removed
- Custom spawn loadout: deleted `custom/loadout.json` and dropped the `PlayerData.spawnGearPresetFiles` key from `cfggameplay.json`, restoring vanilla Sakhal's structure (upstream omits the key rather than setting it empty). New characters now get DayZ's built-in default spawn gear (random t-shirt, canvas pants, athletic shoes, plus a bandage, chemlight, and fruit) instead of our ported chernarus preset. Note this does not change cold survivability — the built-in default is also a temperate loadout with no cold-weather layers.

## [2.0.1] - 2026-07-15

### Changed
- Loot economy: raised `LootDamageMin` in `db/globals.xml` from `0.0` to `0.2`, so spawned loot always carries at least 20% wear (`LootDamageMax` unchanged at `0.82`).

## [2.0.0] - 2026-07-15

### Added
- Merge tooling: `docs/tools/road-to-badlands/` transform scripts (`nerf_loot.py`, `buff_zeds.py`) plus the merge spec and plan under `docs/superpowers/`. Both scripts preserve source line endings byte-for-byte and abort if the matched element count differs from the file's element count. Documentation/tooling only — excluded from the FTP deploy.

### Changed
- Upstream base: adopted Bohemia's **Road to Badlands** `dayzOffline.sakhal` as the new mission-config base, then re-applied our customizations on top. All shared config files were taken from upstream wholesale; only version skew changed except where noted below.
- Loot economy: re-derived the loot nerf against the new base — halved `nominal` and `min` (round half up, floored at 1) for all `db/types.xml` types except the 45 `deloot="1"` (dynamic event loot) and 18 `<usage name="Underground"/>` items, which keep their upstream values (1,922 of 1,985 types nerfed).
- Infected density: re-applied the zed buff against the new base — increased dynamic spawn counts (`dmin`/`dmax`) by **1** for every zone with `dmax > 0` (404 of 417 zones); the 13 `dmax = 0` (static-only) zones and all `smin`/`smax` values are unchanged. CRLF line endings preserved. (Previous baseline was +2 against the old upstream; this re-derives +1 on the RtB base.)

### Removed
- Ignore list: dropped our 5 custom flare entries from `cfgignorelist.xml`, adopting the vanilla Road to Badlands version.

### Preserved (kept over upstream)
- `cfggameplay.json` and `db/globals.xml` kept as-is (verified: Road to Badlands added no new keys to either). `db/messages.xml` and `custom/loadout.json` are ours-only (absent upstream) and untouched.

## [1.5.4] - 2026-07-14

### Removed
- Broadcast messages: removed the scheduled server-restart countdown message from `db/messages.xml` (the `<shutdown>` entry, "#name will restart in #tmin minutes."). Only the two onconnect onboarding messages remain.

## [1.5.3] - 2026-07-14

### Changed
- Climate: raised the daytime ceiling of the constant year-round cold in `cfggameplay.json` — `environmentMaxTemps` −2 → 0 °C for every month. `environmentMinTemps` stays flat at −8 °C, so the cold is 2° less punishing at its warmest while nights are unchanged.

## [1.5.2] - 2026-07-13

### Changed
- Personal light: disabled the ambient player light in `cfggameplay.json` (`PlayerData.disablePersonalLight` `false` → `true`), so players no longer emit the faint glow that softly lit their surroundings at night. Reinforces the hardcore ruleset — nights are genuinely dark and require a real light source.

## [1.5.1] - 2026-07-11

### Changed
- Climate: warmed the constant year-round cold in `cfggameplay.json` by 2 °C across the board — `environmentMinTemps` −10 → −8 °C and `environmentMaxTemps` −4 → −2 °C for every month. Still a flat, seasonless cold, just 2° less punishing.

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
