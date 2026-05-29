# Promethium Optimized 1.2.2

> Archived source of **PO – Promethium Optimized 1.2.2**. Current development lives on the default branch — see the [project README](https://github.com/DonQuaan/Promethium-Optimized#readme).

| | |
|---|---|
| Minecraft | 1.21.10 |
| Mod loader | Fabric Loader 0.19.2 |
| Contents | 111 mods · 2 resource packs · 0 shader packs |
| Snapshot date | 2026-05-30 (estimated from file timestamps) |
| Status | ⚠️ **Unreleased, known broken** development snapshot — kept for history only |

## Why this snapshot is not installable

- It was never published on CurseForge; no installable file is provided.
- The instance runs Minecraft 1.21.10, yet 4 mod files are built for 1.21.11: `CrashAssistant-fabric-1.21.5-1.21.11-1.11.9.jar`, `Ixeris-4.4.0+1.21.11-fabric.jar`, `packetfixer-fabric-3.3.5-1.21.11.jar`, `Voxy World Gen V2-1.21.11-2.2.4.jar`.
- It replaced the Sodium/Iris render stack with VulkanMod + Voxy (`BorderlessWindowedVulkan-1.0.0+1.21.10.jar`, `Voxy World Gen V2-1.21.11-2.2.4.jar`, `voxy-0.2.9-alpha-1.21.10.jar`, `vulkanbobby-0.1.0+mc1.21.10.jar`, `VulkanMod_1.21.10-0.6.1.jar`) and was abandoned because of crashes and instability — the reason PO was rebuilt from scratch as 2.0 (see the audit in `PLAN.md` on the default branch).
- The instance had no `config/` folder, so none is included.

## Source layout

`pack/` is a [packwiz](https://packwiz.infra.link/) pack: `pack.toml`, `index.toml`, metadata files in `mods/`, `resourcepacks/`, `shaderpacks/` (project/file IDs + hashes, no binaries) and the pack's own configuration.

Full mod list: [MODS.md](MODS.md).

## Credits

Promethium Optimized is made by **yangdawn (DonQuaan)** as an upgraded and reworked derivative of [Fabulously Optimized](https://github.com/Fabulously-Optimized/fabulously-optimized) (BSD-3-Clause). See [pack/THIRD-PARTY.md](pack/THIRD-PARTY.md). Every mod belongs to its author and keeps its own license.
