# Modern Floppy Disk Controller

**Revision B engineering design · 15 November 2026**

Design for a standalone Raspberry Pi OS appliance that captures and replays digital flux-transition timing on conventional 8-inch, 5.25-inch, and 3.5-inch disk drives. The drive supplies the read pulses from its analog read channel. The controller timestamps those pulses and produces timing-accurate write pulses. The Raspberry Pi handles disk formats and stores images locally.

This is an engineering specification. PCB routing, firmware implementation, and hardware validation remain to be completed. Model-specific drive profiles are qualified against the actual drive and manual.

## 1. Architecture

| Layer | Revision B design |
|---|---|
| Appliance | Raspberry Pi 4 Model B or Raspberry Pi 5 running 64-bit Raspberry Pi OS Lite, with local terminal access and optional SSH |
| Pi software | C++17 CLI and shared codec/library code, CMake, Linux spidev |
| Board link | SPI0 over the 40-pin GPIO header, with an active-low reply-ready line and a controller-reset line |
| MCU | STM32H563 with local track memory, hardware timers and DMA. Prototype with NUCLEO-H563ZI plus Pi-header and buffered drive wiring |
| Flux timing | 25 MHz reference timer, 40 ns per tick. Hardware capture of read data and index. Independent hardware compare output and DMA for playback |
| Drive connection | One active drive, keyed 34-pin PC/Shugart connection, and a separate keyed 8-inch 50-pin adapter |
| Power | Independently sized Pi and drive supplies. Pi header supplies controller logic only. Use a common signal ground and protected drive-side buffers |

Select, capture, decode, write, and verify from the Pi. Images are ordinary files on local storage. Linux handles commands and block transfers while the STM32 times read and write edges independently.

## 2. Hardware

### 2.1 Main board

- Pi header prototype map, using BCM GPIO numbering. SPI0 MOSI GPIO10/pin 19, MISO GPIO9/pin 21, SCLK GPIO11/pin 23, CE0 GPIO8/pin 24, REPLY_READY GPIO25/pin 22, CONTROLLER_RESET GPIO24/pin 18, 5 V pin 2 and GND pin 6. Use 3.3 V logic. Verify SPI alternate functions on the selected STM32 package. Hold chip select high and make the controller MISO high impedance while deselected. Drive controller reset with an open-drain stage and default it to released.
- Regulate the Pi header 5 V rail locally to 3.3 V. Budget board current with both Pi models. Keep the drive supply isolated from the Pi supply's positive rails. Include SWD, boot recovery, reset, and test points for SPI, flux timing, index, write data, write gate and hardware permit.
- Use 5 V-tolerant Schmitt receiver inputs with specified partial-power-down behavior, such as SN74LVC14A at 3.3 V. Account for inversion in timer edge selection. Use an optional drive-side pull-up network for open-collector outputs. Size the pull-ups to the drive's sink and cable-edge requirements and power them from the drive-side 5 V domain.
- Use active-low open-drain outputs. Prototype with logic-level NMOS stages qualified for gate drive at 2.5–3.3 V, with gate series resistors and pulldowns. Qualify sink current, low-level voltage, rise time, and released-state leakage on a populated ribbon. A 150 Ω pull-up at 5 V draws about 33 mA per asserted line. Never assume that lightweight CMOS GPIO can sink a vintage termination network. Install the drive terminator per its manual and avoid double termination.
- Give WRITE GATE its own hardware logic AND before the sink driver. Inputs include physical write-enable switch, valid drive-side power, adapter/direct-port permit, a live hardware write-window signal, firmware request, and write-protect clear where the selected drive exposes a valid signal. Pull the output sink off through power-on/reset. Cancel, controller reset, invalid drive state, and watchdog expiry clear the permit. Set the hardware gate deadline before playback starts, independently of DMA or the host.
- A mechanical mode switch or equivalent hardware interlock selects the direct port or the adapter. An adapter ID plus presence and drive-power sensing allows software to reject a profile mismatch. The user connects the drive and sets jumpers with power off. Key connectors, label pin 1, and run signal grounds along the ribbon.
- Externally power the drive from a supply sized for that drive's rails and startup current. Supply drive-side 5 V sensing/reference at a protected dedicated input. Verify drive requirements for 5 V, 12 V, 24 V, and any separate motor supply. Keep cabinet mains and motor wiring within the original or separately engineered drive power assembly.

