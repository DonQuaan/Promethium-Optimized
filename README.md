<p align="center"><img src="logo.png" alt="Promethium Optimized" width="112"></p>

# PO — Promethium Optimized

> *One goal: make Minecraft run better than you thought possible.*

**Promethium Optimized (PO)** is a performance-focused Fabric modpack by **yangdawn (DonQuaan)** — an upgraded and reworked derivative of [Fabulously Optimized](https://github.com/Fabulously-Optimized/fabulously-optimized).

[![CurseForge](https://img.shields.io/badge/CurseForge-PO%20--%20Promethium%20Optimized-f16436)](https://www.curseforge.com/minecraft/modpacks/promethium-optimized) [![Latest release](https://img.shields.io/github/v/release/DonQuaan/Promethium-Optimized?label=latest)](https://github.com/DonQuaan/Promethium-Optimized/releases/latest)

## Versions

| Version | Minecraft | Fabric Loader | Mods | Date | Status | Download |
|---|---|---|---:|---|---|---|
| [1.0.0](https://github.com/DonQuaan/Promethium-Optimized/releases/tag/v1.0.0) | 1.21.11 | 0.18.4 | 111 | 2026-02-10 | Release | [`Promethium-Optimized-1.0.0.zip`](https://github.com/DonQuaan/Promethium-Optimized/releases/download/v1.0.0/Promethium-Optimized-1.0.0.zip) |
| [1.1.0](https://github.com/DonQuaan/Promethium-Optimized/releases/tag/v1.1.0) | 1.21.11 | 0.18.4 | 111 | 2026-03-01 | Release | [`Promethium-Optimized-1.1.0.zip`](https://github.com/DonQuaan/Promethium-Optimized/releases/download/v1.1.0/Promethium-Optimized-1.1.0.zip) |
| [1.2.0](https://github.com/DonQuaan/Promethium-Optimized/releases/tag/v1.2.0) | 1.21.11 | 0.18.4 | 89 | 2026-03-04 | Release | [`Promethium-Optimized-1.2.0.zip`](https://github.com/DonQuaan/Promethium-Optimized/releases/download/v1.2.0/Promethium-Optimized-1.2.0.zip) |
| [1.2.1](https://github.com/DonQuaan/Promethium-Optimized/releases/tag/v1.2.1) | 1.21.11 | 0.18.4 | 128 | 2026-04-03 | **Latest release** | [`Promethium-Optimized-1.2.1.zip`](https://github.com/DonQuaan/Promethium-Optimized/releases/download/v1.2.1/Promethium-Optimized-1.2.1.zip) |
| [1.2.2](https://github.com/DonQuaan/Promethium-Optimized/releases/tag/v1.2.2) | 1.21.10 | 0.19.2 | 111 | 2026-05-30 (est.) | ⚠️ Unreleased · broken | source only |
| [2.0.0-alpha.1](https://github.com/DonQuaan/Promethium-Optimized/releases/tag/v2.0.0-alpha.1) | 1.21.1 | 0.19.3 | 14 | 2026-07-11 | Alpha | [`Promethium-Optimized-2.0.0-alpha.1.mrpack`](https://github.com/DonQuaan/Promethium-Optimized/releases/download/v2.0.0-alpha.1/Promethium-Optimized-2.0.0-alpha.1.mrpack) |
| [2.0.0-alpha.2](https://github.com/DonQuaan/Promethium-Optimized/releases/tag/v2.0.0-alpha.2) | 1.21.1 | 0.19.3 | 26 | 2026-07-11 | Alpha | [`Promethium-Optimized-2.0.0-alpha.2.mrpack`](https://github.com/DonQuaan/Promethium-Optimized/releases/download/v2.0.0-alpha.2/Promethium-Optimized-2.0.0-alpha.2.mrpack) |
| [2.0.0-alpha.3](https://github.com/DonQuaan/Promethium-Optimized/releases/tag/v2.0.0-alpha.3) | 1.21.1 | 0.19.3 | 26 | 2026-09-28 | Alpha | [`Promethium-Optimized-2.0.0-alpha.3.mrpack`](https://github.com/DonQuaan/Promethium-Optimized/releases/download/v2.0.0-alpha.3/Promethium-Optimized-2.0.0-alpha.3.mrpack) |

- **Want to play now?** Use **1.2.1** (Minecraft 1.21.11) — the latest release, also on [CurseForge](https://www.curseforge.com/minecraft/modpacks/promethium-optimized).
- **2.0 (alpha)** is a clean rebuild on Minecraft 1.21.1: performance only, Sodium 0.6.13 pinned, managed with packwiz. This branch is where development happens.
- **1.2.2** was an experiment (VulkanMod + Voxy) that never shipped; its source is archived for history only.

Every version is a git tag — browse its exact source at `/tree/v<version>`, e.g. [`v1.2.1`](https://github.com/DonQuaan/Promethium-Optimized/tree/v1.2.1). Full history: [CHANGELOG.md](CHANGELOG.md).

## Install

| File | Prism Launcher | Official app |
|---|---|---|
| `.zip` (1.x, CurseForge format) | *Add Instance → Import* | CurseForge app: *Create Custom Profile → Import* |
| `.mrpack` (2.0, Modrinth format) | *Add Instance → Import* | Modrinth App: *Import from file* |

Mods are downloaded by the launcher from CurseForge/Modrinth; PO does not redistribute them. Exceptions, each allowed by its license: the CC0 *Translations for Sodium* resource pack in the 1.x zips (its CurseForge file was deleted upstream) and one MIT-licensed jar (*Remove Stardust Labs Intro Message*) in the 1.2.1 zip, as in the original release.

## Repository layout

| Path | Content |
|---|---|
| `pack/` | [packwiz](https://packwiz.infra.link/) pack: `pack.toml`, `index.toml`, one metadata file per mod in `pack/mods/` (project/file IDs + hashes) and the pack's own configuration |
| `PLAN.md`, `docs/` | Rebuild plan and benchmark method for 2.0 |
| `CHANGELOG.md` | Changes of every version, computed from the pack metadata |

Build the installable files yourself (needs [packwiz](https://packwiz.infra.link/installation/)):

```sh
cd pack
packwiz modrinth export     # 2.0 → .mrpack
packwiz curseforge export   # 1.x tags → CurseForge .zip
```

## Credits & licensing

- Built on and adapted from **Fabulously Optimized** (BSD-3-Clause, © 2020-2026 Fabulously Optimized Authors) — full notice in [`pack/THIRD-PARTY.md`](pack/THIRD-PARTY.md). Fabulously Optimized does not endorse this project.
- Every mod, shader and resource pack belongs to its author and keeps its own license.
- PO's own configuration, packwiz metadata and documentation: [BSD-3-Clause](LICENSE), © 2026 yangdawn (DonQuaan) — from 2.0.0-alpha.3 on; earlier tags carry no license file.
