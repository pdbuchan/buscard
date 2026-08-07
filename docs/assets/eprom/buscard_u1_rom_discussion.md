# Batteries Included BusCard U1 ROM
## General Discussion and Reverse-Engineering Notes

### Purpose of this document

This document accompanies the annotated 6502 assembler listing of the
Batteries Included BusCard U1 EPROM image, `buscardv0.9-tms2564.bin`.

The listing is a reverse-engineered description of the ROM rather than the
original Batteries Included source code. Symbol names, routine names, and
comments were reconstructed from the machine code, the BusCard schematic,
known Commodore 64 memory and KERNAL conventions, the BusCard implementation
in MAME, and comparison with published Commodore ROM disassemblies. Where the
function of a routine is not yet certain, the annotated listing deliberately
uses a neutral name rather than assigning a possibly incorrect historical
name.

The purpose of this discussion is to describe the overall architecture of the
firmware and explain how its major parts fit together. It is intended to be
read before, or alongside, the detailed assembler listing.

## ROM identity

The U1 image is 8192 bytes long, corresponding exactly to one TMS2564 8K x 8
EPROM. Its checksums are:

```
CRC32:   175e8c96
SHA-1:   8fb4ba7e3d0b58dc01b66ef962955596f1b125b5
SHA-256: c96f1b19aacd3f05fcd539c76dd476613d680c0ef03d7c26725a7e40a99e851a
```

The CRC32 and SHA-1 values match the BusCard U1 ROM identified by MAME as
`0.9.u1`. This provides useful independent confirmation that the image being
disassembled is a known BusCard firmware revision rather than an altered or
partially damaged dump.

## U1 is not a simple contiguous 8K program

The most important fact to understand when reading the listing is that U1 is
not used as one uninterrupted program occupying a single 8K address range.
The BusCard's address-decoding circuitry exposes different portions of the
physical EPROM at different locations in the Commodore 64 address space.

The organization recovered from the ROM and hardware is:

| Physical U1 offset | CPU address | Principal contents |
| --- | --- | --- |
| `$0000-$0BFF` | `$A000-$ABFF` | Machine-language monitor and BusCard low-level I/O drivers |
| `$0C00-$0FFF` | `$EC00-$EFFF` | Resident C64 KERNAL overlay and BusCard dispatch code |
| `$1000-$113B` | `$B000-$B13B` | BASIC 4.0 extension installer/uninstaller and associated data |
| `$113C-$147A` | `$B13C-$B47A` | 831 bytes of zero fill |
| `$147B-$1FFF` | `$B47B-$BFFF` | Encoded relocatable BASIC 4.0 command extension |

This mapping is why a conventional disassembly that simply assumes the whole
8K ROM occupies `$A000-$BFFF` gives misleading addresses for a significant
portion of the firmware. In particular, physical offsets `$0C00-$0FFF` are
intended to appear at `$EC00-$EFFF`, where they replace part of the normal C64
KERNAL address space.

The annotated listing therefore shows both the physical EPROM offset and the
CPU address at which each byte is intended to execute.

## Public entry points

Three consecutive public entry points are present in the resident ROM area:

| BASIC command | Address | Function |
| --- | ---: | --- |
| `SYS 61000` | `$EE48` | Install/enable the BusCard BASIC 4.0 command extension |
| `SYS 61003` | `$EE4B` | Remove/disable the BASIC 4.0 command extension |
| `SYS 61006` | `$EE4E` | Enter the BusCard machine-language monitor |

These are implemented as three-byte JMP instructions, which explains why the
entry points are spaced three bytes apart.

The three entry points also provide a useful high-level description of the
U1 firmware. U1 is not merely an IEEE-488 driver ROM. It contains three major
software facilities:

1. low-level BusCard I/O support and C64 KERNAL interception;
2. a machine-language monitor/assembler/disassembler; and
3. a relocatable BASIC command extension providing BASIC 4.0-style disk
   commands.

## Machine-language monitor

The first major component of U1 is a substantial machine-language monitor at
approximately `$A000-$A7D2`.