### 2.2 34-pin connections

All signal names prefixed with / are active low in the standard profiles. Direct-port output mapping is selectable in software with isolated drive outputs for the configurable lines. Use a straight ribbon as the reference setup and qualify a separate PC twisted-cable profile.

| Pin | PC profile | Shugart-style profile |
|---|---|---|
| 2 | Density select, where fitted | Model-specific or unused |
| 8 | /INDEX input | /INDEX input |
| 10, 12, 14, 16 | /MOTOR A, /SELECT B, /SELECT A, /MOTOR B, respectively at controller connector | Commonly /SELECT 0, /SELECT 1, /SELECT 2, /MOTOR, respectively. Confirm drive jumpers |
| 18, 20 | /DIRECTION, /STEP | Same |
| 22, 24 | /WRITE DATA, /WRITE GATE | Same |
| 26, 28, 30 | /TRACK ZERO, /WRITE PROTECT, /READ DATA | Same |
| 32 | /SIDE SELECT | /SIDE SELECT on double-sided drives |
| 34 | Often /DISK CHANGE | Often /READY |
| Odd pins | Signal returns per drive manual and connector key | Same |

Pin 2 drive mode, pin 34 semantics, drive-select jumpers, side select, RPM and step timing belong in each profile. On a typical PC ribbon the twist swaps conductors 10–16. Firmware uses the selected connector/cable position and never asserts both select outputs.

### 2.3 Dedicated 8-inch adapter

Provide a 50-pin keyed connector or keyed IDC-to-edge harness as required. Carry buffered select, direction, step, write data, write gate, and side select, plus separately controllable head load and reduced-write-current/TG43 and profile-specific motor control where supported. Return raw read data, index, track zero, write protect, and ready. Expose additional pins only after checking the specific drive manual.

| SA800-family reference signal | 50-pin assignment to verify against drive |
|---|---|
| Reduced write current | 2, where enabled by the drive's configuration |
| Side select / head load | 14 / 18, where supported |
| Index / ready | 20 / 22 |
| Drive select | 26, 28, 30, or 32, per jumpers |
| Direction / step | 34 / 36 |
| Write data / write gate | 38 / 40 |
| Track zero / write protect / raw read data | 42 / 44 / 46 |

Keep the adapter harness profile-specific. Use the drive's raw read-data path when the manual offers alternate separated outputs or requires a jumper. For an SA800-class drive with a continuously running spindle, apply head load and wait for ready and index under the configured timing. Treat TG43 as a profile rule with a configurable threshold and polarity. Evaluate it from the known physical cylinder after seek, establish it before write gate, and reject writes on drives requiring that signal unless the adapter exposes it. Blank, missing-index, and hard-sectored media require explicit handling.

## 3. Firmware and timing

### 3.1 Capture

1. Select drive, spin up or load head, seek, and confirm the profile's ready criteria. Home by bounded outward stepping against TRACK ZERO. Invalidate the cylinder after reset, failed seek, or unexpected drive change.
2. Capture the active edge of READ DATA and the active edge of INDEX on separate channels of one timer. Use separate DMA rings, record a shared capture epoch and duration, and preserve all index events. Sample independently of buffer boundaries. Reject timer overcapture, DMA overrun, or event-storage exhaustion with a terminal status.
3. Convert consecutive read timestamps into unsigned 16-bit tick deltas. Reserve 0 as an escape followed by a little-endian 32-bit interval. Record first-edge offset and exact capture duration, even for an empty track. Store index offsets separately with 32-bit tick units. At 25 MHz a 32-bit timer wraps in about 172 s. Bound one operation well below that and handle rollover by unsigned subtraction.
4. Support indexed capture, timed capture, and profile-aware hard-sector capture. For hard sectors, retain every physical pulse and identify revolution boundaries from a verified pattern when the drive presents the sector holes. With no compatible index, capture by time and preserve the unaligned origin. Capture one revolution at a time if the requested count exceeds local capacity. Keep continuous and separately reacquired revolutions distinct in the archive.

