## Summary of the `abus` Program

The `abus` program is a PIO (Programmable Input/Output) application for the Raspberry Pi Pico’s RP2040 microcontroller, designed to interface with the Apple II bus, a hardware interface used in Apple II computers for memory and peripheral communication. The program leverages the RP2040’s PIO subsystem to handle low-level, high-speed bus operations, capturing address and data signals from the Apple II bus while maintaining precise timing. This summary explains the purpose, implementation, and key details of the `abus` program, incorporating insights from questions about PIO, input pins, SET pins, and shift/autopush configurations. It is intended for someone new to the code and the PIO concept, providing a clear understanding of how the program works and why PIO is used.

### Purpose of the `abus` Program

The `abus` program interfaces the Raspberry Pi Pico with the Apple II bus to monitor and shadow bus transactions, such as memory writes and soft switch accesses. It captures the 16-bit address (`AddrHi[7:0]`, `AddrLo[7:0]`), control signals (`~DEVSEL`, `R/W`), and data (`Data[7:0]`) during each bus cycle, synchronized with the Apple II’s `PHI0` clock signal. The program runs on a PIO state machine, which processes bus signals in real-time and pushes 26-bit words to a FIFO buffer for the CPU to analyze. This enables applications like emulating Apple II peripherals, shadowing memory for video output, or detecting CPU resets. The program is tailored for the Apple II’s 1 MHz bus, with references to *Understanding the Apple II* (pages 4-7, 7-8) for bus timing and signal details.

### What is PIO?

The Programmable Input/Output (PIO) subsystem is a unique feature of the RP2040 microcontroller, consisting of four independent state machines per PIO block that can execute small, custom programs to control GPIO pins with precise timing. Unlike the CPU, which may be interrupted or slowed by other tasks, PIO state machines run independently at high speed (up to 125 MHz, or 8 ns per instruction in this case), making them ideal for real-time interfacing with external hardware like the Apple II bus. PIO programs use a simple assembly-like language to perform tasks such as reading/writing pins, shifting data, and managing timing, offloading complex I/O operations from the CPU. In the `abus` program, PIO is used to synchronize with the `PHI0` clock, control transceivers, and read bus signals efficiently.

### Implementation Details

The `abus` program and its C setup code (`abus.c`) configure a PIO state machine to handle Apple II bus cycles. Below are the key components, addressing questions about input pins, SET pins, shift/autopush, and the overall setup.

#### 1. Pin Configuration

- **Input Pins (10 pins, GPIO0–9)**: The program reads `~DEVSEL` (device select, active low, GPIO8), `R/W` (read/write, GPIO9), and `Data[7:0]` (data/address bus, GPIO0–7) from the Apple II bus. These pins are connected to a 74LVC245 transceiver, which outputs either `D0-D7` (data), `A0-A7` (low address), or `A8-A15` (high address) based on the output enable (OE) signals controlled by GPIO11–13. The C function `sm_config_set_in_pins(&c, CONFIG_PIN_APPLEBUS_DATA_BASE)` sets the base pin (GPIO0), and the program implicitly uses 10 pins because the `in PINS, 10` instruction reads 10 bits (`~DEVSEL`, `R/W`, `Data[7:0]`). The C code explicitly initializes GPIO0–9 as inputs with `pio_gpio_init` and disables pull resistors for clean signal reading. Input synchronization is bypassed (`pio->input_sync_bypass`) to reduce latency, as signals are sampled at known stable times.

- **SET Pins (3 pins, GPIO11–13)**: These control the output enable (OE) signals of three 74LVC245 transceivers: GPIO11 (U3, Data), GPIO12 (U2, AddrLo), and GPIO13 (U1, AddrHi). The C function `sm_config_set_set_pins(&c, CONFIG_PIN_APPLEBUS_CONTROL_BASE, 3)` specifies 3 pins, and `set PINS, <value>` instructions (e.g., `set PINS, 0b110`) set these pins to enable/disable transceivers (active low). For example, `set PINS, 0b110` sets GPIO13=1 (AddrHi off), GPIO12=1 (AddrLo off), GPIO11=0 (Data on).

- **Other Pins**: GPIO26 (`PHI0`, bus clock) is an input for synchronization (`wait` instructions), and GPIO9 is also the jump pin for `jmp PIN, read_cycle` to check `R/W`.

#### 2. PIO Program Operation

The program loops per bus cycle, synchronized with `PHI0` (GPIO26):

- **Address Phase**: `set PINS, 0b011` enables the `AddrHi` transceiver, `wait 1 GPIO, PHI0_GPIO` waits for `PHI0` to rise, and `in PINS, 8` reads `AddrHi[7:0]` (GPIO0–7). Then, `set PINS, 0b101 [5]` enables `AddrLo`, and `in PINS, 8` reads `AddrLo[7:0]`.

