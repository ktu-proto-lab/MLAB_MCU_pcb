# MLAB_MCU Test PCB

Test board for the taped-out `MLAB_MCU` (top level: `ibex_simple_system`), packaged in a
QFN32 / MLP5X5-32 open-cavity package.

Two board versions, one schematic project:

| Version | Purpose | Notes |
|---|---|---|
| **A - Socketed (Priority)** | Screening / characterisation of multiple dies | Full instrumentation.  |
| **B - Direct solder (Future)** | Demo, long-term lab setup, student projects | Same schematic, instrumentation depopulated (fixed LDOs, fixed oscillator). |

---

## 1. Chip summary

- Ibex RV32IMC core, Wishbone shared interconnect
- IMEM 8 kB, DMEM 4 kB
- Peripherals: I2C master, UART master, GPIO(10), timer
- Bootloader loads firmware from an external I2C EEPROM (24CS512), writes back SRAM contents to another region of the EEPROM, then releases `rst_core`
- Full scan chain, enabled by `test_mode` (**active high**)
- Timing closed at **80 MHz** (`clk_sys`)

### Top-level ports (post-PnR netlist)

```
VDD, VSS, VDDPAD, VSSPAD       // 1.2 V core, 3.3 V pad ring
clk_sys_Pad                    // input
rst_sys_n_Pad                  // input, active low
SDA_Pad, SCL_Pad               // inout
ext_pad[9:0]                   // inout
test_mode_Pad                  // input, active high
```

### Pin function map

| Pin | `test_mode = 0` | `test_mode = 1` |
|---|---|---|
| `ext_pad[0]` | GPIO0/UART_RX | - |
| `ext_pad[1]` | GPIO1/UART_TX | - |
| `ext_pad[2]` | GPIO2 | **scan_in (sdi)** |
| `ext_pad[3]` | GPIO3 | **scan_out (sdo)** |
| `ext_pad[4]` | GPIO4 | **scan_en** |
| `ext_pad[5..9]` | GPIO5..9 | - |
| `clk_sys_Pad` | system clock | scan shift clock |
| `rst_sys_n_Pad` | system reset | scan reset |

### Packaging / grounds

- Package base contact = **GND**. All unused package pins are bonded to the base.
- `VSSCORE` and `VSSPAD` are connected over this same base contact - **a single ground on the PCB**.

---

## 2. Power

Two rails, common ground.

| Rail | Nominal | Target accuracy at the pin |
|---|---|---|
| `VDD` (core) | 1.2 V | ±2 % from the regulator, ±5 % worst case at the pin |
| `VDDPAD` (I/O) | 3.3 V | ±2 % / ±5 % |

Rationale: timing libraries are characterised at ±10 %, and the worst IR drop seen in power
analysis was 2 mV. A regulator with ≤1 % initial accuracy and a 0.1 % feedback divider
(ADP7118 / TPS7A47 / TLV757 class) leaves plenty of margin for droop, ripple and transients.
**Requirements**

- Both rails brought out to a 2-pin header so a bench SMU can replace the on-board LDO
  (for a VDD-vs-Fmax test).
- **Current sense:** high-side shunt + INA226 on each rail,
  so core and pad-ring current are separated even though ground is shared.
- **Sequencing:** chain the LDO enables (3.3 V first, then 1.2 V, or vice versa) with a jumper
  to reverse the order so the sequence can actually be tested. If the pad ring powers up
  while the core is dead, level shifters can inject current into the unpowered core.
- **Decoupling**: 100 nF per supply pin, placed at the package. 1–10 µF bulk per rail.
  Low-ESR bulk at each LDO output. 0 Ω footprint (0603, bead-compatible) in series with
  each LDO input.

---

## 3. Clock

`clk_sys` closed timing at 80 MHz, so the clock net is the fastest thing on the board.

**Requirements**

- Fixed 3.3 V CMOS oscillator footprint (populate 25 MHz for bring-up, 80 MHz for characterisation).
- Header for an external clock (FPGA-driven, or bench generator).
- Selection by **jumper**. Place the jumper immediately at the source, keep
  the trace to `clk_sys_Pad` short (< ~30 mm), and put a **33 Ω series resistor at the source**.
  Ground return directly underneath the clock trace.


---

## 4. Reset

- Push button + pull-up + debounce capacitor.
- **Open-drain drive point from the FPGA**, so reset can be asserted under software
  control. Required for automated EEPROM reprogramming and shmoo loops.
- Test point on the reset net.

---

## 5. test_mode and scan

`test_mode` is **active high**.

- 10 kΩ pull-**down** on `test_mode_Pad`, plus a header to force it high and a route to the
  FPGA header so scan can be entered under software control. Series resistor, nothing else
  drives this net.
- Scan pins `ext_pad[2]` (sdi), `ext_pad[3]` (sdo), `ext_pad[4]` (scan_en) must stay
  **electrically clean**:
  - no LED, no pull-up/down, no debounce cap on these three
  - series resistor 33 Ω max
  - short, direct route to the FPGA header, ground return alongside

- Note: scan shifting uses clk_sys, so the external clock path must pass DC-to-80 MHz.

---

## 6. Boot EEPROM and I2C

- **Socket** for the 24CS512 (SOIC-to-DIP adapter into a DIP-8 socket, or a small daughterboard
  on a header so firmware images can be swapped by swapping boards).
- **Header** in parallel with the EEPROM for FPGA emulation of the I2C slave.
  Convention: *if the header is used, the EEPROM is removed.* No bus switch, no isolation logic.
