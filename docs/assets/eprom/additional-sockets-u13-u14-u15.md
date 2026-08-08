# Batteries Included BusCard U13, U14, and U15 EPROM Sockets
## Purpose and relationship to the U1 firmware

## Purpose of this document

The Batteries Included BusCard contains one normally populated TMS2564 EPROM,
U1, and three additional TMS2564 sockets, XU13, XU14, and XU15. On the BusCard
examined during this project, only U1 was populated. The purpose of the three
additional sockets is therefore not immediately obvious from the surviving
board alone.

Reverse engineering of U1, examination of the BusCard schematic, and comparison
with the current MAME BusCard hardware model now allow the hardware role of
U13-U15 to be described with fairly high confidence. Their **exact original ROM
contents remain unknown**, but their electrical function and likely intended
software role are much clearer.

The principal conclusion is:

> **U13, U14, and U15 form an optional three-bank ROM facility that can replace
> the normal C64 BASIC ROM in the `$A000-$BFFF` address range. Together they
> provide 24K of physical ROM storage, but only one 8K bank is visible to the
> 6510 at a time. The available evidence strongly indicates that this facility
> was intended for an optional BASIC-related ROM package, distinct from the
> smaller BASIC 4.0-style command extension already contained in U1.**

This distinction between the U1 command extension and the optional U13-U15 ROM
facility is important.

## What the U1 disassembly established

U1 is an 8192-byte TMS2564 EPROM, but it is not used as one simple contiguous
8K program. The BusCard maps different parts of U1 into different parts of the
C64 address space. The reconstructed organization is:

| Physical U1 offset | CPU address | Principal contents |
| --- | --- | --- |
| `$0000-$0BFF` | `$A000-$ABFF` | Machine-language monitor and low-level BusCard I/O drivers |
| `$0C00-$0FFF` | `$EC00-$EFFF` | Resident C64 KERNAL overlay and BusCard dispatch code |
| `$1000-$113B` | `$B000-$B13B` | BASIC extension installer/uninstaller and associated data |
| `$113C-$147A` | `$B13C-$B47A` | Zero fill |
| `$147B-$1FFF` | `$B47B-$BFFF` | Encoded relocatable BASIC command extension |

The resident ROM provides three public entry points:

| BASIC command | Address | Function |
| --- | ---: | --- |
| `SYS 61000` | `$EE48` | Install/enable the BusCard BASIC 4.0-style command extension |
| `SYS 61003` | `$EE4B` | Remove/disable the command extension |
| `SYS 61006` | `$EE4E` | Enter the BusCard machine-language monitor |

The important discovery is that `SYS 61000` does **not** enable U13, U14, or
U15. Instead, U1 contains an encoded relocation stream. The installer expands
this code into RAM below BASIC's current `MEMSIZ`, then hooks four standard
BASIC indirect vectors. On a stock C64 with `MEMSIZ=$A000`, the reconstructed
runtime extension occupies `$955B-$9FFF` and is 2725 bytes long.

The relocated image identifies itself as `BASIC 4.0` and adds fifteen
Commodore-style disk commands:

```
CONCAT   DOPEN    DCLOSE   RECORD    HEADER
COLLECT  BACKUP   COPY     APPEND    DSAVE
DLOAD    CATALOG  RENAME   SCRATCH   DIRECTORY
```

This means that the BusCard already supplies a useful BASIC 4.0-style disk
command environment using **U1 alone**. The optional sockets are therefore not
required merely to make `SYS 61000` work, and they would not simply contain
three more pieces of the U1 command wedge.

## What the schematic says about U13-U15

The schematic shows four TMS2564 devices associated with the BusCard ROM
system:

- U1, controlled by `PD/!PGM1`;
- U13, controlled by `PD/!PGM2`;
- U14, controlled by `PD/!PGM3`;
- U15, controlled by `PD/!PGM4`.

U13, U14, and U15 share the same address and data buses. Each is an 8K x 8
EPROM, so the three sockets provide 24K of physical storage.

The crucial difference is their chip-selection logic. Two outputs from the
Intel 8255 PPI, `PB0` and `PB1`, feed one half of U10, a 74LS139 two-to-four
line decoder. Three decoder outputs are wired to the three optional EPROMs:

| 8255 bank bits | 74LS139 output | Selected EPROM |
| --- | --- | --- |
| `PB1 PB0 = 00` | `Y0` / `PD/!PGM4` | U15 |
| `PB1 PB0 = 01` | `Y1` / `PD/!PGM3` | U14 |
| `PB1 PB0 = 10` | `Y2` / `PD/!PGM2` | U13 |
| `PB1 PB0 = 11` | `Y3` | No U13-U15 EPROM is connected to this decoder output |

Thus U13-U15 are **banks**, not three simultaneously visible 8K pieces of a
24K linear address range.