- **Branching**: `jmp PIN, read_cycle` checks `R/W` (GPIO9) to branch to `read_cycle` (R/W high) or `write_cycle` (R/W low).

- **Write Cycle**: `set PINS, 0b110 [15]` enables the `Data` transceiver, `in PINS, 10` reads `~DEVSEL`, `R/W`, `Data[7:0]`, and `wait 0 GPIO, PHI0_GPIO [7]` waits for `PHI0` to fall.

- **Read Cycle**: `in PINS, 10` reads `~DEVSEL`, `R/W`, `dontcare[7:0]`, followed by a `wait` for `PHI0` to fall.

- **Shift and Autopush**: The ISR is configured to shift left with autopush at 26 bits (`sm_config_set_in_shift(&c, false, true, 26)`). The program collects 8 bits (`AddrHi`), 8 bits (`AddrLo`), and 10 bits (`~DEVSEL`, `R/W`, `Data[7:0]`), forming a 26-bit word pushed to the FIFO.

#### 3. C Code (`abus.c`)

- **Setup (`abus_main_setup`)**: Loads the `abus` program, configures the state machine (pins, shift, clock), and initializes GPIOs. It ensures transceivers start disabled (`pio_sm_set_pins_with_mask`) and sets pin directions.

- **Loop (`abus_loop`)**: Reads 26-bit words from the FIFO, extracts `is_devsel` (bit 8), `is_write` (bit 9), `address` (bits 25–10), and `data` (bits 7–0), and processes them via `shadow_memory` (for memory writes) or `device_write` (for device register writes).

- **Soft Switches**: Functions like `shadow_softsw_*` handle Apple II soft switches (e.g., `80STORE`, `80COL`) by updating state variables when specific addresses are written, supporting features like 80-column mode or double hires graphics.

### Why Use PIO?

The PIO is ideal for the `abus` program because it:

- **Handles High-Speed Timing**: Runs at 125 MHz (8 ns/instruction) to match the Apple II’s 1 MHz bus, ensuring precise synchronization with `PHI0`.

- **Offloads CPU**: Executes bus operations independently, freeing the CPU for other tasks like processing FIFO data or updating video output.

- **Flexible Pin Control**: Manages complex pin operations (e.g., enabling transceivers, reading 10-bit signals) with minimal instructions.

- **Custom Protocols**: Implements the Apple II bus protocol, including address/data multiplexing and control signal handling, which would be challenging for the CPU alone.

### Summary

The `abus` program uses the RP2040’s PIO to interface with the Apple II bus, capturing 26-bit bus transactions (address, control, data) per cycle. It configures 10 input pins (GPIO0–9) for `~DEVSEL`, `R/W`, and `Data[7:0]`, with the count implied by `in PINS, 10`, and 3 SET pins (GPIO11–13) for transceiver control, explicitly set by `sm_config_set_set_pins`. The ISR shifts left and autopushes at 26 bits, delivering data to the CPU via the FIFO. The C code sets up the PIO, initializes pins, and processes bus data for memory shadowing and soft switch handling. The PIO’s ability to handle real-time, high-speed I/O makes it perfect for this application, simplifying complex bus interactions for the Apple II interface.

<br/>

## Diagrams: Data Storage in Memory for the `abus` PIO Program

This section provides diagrams illustrating how data is stored in memory for the `abus` PIO program on the Raspberry Pi Pico’s RP2040 microcontroller. The program interfaces with the Apple II bus, capturing 26-bit bus transactions (`AddrHi[7:0]`, `AddrLo[7:0]`, `~DEVSEL`, `R/W`, `Data[7:0]`) into the **Input Shift Register (ISR)** and pushing them to the **FIFO** buffer. The diagrams show:

1. **ISR Data Storage**: How the 26 bits are shifted into the 32-bit ISR with a left-shift configuration.
2. **FIFO Data Storage**: How the 26-bit word is pushed to the FIFO and structured for CPU access.

These diagrams address your questions about input pins (GPIO0–9), SET pins (GPIO11–13), and the ISR configuration (shift left, autopush at 26 bits), providing a clear visual representation for someone new to the `abus` program and PIO.

### Context from the `abus` Program

- **Input Pins (GPIO0–9)**:
  - GPIO0–7: `Data[7:0]` (or `AddrHi[7:0]`, `AddrLo[7:0]` depending on the transceiver).
  - GPIO8: `~DEVSEL` (device select, active low).
  - GPIO9: `R/W` (read/write signal).
