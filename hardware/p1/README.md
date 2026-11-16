# Modern Floppy Disk Controller – Routed Interface PCB P1

KiCad 9 routed prototype for the Revision B Raspberry Pi OS appliance. The board is 147 × 138 mm with four M3 mounting holes. It connects by ribbon to the Raspberry Pi 4/5 and by a prototype harness to an STM32H563 Nucleo. The STM32 retains the hardware flux capture, write playback and write deadline. The PCB is a signal interface board with through-hole pads throughout.

## Files

- [Bill of materials](BOM.md) lists quantities and U.S. supplier references for the board and harness.

- `floppy-interface-p1.kicad_pcb` is the editable routed PCB. Open it from `floppy-interface-p1.kicad_pro`.
- `pad-net-map.csv` records footprint pins and intentional no-connect pads.
- `routed-top.png` previews the copper and top silkscreen.
- `floppy-interface-p1.kicad_sch`, `fdc-p1.kicad_sym` and `sym-lib-table` provide the editable schematic and its project-local parts. `schematic.pdf` is the print view.
- `nucleo-j4-harness.csv` specifies the 30 harness positions. `bench-validation.md` covers power selection, bridge settings, timers, DMA and hardware acceptance.
- `erc.txt` and `drc.txt` record ERC, DRC, connectivity and schematic-to-PCB parity.

The board uses two copper layers. Nominal widths are 0.25 mm for signals, 0.60 mm for the +5 V and 3.3 V supplies, and 0.45 mm for ground. Local neck-downs reach 0.15 mm. Nominal routing clearance is 0.22 mm, with 0.60 mm vias and 0.30 mm drills. The bottom layer also has a ground fill.

## Connectors and assembly

| Ref | Connection |
|---|---|
| J1 | Keyed 40-pin cable to the Pi GPIO header. Orient conductor 1 at both ends. |
| J2 | Keyed 34-pin 3.5-inch or 5.25-inch connector. Use a straight ribbon for first bring-up and select the appropriate PC or Shugart profile. |
| J3 | Keyed 50-pin connector for a Shugart-style 8-inch drive. Fit a keyed IDC-to-drive-edge harness when needed. This mapping brings out a single drive-select line on pin 26, head load on pin 18 and TG43/reduced write current on pin 2. Qualify the intended drive's pin functions and jumpers. |
| J4 | Keyed 30-pin STM32 module breakout. This is a custom signal harness. Use the included J4 harness CSV. |
| J5 | Drive-side regulated +5 V sense/pull-up input, pin 1 positive, pin 2 ground. Drive power is supplied separately. |
| JP1 | Physical write-enable shunt. Default assembly is shunt removed. |
| TP1–TP6 | Index, read data, write data, write gate, write window, and ground. |

J1–J4 use square pin-1 pads, silkscreen numerals and connector orientation marks. J5 and JP1 have a pin-1 numeral. U1–U5 show pin 1 and the package notch. Verify the shroud key and red stripe on each actual cable.

Connect **one drive at a time**, with both systems powered off while changing cables. The two drive headers share drive signals. The 8-inch spindle/head-load arrangement is intended for the SA800-style baseline and must be checked for the actual mechanism. The 8-inch interface does not provide general-purpose motor-on or special control signals on optional pins. J5 takes the drive's regulated logic rail. It does not feed drive motor or logic power. Keep Pi and drive positive rails separate, with a common signal ground.

## Parts and circuit

| References | Preliminary parts |
|---|---|
| U1–U2 | Socketed SN74LS06N PDIP open-collector drivers. Check sink current and VOL against the selected drive termination. |
| U3–U4 | SN74LVC14A on pin-for-pin SOIC-14 to DIP-14 carriers. The carrier leads are through-hole at the baseboard. Verify carrier envelope and orientation. These two modules and the STM32 development board contain the surface-mount devices in the design. |
| U5 | Socketed CD74HCT21E PDIP gate for firmware request, timer window, physical arm and write-protect clear. Verify thresholds, propagation and power-up behavior. |
| RN1–RN2 | Bussed 1 kΩ ×6 SIP pulldowns, common at pin 1. Check the selected MCU GPIO drive budget against simultaneous assertions. |
| RN3 | Bussed 2.2 kΩ ×6 SIP drive-input pull-ups, common to DRIVE_5V at pin 1. Check original drive output type and termination. Recalculate if the actual pull-up configuration differs. |
| R1–R8, Q1, C1–C5 | Axial resistors, pinout-qualified TO-92 reset MOSFET and radial 100 nF ceramic bypass capacitors. The TO-92 footprint expects source, gate, drain on pads 1, 2, 3. |
| J1–J5, JP1, TP1–TP6 | Through-hole keyed headers, shunt and test pads. |

Receiver outputs are inverted relative to the active-low drive lines. The drive +5 V sense path produces the source for the write-enable jumper through two receive gates. The write gate is active low at the connector. Drive-side write protect is interpreted as asserted low. With no drive connected, the pull-up can indicate a clear write-protect state, so firmware must also validate drive selection and media state before arming.

## Verification and prototype status

KiCad ERC and DRC completed with zero reported violations, zero unresolved connections, and zero schematic-parity differences. All 284 numbered pads were compared with the schematic and original pin map, including intentional no-connects. PCB copper routes and connector placement were preserved.

The Nucleo harness assignments and peripheral functions were checked against ST's documentation. The board and harness still require assembly and physical validation. Follow the bench validation procedure before attaching archival media.

## References

- [Shugart SA800/801 OEM and service manuals](https://bitsavers.trailing-edge.com/pdf/shugart/SA8xx/)
- [SA800-family interface reference](https://allpinouts.org/pinouts/connectors/data_storage/8-floppy-diskdrive/)
- [STM32H563 Nucleo board](https://www.st.com/en/evaluation-tools/nucleo-h563zi.html)
- [Raspberry Pi GPIO header documentation](https://www.raspberrypi.com/documentation/computers/raspberry-pi.html)
- [TI SN74LS06](https://www.ti.com/product/SN74LS06), [TI SN74LVC14A](https://www.ti.com/product/SN74LVC14A), [TI CD74HCT21](https://www.ti.com/lit/ds/symlink/cd74hct21.pdf)
