# Wrestling Empire — Costume / Model Analysis Next Step

Date: 2026-10-23

## Verified input
- `global-metadata.dat`: 7,429,976 bytes, IL2CPP metadata v39.
- `libil2cpp.so`: 55,048,248 bytes, ARM64 Android ELF, stripped.
- `assets.zip`: 68 entries, including `data.unity3d`, `global-metadata.dat`, and Unity data/resources.

## Verified Character appearance path
The `Character` metadata type contains:
- `costume` at offset `0x70`
- `costumeChange` at offset `0x80`
- body-shape parameters such as `height`, `headSize`, `bodyMass`, `muscleMass`, `armMass`, and `legMass`.

A `Costume` type was identified with fields:
- `id`
- `charID`
- `limb`
- `texture`
- `r`
- `g`
- `b`
- `flesh`
- `shape`

This establishes a strong metadata-level link between a character and its costume data.

## Important interpretation limit
At this stage we must NOT assume that `texture` is a Unity Texture object or that `shape` is directly a mesh identifier. Their exact native/runtime representation still has to be confirmed from field type metadata and native-code references.

## Next technical targets
1. Resolve the complete `Costume` type definition and field types/offsets.
2. Resolve `Costume` methods and map their method definitions to native addresses in `libil2cpp.so`.
3. Trace native reads/writes involving the `Costume` fields, especially `limb`, `texture`, and `shape`.
4. Trace `Character.costumeChange` (`0x80`) to determine when/how costume data is applied.
5. Identify the renderer/material/mesh construction path used after costume data is loaded.
6. Cross-reference `shape` and `texture` with Unity asset data to determine which assets correspond to body parts, clothing, textures, or meshes.
7. Build a replacement strategy that changes character appearance without redistributing original proprietary game assets.

## Current confidence
- Character → costume relationship: HIGH.
- Costume fields exist: HIGH.
- `limb`, `texture`, `shape` are relevant to appearance: HIGH at metadata level.
- Exact meaning of each field in the runtime renderer: NOT YET PROVEN.
- Direct model/mesh asset mapping: NOT YET PROVEN.

## Goal
The analysis remains focused on character-model replacement in the user's Wrestling Empire modding project, not the separate X-Man project.
