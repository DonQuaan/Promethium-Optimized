# Promethium Optimized 1.2.1

> Archived source of **PO – Promethium Optimized 1.2.1**. Current development lives on the default branch — see the [project README](https://github.com/DonQuaan/Promethium-Optimized#readme).

| | |
|---|---|
| Minecraft | 1.21.11 |
| Mod loader | Fabric Loader 0.18.4 |
| Contents | 128 mods · 2 resource packs · 2 shader packs |
| Released | 2026-04-03 on CurseForge — [original file](https://www.curseforge.com/minecraft/modpacks/promethium-optimized/files/7869304) |
| Status | Legacy 1.x line (superseded by the 2.0 rebuild) |

## Install

Download **`Promethium-Optimized-1.2.1.zip`** from the [GitHub release](https://github.com/DonQuaan/Promethium-Optimized/releases/tag/v1.2.1) (or the original CurseForge file above), then:

- **Prism Launcher:** *Add Instance → Import* → choose the zip.
- **CurseForge app:** *My Modpacks → Create Custom Profile → Import* → choose the zip.

Your launcher downloads every mod from CurseForge.

## Source layout

`pack/` is a [packwiz](https://packwiz.infra.link/) pack: `pack.toml`, `index.toml`, metadata files in `mods/`, `resourcepacks/`, `shaderpacks/` (project/file IDs + hashes, no binaries) and the pack's own configuration.

Rebuild the CurseForge zip yourself: `cd pack && packwiz curseforge export`.

Full mod list: [MODS.md](MODS.md).

## Differences from the original CurseForge file

- Shader packs are referenced by their CurseForge project/file instead of being bundled (the original zip bundled them; several are not licensed for redistribution). Same files, same versions.
- Mods that the original zip carried in `overrides/mods` are referenced by metadata (the MIT-licensed jar is re-bundled only in the release zip).
- `SodiumTranslations.zip` (Translations for Sodium) is taken from Modrinth — the identical file (same hash, CC0-1.0) was deleted from CurseForge by its author after release (file page returns 404), which breaks installing the original zip. It is bundled in the release zip.
- Removed runtime/personal files that are not part of the pack: `journeymap/data/ (41 files)`, `journeymap/icon/ (334 files)`, `config/resourceful-config-web.json`, `journeymap/journeymap.log` (`resourceful-config-web.json` held an auto-generated password; the mod regenerates it).

## Credits

Promethium Optimized is made by **yangdawn (DonQuaan)** as an upgraded and reworked derivative of [Fabulously Optimized](https://github.com/Fabulously-Optimized/fabulously-optimized) (BSD-3-Clause). See [pack/THIRD-PARTY.md](pack/THIRD-PARTY.md). Every mod belongs to its author and keeps its own license.