`SYS 61006` does not simply jump directly into the monitor. The firmware first
installs `$ED8B` in the C64 BRK vector at `$0316/$0317`, enables KERNAL
messages, and deliberately executes a `BRK` instruction. The BusCard BRK
handler then maps the appropriate U1 ROM region and transfers control to
`$A000`.

This mechanism allows the monitor to enter with a normal 6502 BRK stack frame.
The monitor removes that frame and records the interrupted processor state,
including:

- program counter;
- processor status register;
- accumulator;
- X register;
- Y register; and
- stack pointer.

The saved program counter is adjusted by two bytes so that it identifies the
actual `BRK` instruction rather than the address following it.

The reconstructed monitor command table contains the following commands:

| Command | Function |
| --- | --- |
| `R` | Display/edit saved registers |
| `M` | Display or modify memory |
| `G` | Go/execute from a specified address |
| `X` | Exit the monitor |
| `L` | Load memory from a device/file |
| `S` | Save memory to a device/file |
| `T` | Transfer a block of memory |
| `F` | Fill memory |
| `H` | Hunt/search memory |
| `D` | Disassemble machine code |
| `P` | Print disassembly |
| `A` | Assemble a line of 6502 code |
| `:` | Memory-edit line |
| `;` | Display/edit helper |
| `,` | Assembly/edit continuation helper |

The monitor is therefore much more than a simple memory viewer. It contains a
6502 disassembler and assembler as well as the usual memory manipulation,
execution, load/save, and register facilities expected of a serious
machine-language monitor of the period.

## The resident KERNAL overlay

A particularly interesting feature of the BusCard design is the use of U1 as
a partial replacement for the C64 KERNAL in the `$EC00-$EFFF` region.

Large portions of the beginning of this ROM section are deliberately identical
or very close to the stock Commodore KERNAL. They include keyboard/editor
data, VIC-II defaults, the `LOAD` and `RUN` text, and screen-line tables. This
is necessary because when the BusCard hardware overlays this address range,
ordinary KERNAL data that normally occupies the same addresses must remain
available.

The important changes begin in the KERNAL serial-device routines. At `$ED09`
the normal TALK/LISTEN code begins in the expected manner, but at `$ED0E` the
BusCard replaces the normal continuation with:

```
JMP $ED23
```

This transfers control to BusCard-specific dispatch logic rather than allowing
the stock Commodore serial-bus routine to continue unchanged.

The result is a relatively elegant interception scheme. Programs can continue
to use the standard C64 KERNAL I/O entry points, while the BusCard decides
whether a particular request should be handled by its IEEE-488 interface, its
parallel-printer interface, or an appropriate compatible C64 serial path.

This also explains why the firmware contains a mixture of code that is
byte-for-byte Commodore KERNAL material, lightly modified KERNAL entry paths,
and wholly new BusCard code.

## Intel 8255 PPI and the BusCard hardware

The BusCard uses an Intel 8255 programmable peripheral interface at
`$DEC0-$DEC3`. The firmware accesses these registers repeatedly in the
low-level I/O section.

The recovered functions of the ports are:

| Address | 8255 register | BusCard use |
| --- | --- | --- |
| `$DEC0` | Port A | IEEE-488 DIO0-DIO7; also Centronics printer D0-D7; DIP-switch input when selected |
| `$DEC1` | Port B | ROM-bank control, BASIC-ROM enable, printer STROBE, and DIP-switch selection |
| `$DEC2` | Port C | IEEE-488 handshake/control signals and printer BUSY |
| `$DEC3` | Control register | 8255 mode control and bit set/reset operations |

Port A is consequently shared between two principal external data paths: the
8-bit IEEE-488 data bus and the 8-bit parallel-printer data bus. Port C carries
much of the associated handshaking, while Port B performs several control
functions that include ROM selection as well as printer and switch control.

The low-level code beginning near `$A7D3` contains the detailed handshake and
routing machinery. The annotated listing identifies accesses to the 8255 and
many higher-level relationships, but some of the smallest helper routines
retain neutral names such as `BUSCARD_IO_ROUTINE_A80A`. The exact semantic
role of every individual handshake helper has not yet been proven closely
enough to justify assigning all of them historical-sounding names.

