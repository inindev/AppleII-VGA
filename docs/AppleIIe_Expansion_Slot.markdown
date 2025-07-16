# Apple IIe Expansion Slot Signals

| Pin  | Name            | Description                                                                                     | Pico GPIO        |
|------|-----------------|-------------------------------------------------------------------------------------------------|------------------|
| 1    | I/O SELECT      | Normally high; goes low during φ0 when 65C02 addresses $CnXX (n = connector number).            |                  |
| 2-9  | A0-A7           | Three-state address bus (low byte). Valid during φ1, remains valid during φ0.                   | GPIO0-7 (via 74LVC245, OE GPIO12) |
| 10-17| A8-A15          | Three-state address bus (high byte). Valid during φ1, remains valid during φ0.                  | GPIO0-7 (via 74LVC245, OE GPIO13) |
| 18   | R/W'            | Three-state read/write line. High for read, low for write. Valid with address bus.              | GPIO9            |
| 19   | SYNC'           | Composite horizontal/vertical sync (slot 7 only).                                               | GPIO10           |
| 20   | I/O STROBE'     | Normally high; goes low during φ0 when 65C02 addresses $C800-$CFFF.                             |                  |
| 21   | RDY             | Input to 65C02. Low during φ1 halts 65C02, address bus holds current fetch address.             |                  |
| 22   | DMA'            | Input to address bus buffers. Low during φ1 disconnects 65C02 from address bus.                 |                  |
| 23   | INT OUT         | Interrupt priority daisy-chain output. Usually connected to pin 28 (INT IN).                    |                  |
| 24   | DMA OUT         | DMA priority daisy-chain output. Usually connected to pin 27 (DMA IN).                          |                  |
| 25   | +5V             | +5V power supply. 500mA total available for all peripheral cards.                               | VCC              |
| 26   | GND             | System common ground.                                                                           | GND              |
| 27   | DMA IN          | DMA priority daisy-chain input. Usually connected to pin 24 (DMA OUT).                          |                  |
| 28   | INT IN          | Interrupt priority daisy-chain input. Usually connected to pin 23 (INT OUT).                    |                  |
| 29   | NMI'            | Non-maskable interrupt to 65C02. Low starts interrupt cycle at $03FB.                           |                  |
| 30   | IRQ'            | Interrupt request to 65C02. Low starts interrupt cycle at $03FE if I-flag not set.              |                  |
| 31   | RES'            | Low initiates reset routine (see Chapter 4).                                                    |                  |
| 32   | INH'            | Low during φ1 disables main circuit board memory.                                               |                  |
| 33   | -12V            | -12V power supply. 200mA total available for all peripheral cards.                              | N/A              |
| 34   | -5V             | -5V power supply. 200mA total available for all peripheral cards.                               | N/A              |
| 35   | 3.58M           | 3.58 MHz color reference signal (slot 7 only).                                                  |                  |
| 36   | 7M              | System 7 MHz clock.                                                                             |                  |
| 37   | Q3              | System 2 MHz asymmetrical clock.                                                                |                  |
| 38   | φ1              | 65C02 phase 1 clock.                                                                            |                  |
| 39   | μPSYNC          | 65C02 signals operand fetch by driving high during first read cycle of each instruction.        |                  |
| 40   | φ0              | 65C02 phase 0 clock.                                                                            | GPIO26           |
| 41   | DEVICE SELECT'  | Normally high; goes low during φ0 when 65C02 addresses $COnX (n = connector number + 8).        | GPIO8            |
| 42-49| D0-D7           | Three-state buffered bi-directional data bus. Valid during φ0 high, remains until φ0 low.       | GPIO0-7 (via 74LVC245, OE GPIO11) |
| 50   | +12V            | +12V power supply. 250mA total available for all peripheral cards.                              | N/A              |

**Notes:**
- **Power/Ground**: +5V connects to Pico’s VCC (3.3V adjusted via regulator), GND to Pico’s ground. ±12V and -5V are N/A as Pico doesn’t support these voltages.
- **Slot 7 Signals**: SYNC' (GPIO10) is slot 7-specific, but 3.58M is unmapped.
- **Transceiver Use**: GPIO0-7 are reused for A0-A7, A8-A15, and D0-D7 via separate 74LVC245 transceivers, with OE controlled by GPIO11 (data), GPIO12 (A0-A7), and GPIO13 (A8-A15).
