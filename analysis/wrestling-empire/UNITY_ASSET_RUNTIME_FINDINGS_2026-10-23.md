# Wrestling Empire — Unity Asset Runtime Findings

Date: 2026-10-23

## New verified finding
The supplied `assets/bin/Data/data.unity3d` is a UnityFS bundle.

Header begins with:
- `UnityFS`
- bundle format version: `8`
- Unity player/editor version string: `6000.3.25f1`

This is consistent with the IL2CPP metadata version 39 found in `global-metadata.dat` and confirms that the supplied data belongs to a Unity 6-era runtime.

## Why this matters for character replacement
The character/costume metadata should be traced against Unity 6 asset/runtime structures rather than older Unity assumptions.

The supplied archive contains the key runtime data needed for the next stage:
- `data.unity3d`
- `Managed/Metadata/global-metadata.dat`
- `libil2cpp.so` supplied separately

## Current model-replacement chain
Verified metadata-level chain:

`Character.costume (0x70)`
→ `Costume`
→ `limb / texture / shape`

and:

`Character.costumeChange (0x80)`

The exact mapping from these fields to Unity `Mesh`, `SkinnedMeshRenderer`, `Material`, and `Texture2D` remains to be proven.

## Next target
The next useful step is asset-level identification inside the UnityFS bundle: determine which serialized Unity objects are meshes, skinned renderers, materials, and textures, then correlate those objects with the native code paths that consume the Costume data.

No proprietary game binary or original asset is copied into this analysis repository file; this document stores only analysis metadata.
