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