## IEEE-488, printer, and normal C64 device routing

The firmware is structured so that applications continue to use familiar C64
KERNAL device operations. The BusCard intercepts selected KERNAL paths and
examines the current device/interface configuration before routing the
operation.

This is an important architectural point. The BusCard does not require every
application to know how to manipulate the IEEE-488 bus directly. Instead, it
inserts itself beneath the normal C64 I/O conventions and provides alternate
hardware implementations of those operations.

This arrangement is appropriate for the BusCard's intended use with IEEE-488
Commodore peripherals, such as the 4040 dual disk drive, while retaining
support for a parallel printer and ordinary C64 device behavior where
appropriate.

The firmware also saves and restores C64 memory-configuration and BusCard
control state around many of these operations. Routines such as
`MAP_BUSCARD_ROM` and `RESTORE_MEMORY_MAPPING` are therefore central to the
implementation: BusCard ROM must be visible while its own code executes, but
the machine must subsequently be returned to the expected C64 memory map.

## BASIC 4.0-style command extension in U1

U1 contains a BASIC extension that adds a collection of disk-oriented commands
to the standard C64 BASIC environment. This is the facility installed by
`SYS 61000` and removed by `SYS 61003`.

The terminology deserves some care. The U1 ROM undeniably identifies the
resident extension with the string `BASIC 4.0`, and it supplies a set of
BASIC 4.0-style disk commands. This small extension should not automatically
be equated with the optional 24K of ROM associated with sockets XU13-XU15.
The U1 extension is a compact command package that relocates into RAM; the
three additional 8K EPROM sockets constitute a separate, much larger optional
ROM facility.

### Why the extension is stored in encoded form

The tail of U1, physical offsets `$147B-$1FFF`, cannot be sensibly
interpreted as ordinary 6502 instructions in place. It is an encoded
relocation stream.

When the extension is installed, the routine at `$B03C` starts just beyond the
end of the encoded data and walks backward through the source. At the same
time it walks the C64 BASIC `MEMSIZ` pointer downward, constructing the runtime
extension immediately below the current top of BASIC memory.

Most encoded bytes are copied literally. A zero byte introduces special
handling: an escaped zero represents a literal zero, while another form
contains a 16-bit relocatable address offset. The installer adds the original
`MEMSIZ` value to these offsets before storing the resulting absolute address.
This allows the package to execute correctly even when the top of BASIC memory
is not fixed at one particular address.

The installer also checks the BASIC variable boundary while copying. If the
new code would collide with existing BASIC variables, the installation is
aborted and the original memory pointers are restored.

### Runtime location on a stock C64

For a normal C64 with BASIC `MEMSIZ=$A000`, decoding the stream with the ROM's
own relocation rules produces a 2725-byte runtime image occupying:

```
$955B-$9FFF
```

The encoded stream contains 154 relocation records. The annotated assembler
listing includes a separate *derived* disassembly of this runtime image.
Those derived lines do not represent additional bytes in U1; they show what
the encoded bytes become after the installer has expanded and relocated them
in RAM.

### BASIC vector interception

After relocation, the installer changes four standard BASIC indirect vectors
at `$0304-$030B`:

- ICRNCH -- BASIC tokenizer/crunch hook;
- IQPLOP -- BASIC LIST/detokenizer hook;
- IGONE -- BASIC statement-dispatch hook; and
- IEVAL -- BASIC expression-evaluation hook.

The first twelve bytes of the relocated block are four three-byte JMP
instructions, one for each of these hooks. Immediately following them is the
NUL-terminated signature:

```
BASIC 4.0
```

Through these four vectors, the BusCard can extend BASIC without replacing the
entire C64 BASIC ROM. Normal BASIC syntax and execution continue through the
stock ROM when the BusCard extension has nothing special to handle.

### Additional BASIC keywords

The decoded runtime image contains a Commodore-style keyword table. The high
bit of the last character of each keyword marks the end of the keyword, in the
same general style used by Commodore BASIC ROMs.

