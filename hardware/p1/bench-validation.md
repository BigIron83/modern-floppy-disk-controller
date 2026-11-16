# P1 harness configuration and prototype validation

**Status** · Procedure prepared. No physical measurements have been made.

## Harness and firmware contract

Install the CN11/CN12 ST Morpho headers as needed and wire the keyed J4 harness using `nucleo-j4-harness.csv`. The CSV specifies pin numbers as marked on the Nucleo. Confirm with a continuity meter before mating either end and keep the SPI wiring short with grounds. On the Nucleo, select external 5 V with JP2 at 5–6. J4 pin 2 feeds CN11 pin 6 and J4 pin 3 receives CN11 pin 16 from the regulator. Follow the manual's external-power-then-ST-LINK-USB sequence if using the debugger. Verify the Pi/header 5 V remains within the Nucleo's 4.75–5.25 V E5V range under load. Use the documented default PC13 user-button routing so PA0 remains available. Release PB3 by checking SB33 and SB39 are both open for the Morpho configuration. Use SWD with SWO and full JTAG disabled. Keep unused ST Zio peripherals disconnected.

| Function | Peripheral | DMA request |
|---|---|---|
| Read flux | TIM2 CCR1, PA15 rising edge | 72, peripheral to memory, 32-bit capture |
| Index | TIM2 CCR2, PB3 rising edge | 73, peripheral to memory, 32-bit capture |
| Write edges | TIM5 CCR1, PA0 toggle, start inactive | 88, memory to peripheral, 32-bit next compare |
| Hardware gate window | TIM5 CH4, PA3, active-high PWM2 in single-pulse mode | None |
| Pi link | SPI1 hardware NSS, 3.3 V, SPI mode 0 | RX 6 and TX 7 |

Configure both timers for 25 MHz and verify clock frequency on the bench. In capture mode use a free-running TIM2 for common-epoch timestamps. Treat overcapture and DMA error as invalid capture. On playback use the verified TIM2 TRGO to TIM5 ITR1 route. Candidate firmware configuration uses TIM2 slave-reset mode on TI2FP2 so an index edge produces an update routed through TRGO. Start TIM5 on that trigger in one-pulse mode. Use PWM2 CH4 to open the gate at its compare and close it at the ARR update. TIM5 CH1 compare edges must fall within the window. Require a validated first compare and final inactive edge. Preload the timer registers with the trigger path disabled. Arm only after clearing pending events and ensure the waiting TIM2 counter cannot wrap during the index timeout. Test that a software update, unexpected second index, reset and a failed DMA refill cannot open or prolong the gate. The assignments are verified against RM0481. Actual trigger, DMA, and gate behavior depend on firmware implementation and are bench acceptance tests.

HCT gate truth condition: WG_REQ AND WINDOW_OK AND ARM_OK AND WP_CLEAR. With the physical shunt off or drive +5 V absent, ARM_OK should be low. The probe signal WGATE_N is active low. Drive inputs present as active-high at J4 because of U3. With no drive connected and the pull-ups powered, WP_CLEAR can be high, so write authorization also requires known drive/media state.

## Acceptance procedure

Record controller board revision, Nucleo revision, drive model and jumper settings, media, firmware commit, instruments and trace filenames. Leave JP1 open for steps 1–4.

| Step | Exercise | Pass criterion |
|---|---|---|
| 1. Unpowered checks | Inspect pin 1, keyed ribbons, socket orientation, RESET MOSFET pin order and pull-up networks. Measure continuity on all 30 J4 wires and ground returns. Check isolation of the drive supply from Pi 5 V and MCU 3.3 V. | Wiring agrees with the pad map and harness CSV. Positive rails are isolated as designed and share signal ground. |
| 2. Power sequencing | Power Pi/controller first, then drive side, and repeat in reverse. Start with a current-limited supply and representative drive-side pull-ups. Scope 5 V, 3.3 V, reset and WGATE_N through startup, brownout and power removal. | Rails stay within component limits. No unintended current path into an unpowered board. WGATE_N does not assert across the tested power sequences. |
| 3. SPI and reset | Validate commands on both Pi models, with chip select and REPLY_READY. Reset during transactions and repeat. | Correct framing and response acknowledgment. MISO releases with NSS inactive. Reset invalidates staged writes. |
| 4. Synthetic read | Inject open-collector-style read and index pulses through J2, then J3. Sweep shortest planned pulse spacing and timing. Compare capture offsets against a calibrated source. | No missed or doubled pulses at the selected profile limit. Index and flux share the same timebase. Initial timing target is ±80 ns after fixed fixture delay. Errors are reported visibly. |
| 5. Synthetic playback | Arm with fixture pull-ups and JP1 installed. Scope INDEX, WINDOW_OK, WGATE_N and WDATA_N. Exercise all interlock signals separately and force DMA starvation while armed. | Starts at configured index offset. Pulses and splice fit within the window. Gate stays inactive with an interlock missing and closes by the timer deadline even with DMA stopped. Measure fault-to-gate-off latency. |
| 6. 3.5-inch | Use 720 KB and 1.44 MB references. Capture and decode originals with JP1 open. Use expendable media for writes and independent-controller readback. | Sector and CRC match for each qualified drive/media profile. |
| 7. 5.25-inch | Repeat with native 360 KB and 1.2 MB drives, matching the media and physical geometry. | Correct RPM and stepping. Independent readback matches the source image on scratch media. |
| 8. 8-inch | Begin with an SA800/801-style drive and verify the actual keyed harness and drive power. Capture FM and independently verify a scratch write. Scope head load and the documented TG43 transition. Repeat with a suitable double-sided MFM drive. | Stable index/ready, correct head load and write current selection, and matching independent readback over the qualified cylinders and sides. |

Stop and resolve any step failure before advancing to scratch media. Use original archival media only in read mode.

## Sources

- [ST UM3115 Nucleo hardware manual](https://www.st.com/resource/en/user_manual/um3115-stm32h5-nucleo144-board-mb1404-stmicroelectronics.pdf)
- [STM32H563 datasheet, alternate functions](https://www.st.com/resource/en/datasheet/stm32h563zg.pdf)
- [ST RM0481 reference manual, GPDMA request table and TIMx trigger interconnect](https://www.st.com/resource/en/reference_manual/dm00733993.pdf)