### 3.2 Playback

1. Upload one complete write plan with intended cylinder/head, start mode, first transition offset, pulse width, intervals, and gate start/stop offsets. Check length, CRC, tick arithmetic, timing limits, and media profile. Apply precompensation at the host before upload when synthesizing a track, with parameters stored in the profile. Preserve source capture and record any changes made for the write.
2. Require physical write enable and explicit host commit, then recheck drive selection, current cylinder/head, power, readiness, index expectations, write protect, and media change. Start from a hardware-synchronized index trigger for indexed writes. Bound the wait for index. Use explicitly defined timed playback only with an appropriate profile and a known write window.
3. Prime the compare timer with the first scheduled edge. A timer compare toggles the write-data output and requests DMA to load the next absolute compare time. Expand compact track intervals into alternating assertion and release times in two small DMA buffers, then refill halves ahead of use. Specify the actual write-data pulse width from the drive manual and program a separately bounded hardware gate window. Use a guard/sentinel and verify service margin with DMA contending for the bus. If the next data block cannot be supplied before its deadline or a timer/DMA fault occurs, force gate inactive and return an incomplete-write status. Playback requires no track data transfers after commit.
4. End at the specified splice within the measured target revolution. Match target RPM where required with explicit host rescaling or regeneration. Abort before writing if the requested window cannot fit. Verify with an independent readback and report sector CRC mismatch or flux-quality differences according to format. A media write can be partial after interruption, so identify the affected track and block automatic retries.

### 3.3 Initial engineering budgets

| Parameter | Design target |
|---|---|
| Timebase | 25 MHz from a crystal-referenced clock tree. Verify actual clock, tick conversion, and temperature behavior |
| Peak event workload | Qualify 1 million flux pulses/s and at least 2 million write compare updates/s for synthetic stress. Pulse width and minimum spacing remain profile-specific |
| DMA load | 4 MB/s for raw read timestamps at the stress rate or 8 MB/s for write compare timestamps. Budget two 1,024-entry 32-bit write DMA halves, each providing approximately 512 µs refill time at 2 million compare events/s |
| Track buffer | 480 KiB design allocation in DMA-eligible SRAM with bank, alignment, and cache/coherency mapping checked before layout signoff. Separate ring buffers, index, SPI, stack and diagnostics allocations |
| Track sizing | At 300 RPM and 1 million pulses/s a revolution is 200 ms and approximately 400 kB at two bytes per pulse. At 360 RPM it is about 333 kB. Sparse long intervals use the escape encoding. Multiple revolutions fit according to actual encoded size |
| Read transfer | Transfer captured blocks after completion. Benchmark sustained SPI throughput on both Pi models. At 4 MHz, the two-transaction framing limits useful throughput to less than 0.25 MB/s. At a measured 0.2 MB/s, a 400 kB revolution takes about 2 s, in addition to seek, acquisition, and rotation wait |

Select TIM2/TIM5 roles, channels, trigger routing and GPDMA requests using the package alternate-function tables, reference manual, and Nucleo board schematic before PCB layout. Prove simultaneous read/index capture and buffered write compare timing on the bench before freezing the design.

## 4. Internal SPI protocol

The Pi is SPI master and the STM32 is slave. Start at 4 MHz, mode 0, MSB first. Qualify higher rates on both Pi models after measuring signal integrity and DMA behavior. Reserve GPIO25 for a reply-ready level asserted only after the MCU has staged a complete response and armed SPI transfer DMA.

**Wire frame.** Little-endian fields in this order. ASCII magic `FDC1` (4 bytes), major and minor version (1 byte each), opcode (uint16), request ID (uint32), payload length (uint32), and payload CRC-32/ISO-HDLC (uint32). Payload follows. A request occupies one 1,024-byte CE0 transaction, padded with zeros. After REPLY_READY asserts, clock a separate 1,024-byte reply transaction with zero-filled MOSI. Limit payloads to 1,004 bytes, including metadata, and validate lengths before use. Replies use opcode bit 15, echo the request ID, and begin with uint16 status and uint16 reserved. Post long operations immediately and poll progress through subsequent commands so cancellation stays available. Keep one command/reply exchange outstanding. After the reply, the MCU deasserts REPLY_READY once the next request receiver is armed. The Pi waits for that level before sending another command. Use offset-addressed data blocks and a CRC over the full staged object.

