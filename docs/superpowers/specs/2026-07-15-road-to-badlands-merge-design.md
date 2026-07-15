# Road to Badlands merge — One Life Sakhal

**Date:** 2026-07-15
**Status:** Approved design, pre-implementation

## Goal

Incorporate Bohemia's *Road to Badlands* update into the One Life **Sakhal** server mission
config while preserving our customizations. Mirrors the Chernarus merge (see that repo's
`2026-07-15-road-to-badlands-merge-design.md`) with one map-specific difference: the loot
nerf excludes **`Underground`** items instead of `ContaminatedArea` (Sakhal's underground
bunkers are the analogous protected loot pool).

Source of the new files: `../road-to-badlands-mission-files/dayzOffline.sakhal/`

## Strategy

Adopt the new upstream (vanilla RtB) files as the base, then re-apply our customizations.
The CHANGELOG is the authoritative list of our customizations; all other files differ only
from version skew and are adopted wholesale.

## Per-file actions

| File | Action |
|------|--------|
| `db/types.xml` | Take upstream → **loot nerf**: halve `nominal`/`min` (round up, 1→1) on every type EXCEPT `deloot="1"` and `<usage name="Underground"/>` items |
| `env/zombie_territories.xml` | Take upstream → **zed buff**: `dmin`/`dmax` +1 on every zone with `dmax>0`, skip `dmax=0`. Upstream file is **CRLF** — preserve it (`newline=""`) |
| `cfggameplay.json`, `db/globals.xml` | Keep ours as-is (verified: upstream added no new keys) |
| `db/messages.xml`, `custom/loadout.json` | Ours-only (absent upstream); untouched |
| `cfgignorelist.xml` | **Vanilla** — adopt upstream, drop our 5 flare entries |
| All other shared config files | Adopt upstream wholesale |

## Transforms

Same deterministic rules as Chernarus, via scripts under `docs/tools/road-to-badlands/`
(ported from Chernarus; `nerf_loot.py` swaps `ContaminatedArea` → `Underground`). Both
scripts preserve source line endings and abort loudly if the matched element count differs
from the element count in the file.

- **Loot nerf:** `nominal → ceil(nominal/2)`, `min → ceil(min/2)`; skip `deloot="1"` or `<usage name="Underground"/>`.
- **Zed buff:** `dmin+1`, `dmax+1` where `dmax>0`; skip `dmax=0`.

## Facts (verified against RtB upstream)

- types.xml: 1985 types, 45 `deloot="1"`, 18 `Underground`.
- zombie_territories.xml is CRLF; types.xml is LF.
- `cfggameplay.json`/`globals.xml`: no new upstream keys → safe to keep as-is.
- No upstream-only config files to add; ours-only = `custom/loadout.json`, `db/messages.xml`.

## Verification

- `xmllint --noout` on every touched XML.
- Independent `ElementTree` oracle for both transforms (0 mismatches expected), confirming
  exclusions and CRLF/byte preservation.
- Diff review confirming only intended values changed.

## Release / workflow

Versioned change (current `v1.5.4`). `feature/*` → PR into `develop` (solo-maintainer),
then a `develop`→`main` release, major bump to **v2.0.0** to match the Chernarus release.
