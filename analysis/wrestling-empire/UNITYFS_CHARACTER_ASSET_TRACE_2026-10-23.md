# Wrestling Empire 1.8.0 — UnityFS + Character/Costume Asset Trace (2026-10-23)

## Scope
This note records verified local analysis of the supplied `assets.zip`, `global-metadata.dat`, and `libil2cpp.so` for Wrestling Empire Android 1.8.0. It does **not** copy proprietary game binaries/assets into this repository.

## 1. UnityFS bundle was successfully decompressed

`assets/bin/Data/data.unity3d` is a UnityFS bundle:

- Format: 8
- Unity version: `6000.3.25f1`
- File size: `1,095,858` bytes
- Compressed block-info size: `268`
- Uncompressed block-info size: `453`
- Flags: `0x243`
- Compression: LZ4HC (`flags & 0x3f == 3`)
- `0x40`: blocks/directory info combined
- `0x200`: block-info padding/alignment flag

Important correction: `0x243` does **not** contain `0x80`, so block/directory info is **not** at EOF. For this file the block-info data begins after the 16-byte-aligned header at offset `0x40` (64).

The block info decompresses successfully to 453 bytes. It describes 18 storage blocks and 6 directory nodes. After 16-byte alignment, the storage stream begins at offset `336` and all 18 blocks decompress exactly.

### Directory recovered

| Node | Offset in decompressed stream | Size | Flags |
|---|---:|---:|---:|
| `globalgamemanagers` | 0 | 462,052 | 4 |
| `Resources/unity_builtin_extra` | 462,052 | 782,244 | 4 |
| `globalgamemanagers.assets` | 1,244,296 | 173,328 | 4 |
| `sharedassets0.assets` | 1,417,624 | 17,628 | 4 |
| `sharedassets0.assets.resS` | 1,435,252 | 873,984 | 0 |
| `level0` | 2,309,236 | 3,608 | 4 |

Total decompressed storage: `2,312,844` bytes.

## 2. Serialized Unity files parsed

The extracted Unity serialized files use format version 22 and Unity `6000.3.25f1` metadata. The `endian` byte is 0, so metadata records are little-endian after the header.

The useful object-class inventory is:

- `globalgamemanagers.assets`: 1,181 objects — 1,179 class 115, one class 150, one class 21.
- `sharedassets0.assets`: 9 objects — class 28 (3), class 213 (3), class 150 (1), class 21 (1), class 48 (1).
- `Resources/unity_builtin_extra`: 48 objects — class 48 only.
- `level0`: 26 objects — class 1 (6), class 114 (5), class 224 (4), class 222 (3), class 4 (2), plus classes 20, 81, 104, 157, 196, 223.

There are **no class 43 Mesh or class 137 SkinnedMeshRenderer objects in this particular `data.unity3d` bundle**. This is an important negative finding: the wrestler meshes are not directly present as ordinary Mesh/SkinnedMeshRenderer objects in this small bundle.

## 3. Consequence for the character-model investigation

The absence of Mesh/SkinnedMeshRenderer objects here means we should not claim that `data.unity3d` contains the wrestler models. The character system is primarily represented in IL2CPP managed/native code, and the actual model data may be:

1. in another package/cache/asset source not present in this `assets.zip`,
2. generated/assembled at runtime from other data,
3. stored in a custom/encrypted format, or
4. loaded through a separate asset-loading path.

The extensionless files in `assets/` were also checked. Their first bytes begin with `00 49 41 50 02...` (`IAP` after the leading zero), rather than `UnityFS`; they therefore cannot be treated as ordinary UnityFS bundles without further format identification.

## 4. Character and Costume metadata remains confirmed

From `global-metadata.dat` v39:

- `Costume` type index: **53**
- `Character` type index: **116**

The relevant managed data path remains:

`Character -> costume -> Costume -> charID / limb / texture / shape`

The previously established `Character` field offsets include:

- `costume` = `0x70`
- `scar` = `0x78`
- `costumeChange` = `0x80`

The `Costume` type contains the previously identified fields:

- `id`
- `charID`
- `limb`
- `texture`
- `r`
- `g`
- `b`
- `flesh`
- `shape`

These names are metadata facts. We still do **not** equate `texture` with a Unity `Texture2D` object or `shape` with a Unity Mesh ID until the native code path proves that interpretation.

## 5. v39 metadata parser verification

The v39 header uses 3-value section records `(offset, size, count)`. Verified important sections:

- strings: offset `705,664`, size `1,221,419`, count `68,402`
- methods: offset `2,174,696`, size `1,870,740`, count `62,358`
- parameters: offset `4,318,668`, size `665,110`, count `66,511`
- fields: offset `4,983,780`, size `381,840`, count `38,184`
- type definitions: offset `5,942,560`, size `696,540`, count `9,165`
- images: offset `6,639,100`, size `2,664`, count `74`

For Unity 6 metadata v39, type-definition records in this file are 76 bytes and the variable-width index rules were verified against the record size.

Verified type-definition records:

### Costume (type 53)
- field start: `1186`
- field count: `9`
- method start: `982`
- method count: `35`
- metadata token: `0x02000016`

### Character (type 116)
- field start: `2119`
- field count: `69`
- method start: `1604`
- method count: `77`
- metadata token: `0x02000035`

## 6. Method names are obfuscated

The 35 Costume methods and 77 Character methods have obfuscated names in metadata (examples: `PFCOCIOEFBN`, `MFBNABEJFIE`, `LKHMEONAGGF`, etc.). Constructors remain named `.ctor`.

Therefore the next reliable step is **native method tracing**, not guessing method names from their obfuscated identifiers.

## 7. Native evidence already found

`libil2cpp.so` is stripped ARM64, but the Character constructor was previously resolved at native address:

`0x187BEFC`

A direct search found this address stored in a native data table around file offset `0x51AC38`. Nearby entries contain other code addresses around `0x187...` and `0x189...`. This is evidence that native method-pointer registration data is present in the stripped binary, even though dynamic symbols do not expose gameplay method names.

A separate native code region contains repeated loads from offsets `0x70` and `0x80` through an object reached from a managed wrapper. For example, instructions around `0x11BCC54` and following repeatedly access:

- `[x8, #0x70]`
- `[x8, #0x80]`

These offsets exactly match the established `Character.costume` and `Character.costumeChange` offsets. This is strong evidence that the character/costume data is being consumed by native-generated managed code paths. It is **not yet sufficient** to identify the final Unity renderer assignment function.

## 8. Current model-replacement conclusion

We now have three independently verified layers:

1. **Managed data model:** `Character.costume` and `Costume.texture/shape/...`.
2. **Native IL2CPP linkage:** Character constructor and code that accesses the `0x70/0x80` fields.
3. **Unity asset container:** the supplied `data.unity3d` bundle is fully parsed, but contains no ordinary Mesh/SkinnedMeshRenderer objects.

Therefore the investigation should move from generic bundle scanning to **native tracing of the code that consumes `Costume.texture` / `Costume.shape` and eventually reaches UnityEngine renderer/mesh/material APIs**.

## 9. Next target

The next concrete targets are:

- identify the 9 Costume field definitions and their actual IL2CPP type indices;
- identify Character methods that read/write field offsets `0x70` and `0x80`;
- map Character/Costume method-definition indices to native method pointers using the registration tables;
- trace calls from those methods toward `GameObject`, `Renderer`, `SkinnedMeshRenderer`, `Mesh`, `Material`, and `Texture2D` APIs;
- identify where the actual wrestler mesh/texture source is loaded if it is outside `data.unity3d`.

No claim is made yet that a specific `shape` value is a mesh ID or that a specific `texture` value is a Texture2D pointer.
