# Road to Badlands Merge (Sakhal) — Implementation Plan

> **For agentic workers:** execute task-by-task (superpowers:subagent-driven-development or executing-plans). Mirrors the Chernarus merge; the transform scripts are already ported into `docs/tools/road-to-badlands/` (nerf excludes `Underground`).

**Goal:** Adopt Bohemia's Road to Badlands `dayzOffline.sakhal` as the new base, then re-apply our customizations (loot nerf, zed buff, preserved gameplay/globals/messages/loadout).

**Architecture:** Three logical commits — (1) adopt the vanilla upstream base while keeping our two customized config files, (2) loot-nerf `db/types.xml`, (3) zed-buff `env/zombie_territories.xml`. Then a CHANGELOG + CLAUDE.md commit. Transform scripts already present under `docs/tools/` (excluded from FTP deploy).

**Tech Stack:** DayZ mission config (XML/JSON), Python 3 (stdlib), `xmllint`, bash, git.

## Global Constraints

- Branch: `feature/road-to-badlands-merge` (already created off `origin/develop`; the spec + plan + scripts are already in the working tree uncommitted). Do NOT commit on `main`/`develop`.
- Upstream source: `../road-to-badlands-mission-files/dayzOffline.sakhal/` (relative to the `sakhal` repo root).
- **Loot nerf:** `nominal → ceil/2`, `min → ceil/2`, every `<type>` EXCEPT `deloot="1"` or `<usage name="Underground"/>`. (1985 types, 45 deloot, 18 Underground.)
- **Zed buff:** every `<zone>` with `dmax>0` gets `dmin+1`/`dmax+1`; skip `dmax=0`. Upstream `zombie_territories.xml` is **CRLF** — the script preserves it (`newline=""`); verify with `git diff --numstat`.
- Keep OURS (do not overwrite): `cfggameplay.json`, `db/globals.xml` (verified: no new upstream keys). `db/messages.xml`, `custom/loadout.json` are ours-only (absent upstream) — untouched by the copy.
- `cfgignorelist.xml` → vanilla (adopt upstream; drops our 5 flare entries).
- Commit trailer: `Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>`.

---

### Task 1: Adopt the Road to Badlands vanilla base

**Files:** overwrite every shared file from upstream EXCEPT `cfggameplay.json` and `db/globals.xml`.

- [ ] **Step 1 — safety check:** our keep/ours-only files match `origin/develop`:
```bash
cd /Users/steveharmeyer/Development/dayz-one-life/sakhal
git diff --quiet origin/develop -- cfggameplay.json db/globals.xml db/messages.xml custom/loadout.json && echo CLEAN || echo "WARN differs"
```
- [ ] **Step 2 — copy all upstream files over the repo except the two kept ones:**
```bash
NEW=../road-to-badlands-mission-files/dayzOffline.sakhal
(cd "$NEW" && find . -type f) | while read f; do
  case "$f" in ./cfggameplay.json|./db/globals.xml) continue;; esac
  mkdir -p "$(dirname "$f")"; cp "$NEW/$f" "$f"
done
echo copied
```
- [ ] **Step 3 — verify shape:** `git status --short` should show modified config files but NOT `cfggameplay.json` / `db/globals.xml`; `db/messages.xml` / `custom/loadout.json` absent from the list. No brand-new untracked config files with odd casing (all upstream paths match repo paths exactly — verified).
- [ ] **Step 4 — ignore list vanilla:** `grep -c Flaregun cfgignorelist.xml` → `0`.
- [ ] **Step 5 — xmllint sweep** on every changed `*.xml` (use `git diff --name-only origin/develop | grep '\.xml$'`); all pass.
- [ ] **Step 6 — commit** (`chore: adopt Road to Badlands sakhal base`). Also add the already-present `docs/` spec/plan/scripts in this or the changelog commit.

### Task 2: Loot nerf (`db/types.xml`)

Script already at `docs/tools/road-to-badlands/nerf_loot.py` (excludes `deloot="1"` + `Underground`).
- [ ] Fixture-test the core logic (ceil + both exclusions) on inline strings; must pass.
- [ ] `python3 docs/tools/road-to-badlands/nerf_loot.py db/types.xml`
- [ ] Verify: `xmllint` OK; `grep -c '<type name=' db/types.xml` unchanged (1985); a normal item halved; a `deloot="1"` item unchanged; an `<usage name="Underground"/>` item unchanged; `git diff` touches only `<nominal>`/`<min>` lines.
- [ ] Commit (`feat: re-apply loot nerf (Underground-excluded) to sakhal types.xml`).

### Task 3: Zed buff (`env/zombie_territories.xml`)

Script already at `docs/tools/road-to-badlands/buff_zeds.py`.
- [ ] Fixture-test (+1 normal, +1 on dmin=0/dmax>0, skip dmax=0); must pass.
- [ ] Record N = zones with `dmax>0`.
- [ ] `python3 docs/tools/road-to-badlands/buff_zeds.py env/zombie_territories.xml`
- [ ] Verify: `xmllint` OK; **CRLF preserved** (`grep -lq $'\r'`); a `dmax>0` zone shows +1; a `dmax=0` zone unchanged; `git diff --numstat` shows exactly `N N` (no line-ending flip).
- [ ] Commit (`feat: re-apply zed buff (+1) to sakhal zombie_territories.xml`).

### Task 4: CHANGELOG + CLAUDE.md + final sweep

- [ ] Fill `## [Unreleased]` in `CHANGELOG.md`: adopted RtB base; loot nerf re-derived (halve, round up, exclude `deloot="1"` + `Underground`); zed buff +2→+1 (skip `dmax=0`); removed cfgignorelist flare entries (vanilla); note preserved-as-is files and the `docs/tools/road-to-badlands/` scripts.
- [ ] Update `CLAUDE.md` (repository-layout note about `docs/tools/`), last before PR.
- [ ] Final `xmllint` sweep; commit.

## Independent verification (controller, after Tasks 2–3)

Run an `ElementTree` oracle: parse pristine upstream vs committed `db/types.xml`, recompute expected ceil-halved values honoring `deloot="1"` + `Underground` exclusions → expect 0 mismatches. Same for `zombie_territories.xml` (+1 where `dmax>0`, skip `dmax=0`, non-target attrs unchanged, CRLF preserved).

## Post-plan

`finishing-a-feature` → PR into `develop` → review (COMMENTED, solo) → squash-merge → `drafting-a-release` (major bump to **v2.0.0** to match Chernarus) → `cutting-a-release` (publishes + FTP deploy) → back-merge `main`→`develop`.