Unexpected transaction sizes or invalid frames discard the staged command and invalidate write authorization. On timeout, re-establish a known protocol state by controller reset and HELLO. Commit is never automatically retried. Reset drops the gate and invalidates position and write authorization. During an already committed write, the hardware deadline remains effective if the Pi stops communicating.

| Opcode | Operation |
|---|---|
| 0x0001 / 0x0002 | HELLO / GET_STATUS. Return protocol version, board/firmware version, tick rate, limits, adapter identity, drive inputs, and active operation state |
| 0x0010–0x0013 | SET_PROFILE, SELECT_DRIVE, HOME, SEEK. Validate seek bounds, track step multiplier, and profile identity |
| 0x0020–0x0022 | CAPTURE, READ_CAPTURE_BLOCK, RELEASE_CAPTURE. CAPTURE returns an operation handle and completed data stays available until released or session reset |
| 0x0030–0x0033 | BEGIN_WRITE, WRITE_BLOCK, ARM_WRITE, COMMIT_WRITE. Stage by offset, verify full CRC and profile, then issue a short-lived one-use token scoped to the plan and media state |
| 0x0040 | CANCEL. Stop active hardware operation, invalidate write token, and report whether a write window was entered |

Operation errors include invalid profile, no adapter, no index, no ready, write protected, write disabled, invalid timing, geometry limit, overflow, CRC failure, DMA fault, and interrupted write. Poll operation progress with GET_STATUS. Do not automatically replay COMMIT_WRITE after a timeout. A reconnect requires the host to inspect state or read the affected track before a new write attempt. Support recovery firmware update independently of the main protocol, with a physical boot/recovery path.

## 5. Raspberry Pi OS software

- **Operating system.** Qualify a named 64-bit Raspberry Pi OS Lite image on both Pi 4 and Pi 5 and record its release and kernel build. Enable SPI through `raspi-config`. Default to SPI0 CE0 (`/dev/spidev0.0`), with an override for device path and SPI rate. Use libgpiod for reply-ready and controller reset, discovering the header GPIO controller by label or line identifiers. Avoid fixed gpiochip numbers across Pi models.
- **Deployment.** Build a native arm64 package with CMake. Install an unprivileged CLI and grant the operator access to the SPI device and necessary GPIO lines. Serialize hardware access with an exclusive local lock held throughout each operation. Run locally from a terminal or remotely through SSH. Handle terminal hangup by attempting cancellation when the process can still reach the board. Controller-side gate limits apply independently.
- **Commands.** Provide `list`, `drive-test`, `capture`, `decode`, `write`, and `verify`. Add head, cylinder, revolution and profile selection to the media operations. Keep decoding available offline. Require an explicit write command and the physical write-enable switch. Check free space before full-disk capture, retain completed tracks across interruptions, and finalize the archive after validation.
- **Data and profiles.** Save to a user-selected local directory, with support for mounted removable storage. Use versioned drive profiles for geometry, step timing, RPM range, electrical signal roles and write timing. Keep media format definitions separate. Use stable exit codes and machine-readable diagnostics for scripting.

### 5.1 Images and formats