A third 8255 signal, `PB3`, controls whether this optional ROM bank system is
enabled. MAME's BusCard implementation describes the PPI Port B bits as:

```
PB0  BASIC ROM bank bit 0
PB1  BASIC ROM bank bit 1
PB3  BASIC ROM enable
PB6  printer STROBE
PB7  DIP-switch select
```

Those names are consistent with both the schematic and the behavior recovered
from U1.

## The `$A000-$BFFF` BASIC-ROM window

The most revealing point is where the optional ROMs appear in the C64 memory
map. MAME models U13-U15 as mutually exclusive 8K images at the same CPU
address range:

```
$A000-$BFFF
```

That is the normal C64 BASIC ROM window.

The same address window is also used temporarily by U1. When BusCard firmware
needs its monitor, I/O drivers, or BASIC-extension installer, the hardware can
map U1 over `$A000-$BFFF`. When that operation finishes, the normal memory map
is restored.

The resulting arrangement can be summarized as follows:

| BusCard ROM state | `$A000-$BFFF` contents |
| --- | --- |
| BusCard firmware selected (`PB3=0` in the MAME model) | U1 |
| Optional BASIC-ROM mode, bank `10` | U13 |
| Optional BASIC-ROM mode, bank `01` | U14 |
| Optional BASIC-ROM mode, bank `00` | U15 |
| Optional BASIC-ROM mode, bank `11` | No optional EPROM selected; normal C64 BASIC can remain visible |

The precise logic also depends on the normal C64 `LORAM`/`HIRAM` memory-control
conditions. The table is intended to show the BusCard's ROM-selection concept,
not replace the complete C64 PLA/memory-map truth table.

This organization explains why four physical 8K EPROM positions can coexist
without requiring a 32K continuous cartridge address space: they are selected
at different times and share the same 8K CPU window.

## Why the unused `11` bank state matters

U10 is a two-to-four decoder, so `PB0` and `PB1` naturally provide four bank
states. Only three outputs are connected to U13-U15. The fourth state is not
used for another optional EPROM.

This is not an accidental curiosity. U1's normal PPI initialization uses a
Port B state whose low two bits are both set, while the BASIC-ROM-enable state
is also set. MAME likewise initializes the emulated card with bank value 3.
In that state none of the three optional ROMs is selected, allowing the stock
C64 BASIC ROM to remain available.

The four possible low-bit states therefore make good architectural sense:
three optional ROM banks plus a fourth selection that does not substitute an
optional ROM.

## Relationship to U1's BASIC 4.0-style extension

The most tempting interpretation of U13-U15 is that they must contain the
`BASIC 4.0` code mentioned by the BusCard manual and by `SYS 61000`. The U1
disassembly shows that this cannot be the whole story.

The standard U1 firmware already contains the code that `SYS 61000` installs.
That package is small enough to be encoded in the tail of U1 and relocated into
RAM. It extends the existing C64 BASIC 2.0 environment through the standard
BASIC vectors; it does not replace the entire BASIC ROM.

U13-U15, by contrast, are a hardware ROM-banking system capable of substituting
one of three 8K EPROMs directly into the C64 BASIC ROM address space. This is a
fundamentally different mechanism.

The evidence therefore points to two separate facilities:

1. **U1 BASIC 4.0-style command extension** -- a compact disk-command package
   stored in U1, expanded into RAM, and enabled by `SYS 61000`.
2. **U13-U15 optional BASIC-ROM facility** -- up to 24K of additional physical
   ROM, bank-selected into `$A000-$BFFF` by the 8255 and external decode logic.

They could coexist because the BusCard's resident `$EC00-$EFFF` KERNAL overlay
remains available independently of which program ROM is selected in the BASIC
window. An optional BASIC environment in U13-U15 could therefore still make
use of the BusCard's KERNAL interception and IEEE-488/parallel-printer support.

## What was U13-U15 most likely intended to contain?

The hardware evidence supports a **BASIC-related purpose** strongly:

- `PB0` and `PB1` are the ROM-bank select bits;
- `PB3` is explicitly identified in the MAME hardware model as `BASIC ROM
  enable`;
- the selected EPROM replaces the C64 BASIC ROM at `$A000-$BFFF`;
- the BusCard was advertised/documented as providing BASIC 4.0 functionality;
- three 8K devices provide enough storage for a substantially larger ROM-based
  language environment than the 2725-byte U1 command extension.

For these reasons, the most plausible interpretation is that Batteries
Included designed XU13-XU15 for an **optional banked BASIC ROM package**, very
possibly a more extensive BASIC 4.0 environment than the small disk-command
extension built into U1.

There are two closely related ways the sockets could have been used:

- as three banks belonging to one larger, bank-switched software package; or
- as up to three separately selectable 8K ROM images associated with the
  optional BASIC-ROM feature.