Fifteen additional commands are present:

```
CONCAT
DOPEN
DCLOSE
RECORD
HEADER
COLLECT
BACKUP
COPY
APPEND
DSAVE
DLOAD
CATALOG
RENAME
SCRATCH
DIRECTORY
```

The extension tokenizer recognizes these commands and assigns them additional
BASIC token values beginning at `$CC`. The LIST hook performs the reverse
operation, translating those tokens back into their textual keyword names
when a BASIC program is listed.

A handler table then dispatches the tokens to the appropriate routines. Of
particular interest, `CATALOG` and `DIRECTORY` share the same handler, making
them alternate names for the same operation.

Several handlers ultimately use ordinary C64 KERNAL file operations after
constructing the required disk command or filename syntax. This reinforces
the overall BusCard design philosophy: extend the standard C64 software
interfaces and route their actual I/O through the BusCard hardware rather than
creating an entirely separate programming environment.

### Removing the extension

`SYS 61003` restores the four standard C64 BASIC vectors. The uninstaller also
checks for the `BASIC 4.0` signature in the relocated RAM image, restores the
saved pre-installation `MEMSIZ`, and synchronizes BASIC's related memory
pointers.

Thus installation is not simply a matter of turning on a hardware ROM bank.
For the U1 command extension, code is actually expanded into RAM and BASIC's
memory boundary is lowered to reserve space for it.

## Relationship to the optional U13, U14, and U15 EPROMs

The U1 disassembly helps clarify an apparent ambiguity in the BusCard
architecture.

The standard U1 firmware already provides BASIC 4.0-style disk commands. That
is the compact relocatable extension described above. The schematic, however,
contains sockets XU13, XU14, and XU15 for three additional TMS2564 EPROMs,
totaling 24K. Their selection is controlled by BusCard hardware and by 8255
Port B bank-control bits.

The existence of the U1 extension therefore does **not** mean that U13-U15
would merely duplicate the same code. The evidence instead points to two
distinct facilities:

- U1 supplies the standard BusCard firmware, monitor, device drivers, KERNAL
  patches, and a relatively small RAM-resident set of BASIC 4.0-style disk
  commands.
- U13-U15 provide the hardware capacity for a separate optional 24K ROM BASIC
  4.0 facility.

Precisely what software was supplied in the optional three-ROM set cannot be
determined from the U1 image alone. Until original U13-U15 dumps or their
specific documentation are located, it is preferable not to claim that they
contain an exact copy of any particular Commodore PET/CBM BASIC ROM set.

## Some notable implementation techniques

Several techniques in U1 are representative of carefully written 6502
firmware of its period.

### Computed dispatch with RTS

Both the monitor and BASIC extension use tables containing handler addresses
stored as `address - 1`. The low and high bytes are pushed on the stack and an
`RTS` instruction is executed. Because `RTS` increments the pulled address by
one, control arrives at the desired handler. This provides a compact computed
subroutine-dispatch mechanism on a processor without an indirect JSR
instruction.

### Partial ROM overlay rather than wholesale replacement

The `$EC00-$EFFF` region preserves the stock bytes that must remain visible and
changes only the portions necessary for BusCard interception. This reduces the
amount of Commodore behavior that the board needs to reimplement.

### Relocatable RAM extension

The BASIC command package is not tied to one absolute RAM address. Encoding
absolute references as offsets from the original BASIC `MEMSIZ` permits the
installer to place it immediately below the current top of BASIC memory. For a
stock C64 that is `$955B-$9FFF`, but the design itself is more general.

### Integration through standard vectors

Rather than replacing BASIC wholesale, the extension hooks four published or
well-known BASIC indirect vectors. Likewise, the hardware drivers intercept
standard KERNAL I/O paths. This keeps much of Commodore's existing BASIC and
KERNAL machinery in use.

## How to read the annotated assembler listing

A typical instruction line in the companion listing has the form:

```
CPUADDR [ROM+offset] bytes       instruction              ; comment
```

For example, `ROM+0D0E` identifies a physical byte location in U1, while the
CPU address at the left shows where the instruction is intended to execute.
This distinction is essential because of the BusCard's unusual ROM mapping.