- Archive the raw transition timing and index events first. Keep actual tick rate, cylinder/head, revolution boundaries or capture offsets, profile, RPM estimate, and quality flags with the record. Document the native format and include CRCs. Add SCP import/export with round-trip checks and flag metadata that cannot be preserved in an export.
- Qualify IBM-style FM and MFM sector decode and encode first, then qualify Amiga MFM and Commodore/Apple GCR separately with known-good disks. Preserve sector CRCs and status for reads. Keep standard sector images as derivatives when the format permits them.
- Use a 34-pin drive appropriate to the media when it can physically read and write the format. Original Sony Macintosh mechanisms, Apple Disk II mechanisms and proprietary drives require dedicated electrical adapters where their control interfaces differ. Macintosh 400K/800K variable-speed writes require a qualified mechanism and playback profile. Treat those, 2.88 MB ED, and hard-sector write modes as separately qualified extensions. Preserve multi-revolution variations and report when a drive cannot reproduce observed weak-bit, splice, or protection behavior.
- Match drive head width to the media when writing. In particular, qualify 48-TPI 5.25-inch media in a native 48-TPI mechanism. A 96-TPI drive can double-step for compatible reads but leaves a narrower written track.

## 6. Prototype and acceptance

| Stage | Required evidence |
|---|---|
| Electrical and timing bench | Measure TTL levels, sink current, idle state, rise/fall times, and decode at the drive connector with realistic termination. Inject synthetic read/index patterns through the receiver, sweep minimum pulse width and spacing, and compare timestamps to a calibrated generator. Check repeatable read/write edge error after accounting for fixed propagation delay. Initial acceptance target is ±2 ticks (80 ns) against the fixture, subject to characterization |
| Fail-safe bench | Observe gate at reset, boot, Pi reboot, SPI reset and a stalled Pi process, failed arming, power interruption, write-protect assertion, and forced DMA starvation. Confirm gate never asserts without a valid plan and that active gate clears on detected fault and by the independent deadline. Measure the actual fault-to-gate-off latency |
| 3.5-inch | PC interface HD mechanism. Capture and decode known-good 720 KB and 1.44 MB media, write expendable disks, then verify with an independent controller or legacy system |
| 5.25-inch | Test a native 360 KB drive and a 1.2 MB drive. Verify density, seek and RPM profiles, read at least one original disk per format, and write and independently verify scratch media |
| 8-inch | Qualify an SA800/801-class single-sided FM drive, then a compatible double-sided 8-inch drive for MFM. Verify head load, raw data, ready, index, physical-cylinder tracking, reduced write current behavior across its threshold, and read/write interoperability on scratch disks |
| Pi OS and archival | On both Pi 4 and Pi 5, test startup, SPI detection, transfer under storage and CPU load, cancellation, interrupted SPI, controller reset, and archive round trips. Test missing index, buffer exhaustion, known bad CRC, malformed requests, and Pi reset during a scratch-disk write |

First prototype milestone is a 3.5-inch write/readback on Raspberry Pi OS via SPI to a Nucleo board and 34-pin buffer board, plus timing and gate measurements from a logic analyzer. Bring up 5.25-inch and 8-inch drive profiles on the same firmware before committing to a production PCB. Maintain qualified profile entries by make, model, revision, jumper settings, cable type and supported formats.

## Sources

- [STMicroelectronics STM32H563/573 product line](https://www.st.com/en/microcontrollers-microprocessors/stm32h563-573.html) and [STM32H563 reference documentation](https://www.st.com/en/microcontrollers-microprocessors/stm32h563-573/documentation.html)
- [STMicroelectronics GPDMA application note AN5593](https://www.st.com/resource/en/application_note/an5593-how-to-use-the-gpdma-for-stm32-mcus-stmicroelectronics.pdf)
- [Texas Instruments SN74LVC14A specifications](https://www.ti.com/product/SN74LVC14A)
- [Shugart SA800/801 OEM and service manuals, Bitsavers](https://bitsavers.trailing-edge.com/pdf/shugart/SA8xx/)
- [SA800/801 50-pin interface table](https://allpinouts.org/pinouts/connectors/data_storage/8-floppy-diskdrive/)
- [Greaseweazle raw flux tools](https://github.com/keirf/greaseweazle) and [FluxEngine](https://github.com/davidgiven/fluxengine)
- [Raspberry Pi OS configuration and SPI enablement](https://www.raspberrypi.com/documentation/computers/configuration.html)
- [Raspberry Pi 4 header and GPIO assignments](https://pip-assets.raspberrypi.com/categories/545-raspberry-pi-4-model-b/documents/RP-008341-DS-1-raspberry-pi-4-datasheet.pdf)