- Pull-ups: **4.7 kΩ** on SDA and SCL. TODO: If bootloads in fast mode need another set of **4.7 kΩ** as DNP in parallel.
- Jumpers on EEPROM `A0..A2` and `WP`.
- Test points on SDA and SCL, plus a ground probe loop next to them.
- **Sensor header:** a 4-pin (3V3 / GND / SDA / SCL) header on the same bus so an I2C sensor
  breakout can be plugged in for peripheral testing. Nothing populated by default.

---

## 7. GPIO and UART

- Series resistor 33–100 Ω on every `ext_pad` (except the three scan pins - see §5, 33 Ω max).
- LEDs driven through a buffer (74HC244 from 3.3 V), each behind a solder jumper / DNP resistor
  so it can be lifted. Never drive an LED directly from the pad ring.
- Test point on every `ext_pad`.
- Every `ext_pad` also routed to the FPGA header.
- UART: route RX/TX to a USB bridge (can include in PCB or just headers for external connection)

---

## 8. FPGA / host interface

- One header carrying: `clk_sys`, `rst_sys_n`, `test_mode`, all 10 `ext_pad`, SDA, SCL, 3V3, GND.
- If we use the PYNQ-Z2 3.3 V logic on both sides, so no level shifting needed.

---

## 9. Protection

- **Power-up order matters: the board must be powered before the FPGA / USB bridge is connected.**
  An external driver holding a pin high while the board is unpowered injects current into the pad
  ESD diodes and can latch up the chip on power-up. Silkscreen this on the board next to the
  header.
- Series resistors on all externally driven inputs (already covered above).
- **TVS on the FPGA/USB connectors: optional**

---

## 10. PCB and mechanical

- 4-layer, with solid ground plane on L2 ??
- Package base pad = GND, make sure to add contact for it
- **Silkscreen the pin function map (§1) on the board.**

---

## 11. Bring-up order

1. Bare board: check rails, LDO sequencing, no shorts. Chip not fitted.
2. Fit chip. Power up with reset held, no clock. Measure quiescent current on both rails.
3. Reset still held.  Apply a slow clock and step it (1, 5, 10 MHz). Core current should rise roughly linearly with frequency, if not - clock isn't reaching the die. If jumps to something large - contention or latch-up.
4. T1 from §12. `test_mode = 1`: shift a known pattern through sdi -> sdo. This proves the die is alive and the
   pad ring works, before any firmware exists.
5. EEPROM fitted. Release reset, verify writeback works.(Can skip to GPIO example)
6. `test_mode = 0`: EEPROM fitted with a blink program. Release reset, watch a GPIO toggle.
7. UART hello-world.
8. Ramp clock to 80 MHz, then shmoo.

---

## 12. Test plan

### T1 - Scan chain integrity (structural)
Shift a known pattern (walking 1, PRBS) through sdi -> sdo at a slow clock, compare.
- **Answers:** is the die alive, is the pad ring alive, is the chain unbroken?
- **Needs:** test_mode header, clean ext_pad[4:2] to FPGA header, slow clock path.
- **Why first:** no firmware, no EEPROM, no clock tree required. Cheapest go/no-go.
- **Extension:** re-run at increasing shift frequency to find scan-shift Fmax.

### T2 - Static power
Measure VDD and VDDPAD current with reset held and clock stopped, sweeping VDD.
- **Answers:** leakage vs supply; compares against the PnR power estimate.
- **Needs:** INA226 on each rail, bench VDD.

### T3 - Dynamic power
At fixed VDD, sweep clock frequency and record core current. Repeat for a few VDD points.
- **Answers:** slope gives effective switched capacitance (I = C_eff · V · f), intercept
  gives leakage. Direct measured-vs-simulated comparison with the PnR power analysis.
- **Needs:** swept clock, INA226, a firmware workload with a stable
  activity factor (idle loop vs. busy loop gives two useful data points).

### T4 - Shmoo: Fmax vs VDD
For each (VDD, frequency) point: reset, boot, run a self-test, check pass/fail.
- **Answers:**  how much margin the 80 MHz signoff actually had.
- **Needs:** adjustable VDD, programmable clock, software-controlled reset, a fast
  pass/fail signal (see below).
- **Pass/fail signal:** firmware runs a checksum over a known computation and toggles
  a GPIO at a distinct rate on pass; anything else is a fail. Must complete in ms, not
  seconds, or the sweep takes all day.

### T5 - Boot, memory, and writeback
The bootloader loads firmware from EEPROM into IMEM/DMEM, then reads the SRAMs back out
to a separate EEPROM region. Useful for debugging if the IC doesn't work. Can also perform thorough memory integrity testing.

### T6 - Peripherals
| Peripheral | Test |
|---|---|
| GPIO | Walking-1 out, read back via FPGA header; check drive strength / rise time |
| Timer | Compare programmed interval against scope / FPGA counter; measure drift |
| UART | Sweep baud rate, find max reliable rate; check baud error at 115200 |
| I2C master | Talk to a sensor on the header; sweep SCL frequency, check timing conformance |

### T7 - Die-to-die variation
Repeat T1–T4 across all packaged parts using the socketed board.
- **Answers:** yield, spread of Fmax and leakage across dies.
- **Needs:** the socket, and the whole loop scripted end-to-end.


### T9 - Robustness
- Power sequencing: try both LDO enable orders, watch for latch-up current.
- Reset behaviour: glitch reset, brown-out VDD, check recovery.
- **Needs:** LDO enable jumper, current sense (a latch-up shows up as a current spike).

### T10 - Benchmark (paflexinimui)
CoreMark at max stable frequency.

