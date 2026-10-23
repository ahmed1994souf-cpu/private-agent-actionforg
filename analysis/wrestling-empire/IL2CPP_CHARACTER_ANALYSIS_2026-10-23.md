# Wrestling Empire — IL2CPP Character Analysis
## Snapshot 2026-10-23

This document records the verified analysis state for the Wrestling Empire project. It is analysis data only; original game binaries/assets are not copied into this text file.

### Source files

- `assets.zip`: 68 entries, 23,007,789 bytes uncompressed.
- `global-metadata.dat`: 7,429,976 bytes.
- `data.unity3d`: 1,095,858 bytes.
- `libil2cpp.so`: 55,048,248 bytes.
- SHA-256 `assets.zip`: `2c44a9d3efd8a0b1f2f23367f49ceb6b3ea09a226a9cc29a0b3685536212ecc6`
- SHA-256 `global-metadata.dat`: `22ff7f1421a3128efeda7eeccbd985a92346dcd9b12e05bf29ef7e24221883e8`
- SHA-256 `data.unity3d`: `261ad7e56435d493c2de547f47c64d7a40ebcbf811c7f7512b7cfe614c7491e7`
- SHA-256 `libil2cpp.so`: `08a31a15e7b64f7edac5e2d1998ea23d662fe413ab20e563d4182791003606ab`

### IL2CPP

- `global-metadata.dat` magic: `AF 1B B1 FA`.
- Metadata version identified: v39.
- `libil2cpp.so` is ELF64 for AArch64 (`e_machine = 0xB7`).

### Character type

Verified metadata type:
- Type name: `Character`
- Type index: `116`

Important instance fields and offsets:

| Field | Offset |
|---|---:|
| id | 0x10 |
| player | 0x14 |
| name | 0x18 |
| voice | 0x20 |
| role | 0x24 |
| heel | 0x28 |
| team | 0x2C |
| teamName | 0x30 |
| music | 0x38 |
| light | 0x3C |
| prop | 0x40 |
| pyro | 0x44 |
| musicSpeed | 0x48 |
| gender | 0x4C |
| height | 0x50 |
| age | 0x54 |
| headSize | 0x58 |
| bodyMass | 0x5C |
| muscleMass | 0x60 |
| armMass | 0x64 |
| legMass | 0x68 |
| costume | 0x70 |
| scar | 0x78 |
| costumeChange | 0x80 |
| stat | 0x88 |
| oldStat | 0x90 |
| newStat | 0x98 |
| health | 0xA0 |
| spirit | 0xA4 |
| injury | 0xA8 |
| injuryTime | 0xAC |
| dead | 0xB0 |
| absent | 0xB4 |
| worked | 0xB8 |
| news | 0xBC |
| worth | 0xC0 |
| contract | 0xC4 |
| salary | 0xC8 |
| clause | 0xCC |
| fed | 0xD0 |
| oldFed | 0xD4 |
| experience | 0xD8 |
| relationship | 0xE0 |
| negotiated | 0xE8 |
| agreement | 0xEC |
| moveFront | 0xF0 |
| moveBack | 0xF8 |
| moveGround | 0x100 |
| moveAttack | 0x108 |
| moveCrush | 0x110 |
| taunt | 0x118 |
| tauntWave | 0x120 |
| tauntHandshake | 0x124 |
| stance | 0x128 |
| location | 0x12C |
| x | 0x130 |
| y | 0x134 |
| z | 0x138 |
| platform | 0x13C |
| seat | 0x140 |
| anim | 0x144 |
| promo | 0x148 |
| promoVariable | 0x14C |
| home | 0x150 |
| angle | 0x154 |
| toilet | 0x158 |
| grudge | 0x15C |
| possessive | 0x160 |
| cuffed | 0x164 |

### Costume type

A separate `Costume` type was identified with fields:
- `id`
- `charID`
- `limb`
- `texture`
- `r`
- `g`
- `b`
- `flesh`
- `shape`

This is the main branch of investigation for character-model replacement.

### Character-related methods/strings

Metadata contains character/costume-related names and strings including:
- `ReadCharacters`
- `ReadCharacterReference`
- `Assigning character`
- `Assigning controller`
- `costumeChange`
- `characterSet`
- `headSize`
- `bodyMass`

Combat/move strings observed include:
- `Abdominal Stretch`
- `Cage Fight`
- `Elevated Crab`
- `Step Through Stunner`
- `Queen Of The Ring`
- `Impact_Ball`
- `Impact_Blade`
- `Impact_Hammer`

### Native-code correlation

The Character constructor was previously correlated to native ARM64 code at:
- `0x187BEFC`

The constructor writes values corresponding to several of the metadata-derived field offsets, providing a consistency check between metadata structure and `libil2cpp.so`.

### Next analysis targets

1. Resolve the complete `Costume` type and all of its methods.
2. Trace `charID -> Costume -> limb -> texture/shape`.
3. Identify the code responsible for constructing/updating character body parts.
4. Identify references to mesh/material/texture loading.
5. Map animation/stance fields to the character animation system.
6. Build a character replacement map without modifying the original game binaries.
