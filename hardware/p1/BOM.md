# Revision P1 bill of materials

Quantities are for one Modern Floppy Disk Controller P1 interface PCB. US catalog references checked November 15, 2026. Listing availability, minimum order and lead time can change before checkout. This BOM follows the P1 schematic, routed board and [pad-to-net map](pad-net-map.csv).

## PCB parts

| References | Qty | Manufacturer and MPN | Description | US catalog |
|---|---:|---|---|---|
| C1–C5 | 5 | Vishay K104K15X7RF5TL2 | 100 nF, 50 V, X7R radial ceramic, 2.50 mm pitch | [DigiKey BC1084CT-ND](https://www.digikey.com/en/products/detail/vishay-beyschlag-draloric-bc-components/K104K15X7RF5TL2/286538) |
| J1 | 1 | 3M N2540-6002-RB | 2×20 vertical, keyed shrouded IDC header, 2.54 mm | [DigiKey MHC40K-ND](https://www.digikey.com/en/products/detail/3m/N2540-6002-RB/755184) |
| J2 | 1 | 3M N2534-6002-RB | 2×17 vertical, keyed shrouded IDC header, 2.54 mm, 3.5-/5.25-inch port | [DigiKey MHC34K-ND](https://www.digikey.com/en/products/detail/3m/N2534-6002-RB/755183) |
| J3 | 1 | 3M N2550-6002-RB | 2×25 vertical, keyed shrouded IDC header, 2.54 mm, 8-inch port | [DigiKey MHC50K-ND](https://www.digikey.com/en/products/detail/3m/N2550-6002-RB/755185) |
| J4 | 1 | 3M D3440-6202-AR | 2×15 vertical keyed shrouded through-hole header, 2.54 mm, short ejector latches. Dry-fit latch envelope and select a mating 30-position IDC socket. | [DigiKey 3M157654-ND](https://www.digikey.com/en/products/detail/3m/D3440-6202-AR/1885906) |
| J5 | 1 | JST B2B-XH-A | Vertical two-position XH header, 2.50 mm pitch | [DigiKey 455-B2B-XH-A-ND](https://www.digikey.com/en/products/detail/jst-sales-america-inc/B2B-XH-A/1651045) |
| JP1 | 1 | Sullins PEC02SAAN | 1×2 through-hole male header, 2.54 mm pitch | [DigiKey 35-PEC02SAAN-ND](https://www.digikey.com/en/products/detail/sullins-connector-solutions/PEC02SAAN/859155) |
| JP1 shunt | 1 | Sullins QPC02SXGN-RC | Removable shorting shunt, fit only when intentionally enabling writes | [DigiKey S9337-ND](https://www.digikey.com/en/products/detail/sullins-connector-solutions/QPC02SXGN-RC/2618262) |
| Q1 | 1 | onsemi 2N7000BU | TO-92 reset FET, source/gate/drain on pins 1/2/3. Form the 1.27 mm leads to match the 2.54 mm PCB pitch and confirm orientation. | [DigiKey 2N7000BU-ND](https://www.digikey.com/en/products/detail/onsemi/2N7000BU/976604) |
| R1 | 1 | YAGEO MFR-25FBF52-1K | 1 kΩ, 1%, ¼ W axial metal film | [DigiKey 1.00KXBK-ND](https://www.digikey.com/en/products/detail/yageo/MFR-25FBF52-1K/13011) |
| R2–R5, R7–R8 | 6 | YAGEO MFR-25FBF52-10K | 10 kΩ, 1%, ¼ W axial metal film | [DigiKey 10.0KXBK-ND](https://www.digikey.com/en/products/detail/yageo/MFR-25FBF52-10K/13219) |
| R6 | 1 | YAGEO MFR-25FBF52-100K | 100 kΩ, 1%, ¼ W axial metal film | [DigiKey 100KXBK-ND](https://www.digikey.com/en/products/detail/yageo/MFR-25FBF52-100K/13473) |
| RN1–RN2 | 2 | Bourns 4607X-101-102LF | 1 kΩ × 6, bussed SIP-7, common on pin 1 | [DigiKey 4607X-101-102LF-ND](https://www.digikey.com/en/products/detail/bourns-inc/4607X-101-102LF/2565620) |
| RN3 | 1 | Bourns 4607X-101-222LF | 2.2 kΩ × 6, bussed SIP-7, common on pin 1 | [DigiKey 4607X-101-222LF-ND](https://www.digikey.com/en/products/detail/bourns-inc/4607X-101-222LF/2565633) |
| U1–U2 | 2 | TI SN74LS06N | PDIP-14 hex inverting open-collector driver | [Mouser 595-SN74LS06N](https://www.mouser.com/ProductDetail/Texas-Instruments/SN74LS06N) · [DigiKey 296-15069-5-ND, Rochester Marketplace](https://www.digikey.com/en/products/detail/texas-instruments/SN74LS06N/563053) |
| U3–U4 | 2 | TI SN74LVC14ADR | SOIC-14 narrow-body Schmitt inverter, used with carriers below | [DigiKey 296-1699-1-ND, cut tape](https://www.digikey.com/en/products/detail/texas-instruments/SN74LVC14ADR/377434) |
| U3–U4 carriers | 2 | Chip Quik PA0003C | Compact SOIC-14 to DIP-14 adapter, 7.62 mm DIP row spacing. Assemble SOIC pin 1 to carrier pin 1. | [DigiKey 315-PA0003C-ND](https://www.digikey.com/en/products/detail/chip-quik-inc/PA0003C/12352170) |
| U5 | 1 | TI CD74HCT21E | PDIP-14 dual four-input AND, TTL-compatible inputs | [DigiKey 296-9262-5-ND](https://www.digikey.com/en/products/detail/texas-instruments/CD74HCT21E/386074) · [Mouser 595-CD74HCT21E](https://www.mouser.com/ProductDetail/Texas-Instruments/CD74HCT21E) |
| U1, U2, U5 sockets | 3 | TE Connectivity 1-2199298-3 | Optional 14-pin DIP sockets for the PDIP devices, 7.62 mm row spacing | [DigiKey A120348-ND](https://www.digikey.com/en/products/detail/te-connectivity-amp-connectors/1-2199298-3/5022040) |
| TP1–TP6 | 0 fitted | PCB test pads | Optional wire loops can be fitted for the bench fixture | — |
| H1–H4 | 4 sets if mounted | M3 mounting hardware | Select standoff length and fasteners for the enclosure | — |

## Off-board parts and cables

| Item | Qty | Procurement |
|---|---:|---|
| Controller module | 1 | ST NUCLEO-H563ZI, [DigiKey 497-NUCLEO-H563ZI-ND](https://www.digikey.com/en/products/detail/stmicroelectronics/NUCLEO-H563ZI/18110502) or [Mouser 511-NUCLEO-H563ZI](https://www.mouser.com/ProductDetail/STMicroelectronics/NUCLEO-H563ZI). The checked DigiKey listing showed zero stock and Mouser listed backorder. |
| Pi host | 1 | Raspberry Pi 4 Model B or Pi 5. Example [Pi 5 at DigiKey](https://www.digikey.com/en/products/detail/raspberry-pi/SC1431/21658261). Include Pi-rated power supply and boot storage for Raspberry Pi OS. |
| Nucleo Morpho headers | 2 if unpopulated | 2×33, 2.54 mm through-hole male. A [Sullins PBC36DAAN](https://www.digikey.com/en/products/detail/sullins-connector-solutions/PBC36DAAN/861057) breakaway can be cut to 33 rows. |
| Pi cable | 1 | 40-conductor straight-through ribbon with female 2×20 ends. Verify pin 1 at both ends, particularly at the Pi's exposed header. |
| J4 harness | 1 | Keyed female 2×15 IDC end and individual 2.54 mm female contacts or breakouts at CN11/CN12. Build to the [harness pin map](nucleo-j4-harness.csv). |
| J5 mate | 1 housing, 2 contacts | [JST XHP-2](https://www.digikey.com/en/products/detail/jst-sales-america-inc/XHP-2/555485) housing and [SXH-001T-P0.6 cut-tape contacts](https://www.digikey.com/en/products/detail/jst-sales-america-inc/SXH-001T-P0.6/527370). Select wire to suit the contacts. |
| Drive signal cable and drive supply | 1 set per drive type used | Straight 34-conductor cable for J2, with the appropriate IDC or edge connector at the drive. Use a 50-conductor cable and the correct 8-inch drive termination or edge connector at J3. Verify ground/key positions, pin-1 orientation and drive supply rails against the drive manual. |
| Bare PCB | 1 | Order from reviewed P1 fabrication outputs. |

## Before ordering or assembly

- **U1/U2 are the small-order sourcing risk.** Compare the Mouser listing and DigiKey Marketplace cart at quantity two. The Marketplace snapshot showed stock and a price break at 1,000. It did not establish a usable two-piece price. Preserve LS TTL input loading and the open-collector sink rating when qualifying a substitute.
- **U3/U4 carriers are the surface-mount exception.** The carrier-to-baseboard connection is through-hole. Fit U3/U4 carriers directly or use sockets specifically rated for the carrier's pin dimensions. Count two additional DIP sockets only if socketing both carriers.
- **Check mechanical fit before buying multiples.** Compare J4's latch clearance and connector entry with the mating socket. Check carrier height and socket fit. J2 pad 3 is intentionally NC. If a mating plug blocks position 3, use the matching key arrangement before insertion. Never force a keyed ribbon. Connect one drive at a time.
- **Prototype acceptance follows assembly.** Use the [bench validation procedure](bench-validation.md) for power, write interlock, and drive testing.
