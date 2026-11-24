# letsdoit

A single-file StarfallEx chip that spawns a Prop2Mesh arcade cabinet
textured with `DefaultMaterial.png` (4096x4096).

## Files

| File | Role |
| --- | --- |
| `DefaultMaterial.png` | Texture (4096x4096) |
| `arcadecabinet.obj` | Model |
| `arcadecabinet.mtl` | Reference only (not used by the chip) |
| `arcade_cabinet.txt` | **The chip** — one file, contains both server and client code |

## Setup

1. Copy `DefaultMaterial.png` and `arcadecabinet.obj` into your
   `garrysmod/data/` folder.
2. Spawn `arcade_cabinet.txt` as a **server** Starfall chip.
3. Spawn `arcade_cabinet.txt` (the same file) as a **client** Starfall chip.
4. Grant the **"Create material from image"** permission when prompted.

The cabinet appears at the server chip's position.

> **Why spawn the same file twice?** A Starfall chip is either a server chip
> or a client chip. This one file contains both halves (`if SERVER ... elseif
> CLIENT ...`), so you spawn it once as server and once as client. In
> singleplayer you do both; the cabinet then appears for you.

## Singleplayer vs multiplayer

- **Singleplayer:** works fully. You see the textured cabinet.
- **Multiplayer:** the person who spawns the chips sees the cabinet. Other
  players will **not** see the mesh.

### Why other players don't see it in multiplayer

Prop2Mesh's Starfall API can only push **`.mdl` model paths** to the server
(`p2mPushModel`). It cannot push **`.obj`** data. The only way to get `.obj`
data onto the server is Prop2Mesh's raw upload net protocol
(`prop2mesh_upload_start` / `prop2mesh_upload`), but a Starfall chip cannot
send those messages — Starfall's `net` library only talks to other Starfall
chips (`SF_netmessage`), and Starfall does not expose `util.Compress` (which
the upload requires). So the mesh has to be built client-side, which only the
spawner's client does.

### For true multiplayer (everyone sees it)

Convert the model to a GMod `.mdl` and the texture to a `.vmt`, then use a
**server-only** chip. A `.mdl` path *can* be pushed server-side, so the server
stores it and syncs it to every client.

1. Convert `arcadecabinet.obj` → `arcadecabinet.mdl` (e.g. import into
   Blender, export `.smd`, compile with StudioMDL). Put it in
   `garrysmod/models/arcadecabinet.mdl`.
2. Make a material from `DefaultMaterial.png` (e.g. with VTFEdit → `.vtf` +
   `.vmt`), e.g. `garrysmod/materials/arcadecabinet/cabinet.vmt`.
3. Distribute the `.mdl` / `.vmt` / `.vtf` to everyone (workshop addon,
   FastDL, or a shared addon folder).
4. Spawn this **server-only** chip:

```lua
-- arcade_cabinet_mdl.txt  (spawn as a SERVER chip only)
local MODEL    = "models/arcadecabinet.mdl"
local MATERIAL = "arcadecabinet/cabinet" -- the .vmt name, no extension

local ent = p2m.create(1, chip():getPos(), chip():getAngles())
ent:p2mPushModel(1, MODEL, Vector(0, 0, 0), Angle(0, 0, 0))
ent:p2mBuild()
ent:p2mSetMaterial(1, MATERIAL)
```

This works in singleplayer and multiplayer, and every player sees the cabinet.

## How the .obj chip works

The server half creates the Prop2Mesh entity and applies the texture. The
client half creates the texture material from the PNG and builds the mesh from
the OBJ (via `p2mSetControllerInfo`). The two halves coordinate over Starfall
net. The OBJ is Y-up while GMod is Z-up, so the part is pitched +90° to stand
the cabinet upright; its UVs are already 0–1 normalized, so the 4096×4096
texture maps straight across the model.

## Requires

- [Prop2Mesh](https://github.com/ACF-Team/Prop2Mesh) (the `sent_prop2mesh`
  entity and its Starfall `p2m` library)
- [StarfallEx](https://github.com/thegrb93/starfallex)