- **Instructions**:
  - `in PINS, 8`: Reads 8 bits (`AddrHi[7:0]` or `AddrLo[7:0]`) from GPIO0–7.
  - `in PINS, 10`: Reads 10 bits (`~DEVSEL`, `R/W`, `Data[7:0]` or `dontcare[7:0]`) from GPIO0–9.
- **ISR Configuration**: Shift left, autopush at 26 bits (`sm_config_set_in_shift(&c, false, true, 26)`).
- **FIFO**: Receives 26-bit words (padded to 32 bits) for CPU processing via `pio_sm_get_blocking`.
- **C Code**: The `abus_loop` function extracts the address (bits 25–10), `~DEVSEL` (bit 8), `R/W` (bit 9), and data (bits 7–0).

### Diagram 1: ISR Data Storage

The ISR is a 32-bit register that accumulates data from `in PINS` instructions, shifting left. The program reads 26 bits per bus cycle in three steps:
1. `in PINS, 8` (AddrHi[7:0]).
2. `in PINS, 8` (AddrLo[7:0]).
3. `in PINS, 10` (~DEVSEL, R/W, Data[7:0]).

The diagram shows the ISR state after each `in PINS` instruction, with bits entering from the right (LSB) and shifting existing bits left.

```plaintext
Step 1: After `in PINS, 8` (AddrHi[7:0])
-----------------------------------------------------
| 31 | 30 | ... | 8 | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
-----------------------------------------------------
|  0 |  0 | ... | 0 | A7| A6| A5| A4| A3| A2| A1| A0|
-----------------------------------------------------
A7–A0 = AddrHi[7:0] (from GPIO0–7)

Step 2: After `in PINS, 8` (AddrLo[7:0])
-------------------------------------------------------------------------------------
| 31 | 30 | ... | 16| 15| 14| 13| 12| 11| 10| 9 | 8 | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
-------------------------------------------------------------------------------------
|  0 |  0 | ... | 0 | A7| A6| A5| A4| A3| A2| A1| A0| L7| L6| L5| L4| L3| L2| L1| L0|
-------------------------------------------------------------------------------------
A7–A0 = AddrHi[7:0], L7–L0 = AddrLo[7:0] (from GPIO0–7)

Step 3: After `in PINS, 10` (~DEVSEL, R/W, Data[7:0])
-------------------------------------------------------------------------------------------------------------------------------------------------------
| 31 | 30 | 29 | 28 | 27 | 26 | 25 | 24 | 23 | 22 | 21 | 20 | 19 | 18 | 17 | 16 | 15 | 14 | 13 | 12 | 11 | 10 | 9 | 8 | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
-------------------------------------------------------------------------------------------------------------------------------------------------------
|  0 |  0 |  0 |  0 |  0 |  0 | A7 | A6 | A5 | A4 | A3 | A2 | A1 | A0 | L7 | L6 | L5 | L4 | L3 | L2 | L1 | L0 | DS| RW| D7| D6| D5| D4| D3| D2| D1| D0|
-------------------------------------------------------------------------------------------------------------------------------------------------------
A7–A0 = AddrHi[7:0], L7–L0 = AddrLo[7:0], DS = ~DEVSEL (GPIO8), RW = R/W (GPIO9), D7–D0 = Data[7:0] (GPIO0–7)
```

**Explanation**:
- **Shift Left**: Each `in PINS` instruction shifts new bits into the ISR from the right (LSB, bit 0), pushing existing bits left.
- **Step 1**: `in PINS, 8` reads `AddrHi[7:0]` (GPIO0–7), placing it in ISR bits 7–0.
- **Step 2**: `in PINS, 8` reads `AddrLo[7:0]` (GPIO0–7), shifting `AddrHi` to bits 15–8 and placing `AddrLo` in bits 7–0.
- **Step 3**: `in PINS, 10` reads `~DEVSEL` (GPIO8), `R/W` (GPIO9), and `Data[7:0]` (GPIO0–7), shifting `AddrHi` to bits 25–18, `AddrLo` to bits 17–10, and placing `~DEVSEL`, `R/W`, `Data[7:0]` in bits 9–0.
- **Autopush**: After 26 bits are accumulated, the ISR’s contents (bits 25–0) are pushed to the FIFO, and the ISR is cleared.

### Diagram 2: FIFO Data Storage

The FIFO receives the 26-bit word from the ISR, padded to 32 bits (6 MSBs set to 0). The CPU reads this word using `pio_sm_get_blocking`, and the `abus_loop` function extracts fields:
- Address: Bits 25–10 (16-bit address, `AddrHi[7:0] | AddrLo[7:0]`).
- `~DEVSEL`: Bit 9.
- `R/W`: Bit 8.
- Data: Bits 7–0 (`Data[7:0]` or `dontcare[7:0]`).

