# Wrestling Empire — Costume / Character Asset Analysis

Snapshot: 2026-10-23

## Verified binary inputs
- `assets.zip` contains 68 entries.
- `global-metadata.dat`: 7,429,976 bytes.
- `libil2cpp.so`: 55,048,248 bytes.
- `libil2cpp.so` is ELF64 AArch64, Android shared library, stripped.
- Build ID: `d32d84554cf9e27ee55d1933a9d5702066310c74`.

## Character metadata findings
The metadata contains game-relevant identifiers including:
- `Character`
- `Characters`
- `wrestler`
- `costume`
- `costumeChange`
- `characterSet`
- `ReadCharacters`
- `ReadCharacterReference`
- `holdLimb`
- `limb`
- `texture`
- `shape`

Important: some occurrences of `Character`, `texture`, and related words belong to Unity/.NET framework or XML parser classes. They must not automatically be interpreted as the Wrestling Empire gameplay class. The game-specific `Character`/`Costume` interpretation is based on the previously decoded type/field metadata and must be cross-checked against method/type definitions.

## Previously decoded Character fields
The verified Character type was identified as type index 116 in the metadata analysis. Relevant field offsets previously recovered include:
- `headSize` 0x58
- `bodyMass` 0x5C
- `muscleMass` 0x60
- `armMass` 0x64
- `legMass` 0x68
- `costume` 0x70
- `scar` 0x78
- `costumeChange` 0x80
- `moveFront` 0xF0
- `moveBack` 0xF8
- `moveGround` 0x100
- `moveAttack` 0x108
- `moveCrush` 0x110
- `taunt` 0x118
- `stance` 0x128
- `anim` 0x144

These offsets are analysis results and should be treated as version-specific to the supplied binaries.

## Costume findings
A separate `Costume` type was previously identified with fields including:
- `id`
- `charID`
- `limb`
- `texture`
- `r`
- `g`
- `b`
- `flesh`
- `shape`

This makes `Costume` the next high-priority target for tracing character appearance.

## Current interpretation
The strongest currently supported appearance path is:

`Character`
→ `costume`
→ `Costume`
→ `charID / limb / texture / shape`

This does NOT yet prove that `texture` is the final Unity Texture object or that `shape` is a mesh identifier. Those relationships require native-code tracing.

## Next analysis targets
1. Recover the complete `Costume` type definition and all method definitions associated with it.
2. Identify every method that reads `limb`, `texture`, or `shape`.
3. Cross-reference those methods with native addresses in `libil2cpp.so`.
4. Determine whether `limb` selects body-part slots, renderer slots, or another internal enum/index.
5. Determine whether `shape` controls geometry, body proportions, or a shape preset.
6. Determine how `texture` reaches Unity rendering/material APIs.
7. Trace `costumeChange` to determine when appearance is rebuilt or swapped.
8. Only after this, identify the safest replacement point for a custom character model.

## Important caution
Do not assume a string occurrence in `global-metadata.dat` is a gameplay method. Unity/.NET framework strings are abundant in this binary. Type/method table context is required before assigning a string to Wrestling Empire gameplay code.
