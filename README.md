# BusCard Project

The complete illustrated project description is available on the [BusCard GitHub Pages website](https://pdbuchan.github.io/buscard/).

This repository documents information on the BusCard, an IEEE-488 and parallel printer interface for the Commodore 64. The BusCard, and subsequent BusCard II, were manufactured by Batteries Included Ltd. of Toronto, Canada. On my BusCard, the circuit board is labeled "Rev A" and "1983". I used it to provide IEEE-488 functionality to my Commodore 64 in order to use a 4040 disk drive and 8023P printer. It also provides BASIC 4.0 and comes with an assembler and disassembler.

Unlike the BusCard II, the BusCard does not contain any proprietary integrated circuits. As a result, I was able to successfully reverse-engineer it and produce an exact working duplicate. I used EAGLE CAD software for the schematic and circuit board design. All information required to produce it is provided. If you would like to build it, you can send the GERBER files provided here to the manufacturer of your choice. You would only need EAGLE if you wish to change the board layout or circuit.

## Repository layout

The repository layout is designed so that the page can be published directly with GitHub Pages.

```text
buscard/
├── README.md
└── docs/
    ├── README.md
    ├── index.html
    ├── .nojekyll
    └── assets/
        ├── buscard_rev_a_manual.pdf
        ├── schematic.pdf
        ├── cam/
        │   ├── README.md
        │   └── elecrow-gerb274x.cam
        ├── datasheets/
        │   ├── README.md
        │   ├── datasheets.html
        │   ├── 1N4148.pdf
        │   ├── 2N3904.pdf
        │   ├── 2N4401.pdf
        │   ├── 74LS00N.pdf
        │   ├── 74LS04N.pdf
        │   ├── 74LS08N.pdf
        │   ├── 74LS139N.pdf
        │   ├── 74LS21N.pdf
        │   ├── 74LS32N.pdf
        │   ├── 75160B.pdf
        │   ├── 75161B.pdf
        │   ├── 8255AC-5.pdf
        │   ├── 82C55A.pdf
        │   ├── 8-SPST-DIP-Switch.pdf
        │   ├── CK05BX104K.pdf
        │   ├── clip_cable.pdf
        │   ├── clip_receptacle.pdf
        │   ├── female-edge-connector.pdf
        │   ├── micro_clip.pdf
        │   ├── resistor-network.pdf
        │   ├── resistors.pdf
        │   ├── shunt.pdf
        │   └── TMS2564.pdf
        ├── drawings/
        │   ├── README.md
        │   ├── buscard_dimensions.pdf
        │   ├── buscard_dimensions.vsdx
        │   ├── buscard_expansion_port.pdf
        │   ├── buscard_expansion_port.vsdx
        │   ├── buscard_IEEE-488_port.pdf
        │   ├── buscard_IEEE-488_port.vsdx
        │   ├── buscard_parallel_port.pdf
        │   ├── buscard_parallel_port.vsdx
        │   ├── buscard_parts_layout.pdf
        │   ├── buscard_parts_layout.vsdx
        │   ├── buscard_schematic.pdf
        │   ├── new_buscard_dimensions.pdf
        │   └── new_buscard_dimensions.vsdx
        ├── eagle/
        │   ├── README.md
        │   ├── buscard.brd
        │   └── buscard.sch
        ├── eprom/
        │   ├── README.md
        │   ├── buscardv0.9-tms2564.html
        │   ├── buscard_u1_rom_discussion.html
        │   ├── additional-sockets-u13-u14-u15.html
        │   ├── burning_TMS2564_EPROM.docx
        │   ├── burning_TMS2564_EPROM.pdf
        │   ├── buscard_u1_6502_annotated_listing.txt
        │   ├── buscard_u1_rom_discussion.md
        │   ├── additional-sockets-u13-u14-u15.md
        │   └── buscardv0.9-tms2564.bin
        ├── gerber/
        │   ├── README.md
        │   ├── gerber.html
        │   ├── buscard.GBL
        │   ├── buscard.GBO
        │   ├── buscard.GBS
        │   ├── buscard.GML
        │   ├── buscard.GTL
        │   ├── buscard.GTO
        │   ├── buscard.GTS
        │   ├── buscard.TXT
        │   ├── PLEASE_CHAMFER_CARD_EDGES.jpg
        │   └── PLEASE_CONTACT_ME_ABOUT_GOLD_FINGERS.txt
        ├── images/
        │   ├── README.md
        │   ├── orig_gallery.html
        │   ├── new_gallery.html
        │   ├── bottom_cover_inside.jpg
        │   ├── bottom_cover_inside_thumb.jpg
        │   ├── bottom_cover_underside.jpg
        │   ├── bottom_cover_underside_thumb.jpg
        │   ├── bottom.jpg
        │   ├── bottom_of_board_backlit.jpg
        │   ├── bottom_of_board_backlit_thumb.jpg
        │   ├── bottom_thumb.jpg
        │   ├── clips.jpg
        │   ├── clips_thumb.jpg
        │   ├── diodes.jpg
        │   ├── diodes-switch_foil_backlit_top.jpg
        │   ├── diodes-switch_foil_backlit_top_thumb.jpg
        │   ├── diodes-switch_foil_top.jpg
        │   ├── diodes-switch_foil_top_thumb.jpg
        │   ├── diodes_thumb.jpg
        │   ├── expansion_port_backlit1.jpg
        │   ├── expansion_port_backlit1_thumb.jpg
        │   ├── expansion_port_backlit2.jpg
        │   ├── expansion_port_backlit2_thumb.jpg
        │   ├── new_bottom.jpg
        │   ├── new_bottom_thumb.jpg
        │   ├── new_top1.jpg
        │   ├── new_top1_thumb.jpg
        │   ├── new_top2.jpg
        │   ├── new_top2_thumb.jpg
        │   ├── new_top_no_chips1.jpg
        │   ├── new_top_no_chips1_thumb.jpg
        │   ├── new_top_no_chips2.jpg
        │   ├── new_top_no_chips2_thumb.jpg
        │   ├── not_plugged_in.jpg
        │   ├── not_plugged_in_thumb.jpg
        │   ├── photo_buscard_board_bottom_r0.jpg
        │   ├── photo_buscard_board_top_r0.jpg
        │   ├── plugged_in.jpg
        │   ├── plugged_in_thumb.jpg
        │   ├── side_view01.jpg
        │   ├── side_view01_thumb.jpg
        │   ├── side_view02.jpg
        │   ├── side_view02_thumb.jpg
        │   ├── side_view03.jpg
        │   ├── side_view03_thumb.jpg
        │   ├── side_view04.jpg
        │   ├── side_view04_thumb.jpg
        │   ├── side_view05.jpg
        │   ├── side_view05_thumb.jpg
        │   ├── side_view06.jpg
        │   ├── side_view06_thumb.jpg
        │   ├── side_view07.jpg
        │   ├── side_view07_thumb.jpg
        │   ├── side_view08.jpg
        │   ├── side_view08_thumb.jpg
        │   ├── side_view09.jpg
        │   ├── side_view09_thumb.jpg
        │   ├── side_view10.jpg
        │   ├── side_view10_thumb.jpg
        │   ├── side_view11.jpg
        │   ├── side_view11_thumb.jpg
        │   ├── side_view12.jpg
        │   ├── side_view12_thumb.jpg
        │   ├── side_view13.jpg
        │   ├── side_view13_thumb.jpg
        │   ├── top_cover_inside.jpg
        │   ├── top_cover_inside_thumb.jpg
        │   ├── top_foil1.jpg
        │   ├── top_foil1_thumb.jpg
        │   ├── top_no_covers1.jpg
        │   ├── top_no_covers1_thumb.jpg
        │   ├── top_no_covers2.jpg
        │   ├── top_no_covers2_thumb.jpg
        │   ├── top_of_board_backlit.jpg
        │   ├── top_of_board_backlit_thumb.jpg
        │   ├── top_with_bottom_cover1.jpg
        │   ├── top_with_bottom_cover1_thumb.jpg
        │   ├── top_with_bottom_cover2.jpg
        │   ├── top_with_bottom_cover2_thumb.jpg
        │   ├── top_with_cover1.jpg
        │   ├── top_with_cover1_thumb.jpg
        │   ├── top_with_cover2.jpg
        │   └── top_with_cover2_thumb.jpg
        ├── libraries/
        │   ├── README.md
        │   ├── libraries.html
        │   ├── 1N4148.lbr
        │   ├── 2564.lbr
        │   ├── 2N3904_and_2N4401.lbr
        │   ├── 74LS00N.lbr
        │   ├── 74LS04N.lbr
        │   ├── 74LS08N.lbr
        │   ├── 74LS139N.lbr
        │   ├── 74LS21N.lbr
        │   ├── 74LS32N.lbr
        │   ├── 75160AN.lbr
        │   ├── 75161AN.lbr
        │   ├── 8255AC-5.lbr
        │   ├── capacitor-CK05BX104K.lbr
        │   ├── clip-header-MTA02-100.lbr
        │   ├── commodore-card-edges.lbr
        │   ├── dip-switch-DS08.lbr
        │   ├── female-44-pin-edge-connector.lbr
        │   ├── frame.lbr
        │   ├── jumper-header-3.lbr
        │   ├── mystery_plug-MTA04-100.lbr
        │   ├── resistor.lbr
        │   └── resistor-network-sil-G09R.lbr
        └── parts/
            ├── README.md
            ├── new_buscard_bom.xlsx
            └── orig_buscard_parts.xlsx
```
