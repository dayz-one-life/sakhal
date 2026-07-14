# CLAUDE.md

This repository holds the server configuration for a **DayZ Sakhal** server. It was created
from the Claude Code workflow template; the workflow below is enforced by committed hooks in
`.claude/` and streamlined by repo-level skills.

## Server configuration

The repository root contains the DayZ mission config, based on the official `dayzOffline.sakhal`
set:

- **Economy / loot:** `cfgeconomycore.xml`, `cfgspawnabletypes.xml`, `cfgrandompresets.xml`,
  `cfglimitsdefinition*.xml`, and `db/` (`types.xml`, `events.xml`, `globals.xml`, `economy.xml`).
  `db/globals.xml` also holds server timers (login/hop/relog penalties, idle mode).
- **Broadcast messages:** `db/messages.xml` — scheduled/onconnect server messages and the restart
  countdown. Auto-loaded from `db/` (no registration in `cfgeconomycore.xml` needed).
- **Map groups & spawns:** `mapgroup*.xml`, `mapcluster*.xml`, `cfgplayerspawnpoints.xml`,
  `cfgeventspawns.xml`, `cfgeventgroups.xml`.
- **Environment / AI territories:** `cfgenvironment.xml`, `cfgweather.xml`, and `env/` (per-animal
  and zombie territory files). Each `<zone>` carries a static spawn range (`smin`/`smax`) and a
  dynamic spawn range (`dmin`/`dmax`); a range set to `0`/`0` means that spawn mode is unused for
  that zone. Tune infected/animal density via these attributes.
- **Gameplay & entry points:** `cfggameplay.json`, `config.cpp`, `init.c`, plus effect/area and
  underground trigger config. Hardcore ruleset: permadeath (`disableRespawnInUnconsciousness`), a
  constant year-round cold climate (`environmentMinTemps`/`environmentMaxTemps` flat at −8/0 °C), and
  no ambient personal light (`PlayerData.disablePersonalLight`) so nights stay genuinely dark.
- **Spawn loadout:** `custom/loadout.json`, referenced from `cfggameplay.json` via
  `PlayerData.spawnGearPresetFiles`. Defines the gear new characters spawn with. Paths in
  `spawnGearPresetFiles` are relative to the mission root (`./custom/...`).
- **Base building:** `cfggameplay.json` `BaseBuildingData` runs **free-form placement** — all
  `HologramData`/`ConstructionData` `disable*Check` flags are `true` (build anywhere, including cold
  zones). `disallowedTypesInUnderground` still blocks specific kits underground.

Edits are vetted against the official DayZ config schemas; changes flow through the workflow below.

### Loot economy conventions

When rebalancing `db/types.xml` spawn counts (`nominal`/`min`) globally, two classes of item are
intentionally **exempt** and keep their upstream values:

- Items flagged `deloot="1"` in `<flags>` — dynamic event loot, tuned by event spawns rather than
  the ambient economy.
- Items with a `<usage name="Underground"/>` flag — Sakhal underground-facility loot.

The current baseline reflects a global ~50% reduction of `nominal`/`min` (round half up, never below
zero) applied to every non-exempt type.

## Deployment

`.github/workflows/deploy.yml` deploys the mission config to the DayZ (Nitrado) server over FTP.
It triggers on **GitHub release published** (i.e. every release cut by the workflow below) and
uploads only changed files via [`SamKirkland/FTP-Deploy-Action`](https://github.com/SamKirkland/FTP-Deploy-Action),
tracking sync state in `.ftp-deploy-sync-state.json` on the server. Repo-management files
(`.git`, `.github`, `.claude`, `*.md`, etc.) are excluded — only the mission config is uploaded.

**Required repository secrets** (Settings → Secrets and variables → Actions). Until the `FTP_SERVER`
secret is set, the deploy job **skips gracefully** — it ends green with a notice rather than failing,
so releases cut before the server is provisioned stay clean:

| Secret | Purpose |
| --- | --- |
| `FTP_SERVER` | Nitrado FTP host |
| `FTP_USERNAME` | FTP account username |
| `FTP_PASSWORD` | FTP account password |
| `FTP_DIRECTORY` | Server-side target dir for the mission files |

## On session start

A SessionStart hook injects a role-aware orientation. **Present that orientation to the
user at the start of a fresh session.**

## The workflow

1. All feature work happens on a **fork**, on a `feature/*` branch.
2. Updating this file (`CLAUDE.md`) is the **last step** before opening a PR.
3. `CHANGELOG.md` is updated on **every** PR.
4. PRs go into the canonical repo's **`develop`** branch.
5. Reviews are done in Claude Code and posted back to the contributor.
6. Approved PRs are **squash-merged** into `develop`.
7. Production releases go out via a **`develop` → `main`** PR.
8. Merging that PR **cuts a release** with notes.

## Skills

- Contributor: `starting-work`, `finishing-a-feature`.
- Maintainer: `reviewing-a-contribution`, `merging-a-contribution`, `drafting-a-release`, `cutting-a-release`.
- Setup: `workflow-setup` (run once).

## Guardrails (enforced by `.claude/hooks/guard.py`)

- No commits, pushes, or merges on `main`/`develop` (tag pushes and the one-time `workflow.json` setup commit are exempt).
- On a fork: PRs must target `develop` and require CHANGELOG.md + CLAUDE.md updates.
- On the canonical repo: feature work is blocked (fork instead). Fork contributions into `develop` must be squash-merged and approved; the maintainer's own same-repo release/back-merge PRs are exempt from that gate.
- Once the project is initialized (`workflow-setup` run), write/git actions are blocked unless the Superpowers plugin is installed.
- **Solo maintainer mode:** setting `soloMaintainer: true` in `.claude/workflow.json` activates a `solo` role that holds the union of contributor + maintainer permissions from a single clone (no remote swapping). Protected branches stay PR-only; contribution merges into `develop` still require `--squash` + a posted review (a `COMMENTED` review counts, since self-approval is impossible); release (`develop`→`main`) and back-merge (`main`→`develop`) PRs are exempt from the changelog/review gates. Off by default.

## Honest limitations

- Hooks only bind inside Claude Code; plain `git`/`gh` in a shell bypasses them.
- Superpowers/role detection are filesystem/remote heuristics; they fail with clear messages.
- Approved-review detection needs the canonical repo to be a real GitHub remote.

## Configuration

`.claude/workflow.json` holds `canonicalRepo`, branch names, the optional `soloMaintainer` flag (default `false`), and optional `commands.test`/`commands.lint`.