The circuit alone does not distinguish these possibilities. What it proves is
the banking mechanism and address window, not the software organization inside
missing EPROMs.

## What we should not claim

At present there are no known U13, U14, or U15 ROM dumps in the material used
for this reverse engineering. MAME deliberately declares the three images as
`unpopulated.u13`, `unpopulated.u14`, and `unpopulated.u15`, while the known U1
image is loaded normally as `0.9.u1`.

Consequently, the evidence does **not** establish any of the following:

- that U13-U15 were populated on every BusCard;
- that Batteries Included necessarily shipped a three-ROM set with the card;
- that the three ROMs contained an unmodified Commodore PET/CBM BASIC 4.0 ROM
  set;
- that all 24K formed one continuous logical program;
- the exact entry points, commands, bank-switching protocol, or revision of
  any software intended for those sockets.

In particular, it would be premature to identify the missing ROMs with a
specific Commodore BASIC 4.0 ROM set merely because the BusCard uses the phrase
`BASIC 4.0`. The U1 firmware demonstrates that Batteries Included already used
that term for its own compact C64 extension.

## Confidence of the present conclusions

| Conclusion | Confidence | Basis |
| --- | --- | --- |
| U13-U15 are three 8K TMS2564 ROM positions | Very high | Schematic and device type |
| They share the `$A000-$BFFF` CPU window rather than forming a directly visible 24K range | Very high | Decode logic and MAME hardware model |
| `PB0/PB1` select among U15, U14, and U13 | Very high | Schematic 74LS139 wiring and MAME bank mapping |
| `PB3` enables the optional BASIC-ROM selection mechanism | Very high | MAME PPI description and ROM decode behavior |
| The fourth bank code (`11`) leaves no U13-U15 EPROM selected | High | Unused decoder output plus MAME behavior |
| U13-U15 are separate from the U1 `SYS 61000` RAM-resident command extension | Very high | U1 disassembly and relocation analysis |
| The sockets were intended for a larger optional BASIC-related ROM facility | High, but inferential | Address window, control-signal role, capacity, and BusCard feature set |
| They contained an exact Commodore PET/CBM BASIC 4.0 ROM set | Unknown | No ROM dumps or definitive documentation presently available |

## What would settle the remaining questions

The most valuable discovery would be an original BusCard with one or more of
XU13-XU15 populated. Even photographs of EPROM labels could identify a product
or revision. Ideally the chips could be dumped and compared with known
Commodore and Batteries Included ROMs.

Other useful evidence would include:

1. an original option sheet, price list, dealer catalog, or advertisement that
   specifically mentions the three-ROM option;
2. a BusCard manual revision describing XU13-XU15 or ROM bank selection;
3. surviving EPROM images labelled U13, U14, or U15;
4. another BusCard firmware revision that contains code explicitly selecting
   banks 0, 1, or 2 for an optional language package;
5. execution traces from a populated board or a reconstructed ROM set in MAME.

Until such evidence is found, the best description is functional rather than
historical: **XU13-XU15 are sockets for three optional, bank-selected 8K ROMs in
the C64 BASIC address space, with strong evidence that the facility was
intended for BASIC-related expansion.**

## References

- BusCard project repository: <https://github.com/pdbuchan/buscard>
- Reverse-engineered EAGLE schematic: <https://github.com/pdbuchan/buscard/blob/main/docs/assets/eagle/buscard.sch>
- BusCard schematic PDF: <https://github.com/pdbuchan/buscard/blob/main/docs/assets/schematic.pdf>
- Original BusCard Rev A manual: <https://github.com/pdbuchan/buscard/blob/main/docs/assets/buscard_rev_a_manual.pdf>
- MAME BusCard implementation: <https://github.com/mamedev/mame/blob/master/src/devices/bus/c64/buscard.cpp>
- Companion U1 annotated 6502 disassembly: `buscard_u1_6502_annotated_listing.txt`
- Companion U1 ROM discussion: `buscard_u1_rom_discussion.md`

## Summary

U13, U14, and U15 are best understood not as missing pieces required by the
standard U1 firmware, but as an **optional banked ROM expansion subsystem**.
Each socket accepts an 8K TMS2564, and the three devices share the C64
`$A000-$BFFF` BASIC-ROM window. The 8255's `PB0` and `PB1` outputs select one of
the three EPROMs through a 74LS139 decoder, while `PB3` controls the BASIC-ROM
selection mode.

U1 itself already supplies the BusCard's monitor, low-level I/O support,
KERNAL interception, and a compact BASIC 4.0-style command package that is
relocated into RAM. U13-U15 therefore represent a second and more expansive
ROM facility. The strongest interpretation is that they were intended for an
optional banked BASIC environment, but without surviving ROM dumps or explicit
documentation the exact software that Batteries Included intended to place in
those sockets remains an open question.