```plaintext
FIFO Word (32 bits, after autopush)
-------------------------------------------------------------------------------------------------------------------------------------------------------
| 31 | 30 | 29 | 28 | 27 | 26 | 25 | 24 | 23 | 22 | 21 | 20 | 19 | 18 | 17 | 16 | 15 | 14 | 13 | 12 | 11 | 10 | 9 | 8 | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
-------------------------------------------------------------------------------------------------------------------------------------------------------
|  0 |  0 |  0 |  0 |  0 |  0 | A7 | A6 | A5 | A4 | A3 | A2 | A1 | A0 | L7 | L6 | L5 | L4 | L3 | L2 | L1 | L0 | DS| RW| D7| D6| D5| D4| D3| D2| D1| D0|
-------------------------------------------------------------------------------------------------------------------------------------------------------
- Bits 31–26: 0 (padding)
- Bits 25–18: AddrHi[7:0] (A8-A15)
- Bits 17–10: AddrLo[7:0] (A0-A7)
- Bit 9: ~DEVSEL (GPIO8, 0 = device selected)
- Bit 8: R/W (GPIO9, 0 = write, 1 = read)
- Bits 7–0: Data[7:0] (D0-D7, or dontcare[7:0] for read cycles)
```

**Explanation**:
- **FIFO Structure**: The 26-bit ISR word is pushed to the FIFO, padded to 32 bits with zeros in the MSBs. The FIFO is a 4-word buffer, but each bus cycle typically produces one word.
- **CPU Access**: The `abus_loop` function reads the 32-bit word:
  ```c
  uint32_t value = pio_sm_get_blocking(CONFIG_ABUS_PIO, ABUS_MAIN_SM);
  const bool is_devsel = ((value & (1u << (CONFIG_PIN_APPLEBUS_DEVSEL - CONFIG_PIN_APPLEBUS_DATA_BASE))) == 0); // Bit 8
  const bool is_write = ((value & (1u << (CONFIG_PIN_APPLEBUS_RW - CONFIG_PIN_APPLEBUS_DATA_BASE))) == 0); // Bit 9
  uint_fast16_t address = (value >> 10) & 0xffff; // Bits 25–10
  uint_fast8_t data = value & 0xff; // Bits 7–0
  ```
- **Processing**: The CPU uses `is_devsel`, `is_write`, `address`, and `data` to handle memory shadowing (`shadow_memory`) or device writes (`device_write`), and updates soft switches via `shadow_softsw_*` functions.

### Pin Mapping and Data Flow

- **Input Pins (GPIO0–9)**: Configured with `sm_config_set_in_pins(&c, CONFIG_PIN_APPLEBUS_DATA_BASE)` (base = GPIO0). The 74LVC245 transceivers, controlled by SET pins (GPIO11–13), select whether GPIO0–7 carry `AddrHi[7:0]`, `AddrLo[7:0]`, or `Data[7:0]`. GPIO8 (`~DEVSEL`) and GPIO9 (`R/W`) are fixed control signals.
- **SET Pins (GPIO11–13)**: Configured with `sm_config_set_set_pins(&c, CONFIG_PIN_APPLEBUS_CONTROL_BASE, 3)` (base = GPIO11). Instructions like `set PINS, 0b110` control transceivers (e.g., GPIO11=0 enables Data).
- **Data Flow**:
  - GPIO0–7 → Transceiver (U1/U2/U3) → ISR (via `in PINS, 8` or `in PINS, 10`).
  - GPIO8–9 → ISR (via `in PINS, 10`).
  - ISR → FIFO (autopush at 26 bits).
  - FIFO → CPU (via `pio_sm_get_blocking`).

### Why This Structure?

- **Shift Left**: Ensures `AddrHi[7:0]` occupies higher-order bits (25–18), followed by `AddrLo[7:0]` (17–10), `~DEVSEL` (9), `R/W` (8), and `Data[7:0]` (7–0), matching the CPU’s expected format.
- **Autopush at 26 Bits**: Automates pushing the 26-bit word (`8 + 8 + 10`) to the FIFO, streamlining data transfer.
- **10 Input Pins**: The `in PINS, 10` instruction requires 10 pins (GPIO0–9), implicitly set by the program’s logic and confirmed by the C code’s initialization loop.

### Documentation References

- **RP2040 Datasheet**: Section 3.5 (PIO) for ISR and FIFO details: [RP2040 Datasheet](https://www.raspberrypi.com/documentation/microcontrollers/silicon.html#rp2040-2).
- **Pico C/C++ SDK**: For `sm_config_set_in_shift` and `pio_sm_get_blocking`: [Pico SDK](https://www.raspberrypi.com/documentation/pico-sdk/hardware.html#group_hardware_pio).
