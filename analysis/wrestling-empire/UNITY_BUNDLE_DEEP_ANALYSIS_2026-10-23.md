# Wrestling Empire — Deep Unity Bundle Analysis

Date: 2026-10-23

## UnityFS extraction
`assets/bin/Data/data.unity3d` is a valid UnityFS v8 bundle using Unity `6000.3.25f1`.

Header values verified:
- File size: 1,095,858 bytes
- Compressed block-info size: 268 bytes
- Uncompressed block-info size: 453 bytes
- Flags: 579 (`0x243`)
- Compression: LZ4HC
- Block-info is combined with directory info and requires 16-byte alignment before block data.

The bundle contains 18 compressed data blocks and 6 directory entries:
- `globalgamemanagers` — 462,052 bytes
- `Resources/unity_builtin_extra` — 782,244 bytes
- `globalgamemanagers.assets` — 173,328 bytes
- `sharedassets0.assets` — 17,628 bytes
- `sharedassets0.assets.resS` — 873,984 bytes
- `level0` — 3,608 bytes

All 18 LZ4/LZ4HC storage blocks were successfully decompressed locally. The resulting storage size is 2,309,236 bytes, matching the directory offsets/sizes.

## Character-related asset evidence
String inspection of `globalgamemanagers` found a large set of explicit headwear asset names:
`headwear/headwear01` through `headwear/headwear54` (with numbering gaps corresponding to the actual asset list).

String inspection of `globalgamemanagers.assets` found the following gameplay-related names:
- `Characters`
- `CharacterElement`
- `Character`
- `Costume`
- `Player_Costumes`

These are not merely generic Unity shader strings; they occur alongside the game's serialized/script asset data and are therefore useful targets for the next object-level analysis.

## SerializedFile findings
The extracted `.assets` files use SerializedFile format version 22 and target Android (platform ID 13). TypeTree definitions are stripped from these player-build files (`enableTypeTree = false`), so complete field-level object decoding requires using the IL2CPP metadata/native definitions or an external Unity type database.

`globalgamemanagers.assets` has type entries corresponding to Unity class IDs including 115 (MonoScript), 150 (AssetBundle), and 21 (Material). This explains why game script names such as `Character`, `Costume`, and `Player_Costumes` appear in the serialized data without the gameplay class itself being a native Unity class.

## Important new conclusion
The asset side now independently confirms that the game's build contains named character/costume data in Unity serialized content, while the IL2CPP side independently identifies `Character.costume`, `Character.costumeChange`, and the `Costume` fields `charID`, `limb`, `texture`, and `shape`.

Therefore the working model is now:

`Character data (IL2CPP)`
→ `Costume` fields
→ `Player_Costumes / character-related serialized assets`
→ Unity asset objects
→ final character appearance.

The exact mapping from `shape`/`texture` to Mesh/Material/Texture is still not proven. Because TypeTrees are stripped, the next step is native-code tracing plus object-level Unity metadata/type definitions.

## Hashed assets
The APK package also contains many ~114–146 KB files under `assets/` with opaque names. They do not begin with the normal `UnityFS` signature and therefore are not being treated as ordinary UnityFS bundles. Their custom header/packing format requires separate identification before extracting them.

## Modding implication
The strongest current route for replacing a wrestler model is to identify which serialized/native object consumes the `Costume.shape` and `Costume.texture` values, then replace or redirect that appearance resource while keeping the original game's proprietary assets private.

## Confidence
- UnityFS extraction: HIGH
- Unity version 6000.3.25f1: HIGH
- Character/Costume strings in serialized content: HIGH
- Headwear asset naming: HIGH
- Character → Costume IL2CPP relationship: HIGH
- `shape` = exact Mesh object: NOT YET PROVEN
- `texture` = exact Texture2D/Material object: NOT YET PROVEN
