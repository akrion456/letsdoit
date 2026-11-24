# letsdoit

A single-file StarfallEx chip (`arcade_cabinet.txt`) that spawns a Prop2Mesh
arcade cabinet textured with `DefaultMaterial.png` (4096×4096) in both
**singleplayer** and **multiplayer** (visible to all players, not just the chip
owner).

## Files

| File | Role |
| --- | --- |
| `arcade_cabinet.txt` | **Single-file StarfallEx chip** (`--@shared`, runs on server + all clients) |
| `DefaultMaterial.png` | 4096×4096 cabinet texture |
| `arcadecabinet.obj` | Arcade cabinet 3D mesh |
| `arcadecabinet.mtl` | Reference only (not needed by the chip) |

## Setup

Only the **chip owner** needs the model and texture files on disk:

1. Copy `DefaultMaterial.png` and `arcadecabinet.obj` into your
   `garrysmod/data/` folder (or `garrysmod/data/sf_filedata/`).
2. Copy `arcade_cabinet.txt` into `garrysmod/data/starfall/` (or paste it into
   the Starfall editor).
3. Spawn `arcade_cabinet.txt` **once** as a Starfall Processor chip.
4. If your Starfall client permission settings prompt for access, press **E
   (Use)** on the chip to grant permissions.

The arcade cabinet spawns linked to the chip's position and angles, so moving
or rotating the chip moves the cabinet with it.

## How Singleplayer & Multiplayer Work

- **Single chip (`--@shared`):** A single Starfall Processor runs the `SERVER`
  block on the server and the `CLIENT` block on every connected player's
  client.
- **Singleplayer:** The owner's client reads `arcadecabinet.obj` and
  `DefaultMaterial.png` directly from `garrysmod/data/` with zero network
  streaming overhead, builds a `VertexLitGeneric` material, and applies the
  mesh and texture to the `sent_prop2mesh` controller.
- **Multiplayer (all players see it):**
  1. The server creates the `sent_prop2mesh` entity and links controller `1` to
     the chip.
  2. Because `file.readInGame` is restricted to the chip owner and Prop2Mesh's
     server-side Starfall API only accepts `.mdl` paths, the owner's client
     condenses `arcadecabinet.obj` (stripping unused normals/material lines in
     a CPU-quota-safe coroutine down to ~127 KB compressed) and streams it to
     the server via `net.writeStream`.
  3. Every other player's client (including late joiners) downloads the
     condensed `.obj` stream from the server and applies it locally via
     `ent:p2mSetControllerInfo(1, partlist)`, automatically waiting for
     `sent_prop2mesh`'s initial sync and re-applying if the entity leaves and
     re-enters PVS.
  4. When another player needs the 4096×4096 PNG texture, the server streams
     `DefaultMaterial.png` once (`compress = false` so the 22.6 MB PNG uses no
     LZMA CPU quota), and each non-owner client caches it in Starfall temp
     storage (`file.writeTemp` / `file.existsTemp`) so future spawns load the
     texture from disk with zero re-download.

## Orientation & UVs

- `arcadecabinet.obj` is **Y-up** while Garry's Mod is **Z-up**, so the part is
  rotated by `Angle(90, 0, 0)` and offset by `Vector(-21.5, 0, -1)` to center
  and ground the cabinet on the chip.
- The `.obj` UVs are already normalized to `0..1`, mapping 1:1 onto the
  4096×4096 `DefaultMaterial.png` texture.

## Requirements

- [StarfallEx](https://github.com/thegrb93/StarfallEx)
- [Prop2Mesh](https://github.com/ACF-Team/Prop2Mesh)