Data is represented with `.byte`, `.word`, or `.res` rather than being forced
through the 6502 instruction decoder. Long verified runs of zero bytes are
shown with `.res` for readability, but every byte of the original 8192-byte
EPROM is accounted for.

The final portion of the listing is headed as a *derived runtime image*. It is
not part of the EPROM address space. It is a reconstruction of the BASIC 4.0
extension after the ROM's relocation algorithm has processed the encoded data
for the normal `MEMSIZ=$A000` case.

Names beginning with descriptive terms such as `BASIC4_`, `MON_`,
`KERNAL_`, and `BUSCARD_` were assigned during reverse engineering; they are
not original Batteries Included source labels.

## Limits of the present reverse engineering

A great deal of U1 can be identified with high confidence because it connects
to standard C64 vectors, contains recognizable strings or tables, or performs
clearly identifiable hardware accesses. Some areas remain less certain.

In particular, the low-level IEEE-488 and parallel-printer code contains
numerous small handshake routines whose broad purpose is clear from the 8255
signals and their callers, but whose exact individual semantic names have not
all been established. The annotated listing intentionally preserves generic
labels in these cases.

Similarly, the U1 ROM does not by itself disclose the contents of the optional
U13-U15 EPROMs. Hardware behavior can tell us how those ROMs were intended to
be selected, but not the bytes that Batteries Included originally supplied in
them.

The listing should therefore be regarded as a documented reverse-engineering
work in progress, with a distinction between observations directly supported
by the ROM and hardware and interpretations inferred from context.

## Further areas for investigation

Several useful extensions to this work would be possible:

1. Trace each low-level IEEE-488 state transition against the schematic and
   IEEE-488 handshake sequence, allowing the remaining generic I/O routine
   names to be replaced with more precise functional names.
2. Identify the exact role of each BusCard DIP switch in the firmware and
   correlate each branch of the interface-selection code with the documented
   switch settings.
3. Compare the BASIC 4.0 command handlers in detail with Commodore PET/CBM
   BASIC 4.0 implementations to distinguish copied concepts from independent
   Batteries Included implementations.
4. Locate surviving U13-U15 ROM dumps or documentation for the optional 24K
   BASIC 4.0 package and determine exactly how that feature interacts with U1.
5. Compare other BusCard firmware revisions, if they can be found, against
   revision 0.9 to identify bug fixes or feature changes.
6. Test the reconstructed behavior in MAME or on physical hardware and use
   execution traces to validate the remaining uncertain routine descriptions.

## References used in the reverse engineering

- BusCard U1 annotated 6502 listing accompanying this document.
- BusCard schematic and related design material in the BusCard repository.
- MAME BusCard implementation:
  `https://github.com/mamedev/mame/blob/master/src/devices/bus/c64/buscard.cpp`
- C64 KERNAL disassembly at Zimmers.net:
  `https://www.zimmers.net/anonftp/pub/cbm/src/c64/c64_kernal_disassembly.txt`
- PET BASIC 4.0 disassembly at Zimmers.net:
  `https://zimmers.net/anonftp/pub/cbm/src/pet/pet_rom4_disassembly.txt`
- Commodore archive at Zimmers.net:
  `https://zimmers.net/anonftp/pub/cbm/`

## Summary

The BusCard U1 EPROM is considerably more sophisticated than a simple
peripheral driver ROM. Its 8K image combines a machine-language monitor, an
IEEE-488/parallel-printer I/O subsystem, a carefully constructed partial C64
KERNAL overlay, and an encoded relocatable BASIC command extension.

The firmware's architecture is especially notable for the way it integrates
with the C64 rather than attempting to replace it. It intercepts existing
KERNAL device operations, preserves the stock KERNAL bytes that must remain
visible under the ROM overlay, hooks standard BASIC vectors, and relocates its
BASIC extension below the current top of BASIC memory. The result is a card
whose additional hardware and software can be used through familiar C64
interfaces while still providing a substantial machine-language monitor and
BASIC 4.0-style disk-command environment.
