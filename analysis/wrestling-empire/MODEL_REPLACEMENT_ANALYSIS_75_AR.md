# Wrestling Empire — Model Replacement Analysis 75 (2026-10-23)

## New local binary finding
I re-opened the tracked `assets.zip` in the current runtime and inspected all top-level opaque files larger than 100 KB, excluding the DEX file.

- 22 opaque files were found.
- All start with `00 49 41 50 02 00 00 00 08 07` (`IAP` appears after a leading zero).
- Their measured byte entropy is about 7.89–7.92 bits/byte, which is consistent with highly compressed or encrypted data; entropy alone cannot distinguish these cases.
- Four groups share their first 32 bytes: one group of 8 files and three groups of 2 files.
- Within those groups, the first differing byte is typically offset 87 or 89. This suggests a common structured prefix/header, but does not prove the header length or file format.
- File sizes range from 114,010 to 515,907 bytes among the opaque files inspected.

## Interpretation
The common prefix and high entropy justify treating these as a candidate custom/packed asset format. They are not confirmed UnityFS bundles, and there is not yet enough evidence to label them encrypted, compressed, or character-model data. No decryption key, decoded payload, mesh, or texture has been identified from this observation alone.

## Relation to character replacement
Previously verified metadata still points to:
`Character.costume -> Costume(id, charID, limb, texture, r, g, b, flesh, shape)`.

The available `data.unity3d` bundle does not contain ordinary Mesh (class 43) or SkinnedMeshRenderer (class 137) objects. Therefore, the opaque files are now a promising next analysis target, alongside native IL2CPP method tracing.

## Next steps
1. Compare full 87-byte/89-byte prefixes across all opaque files and determine whether the prefix contains lengths, identifiers, checksums, or cryptographic metadata.
2. Check for recognizable compression/container structures at offsets 8, 16, 32, 64, 80, 87, and 89; do not alter originals.
3. Correlate repeated prefix groups with file sizes and hashes to see whether they are versions/variants of one payload family.
4. Continue recovering the actual `Costume` field types and trace appearance-update methods.
5. Do not claim model replacement is ready until a payload yields verifiable mesh/texture structure and can be mapped to a wrestler.

## Confidence
- Opaque file signatures and byte statistics: HIGH (measured directly from the current archive).
- Custom-format hypothesis: MEDIUM.
- Files contain wrestler meshes/textures: UNCONFIRMED.
- Replacement implementation: NOT YET READY.
