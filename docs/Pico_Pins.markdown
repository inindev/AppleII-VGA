# Raspberry Pi Pico Pin Mapping

| Pin | Name       | Description                                                                | Notes                                  |
|-----|------------|----------------------------------------------------------------------------|----------------------------------------|
| 1   | GPIO0      | Connected to 74LVC245 (D0, A0, or A8, depending on OE)                     | OE controlled by GPIO11, 12, or 13     |
| 2   | GPIO1      | Connected to 74LVC245 (D1, A1, or A9, depending on OE)                     | OE controlled by GPIO11, 12, or 13     |
| 3   | GND        | System ground                                                              | Connected to Apple II GND (pin 26)     |
| 4   | GPIO2      | Connected to 74LVC245 (D2, A2, or A10, depending on OE)                    | OE controlled by GPIO11, 12, or 13     |
| 5   | GPIO3      | Connected to 74LVC245 (D3, A3, or A11, depending on OE)                    | OE controlled by GPIO11, 12, or 13     |
| 6   | GPIO4      | Connected to 74LVC245 (D4, A4, or A12, depending on OE)                    | OE controlled by GPIO11, 12, or 13     |
| 7   | GPIO5      | Connected to 74LVC245 (D5, A5, or A13, depending on OE)                    | OE controlled by GPIO11, 12, or 13     |
| 8   | GND        | System ground                                                              | Connected to Apple II GND (pin 26)     |
| 9   | GPIO6      | Connected to 74LVC245 (D6, A6, or A14, depending on OE)                    | OE controlled by GPIO11, 12, or 13     |
| 10  | GPIO7      | Connected to 74LVC245 (D7, A7, or A15, depending on OE)                    | OE controlled by GPIO11, 12, or 13     |
| 11  | GPIO8      | DEVICE SELECT' (Apple II pin 41)                                           | Connected to Apple II DEVSEL' (pin 41) |
| 12  | GPIO9      | R/W' (Apple II pin 18)                                                     | Connected to Apple II R/W' (pin 18)    |
| 13  | GND        | System ground                                                              | Connected to Apple II GND (pin 26)     |
| 14  | GPIO10     | SYNC' (Apple II pin 19)                                                    | Connected to Apple II SYNC' (pin 19)   |
| 15  | GPIO11     | Output Enable for U3 D0-D7                                                 | 74LVC245 transceiver for D0-D7         |
| 16  | GPIO12     | Output Enable for U2 A0-A7                                                 | 74LVC245 transceiver for A0-A7         |
| 17  | GPIO13     | Output Enable for U1 A8-A15                                                | 74LVC245 transceiver for A8-A15        |
| 18  | GND        | System ground                                                              | Connected to Apple II GND (pin 26)     |
| 19  | GPIO14     | VGA_B 2000 output                                                          | VGA Blue (pin 5)                       |
| 20  | GPIO15     | VGA_B 1000 output                                                          | VGA Blue (pin 5)                       |
| 21  | GPIO16     | VGA_B 500 output                                                           | VGA Blue (pin 5)                       |
| 22  | GPIO17     | VGA_G 2000 output                                                          | VGA Green (pin 3)                      |
| 23  | GND        | System ground                                                              | Connected to Apple II GND (pin 26)     |
| 24  | GPIO18     | VGA_G 1000 output                                                          | VGA Green (pin 3)                      |
| 25  | GPIO19     | VGA_G 500 output                                                           | VGA Green (pin 3)                      |
| 26  | GPIO20     | VGA_R 2000 output                                                          | VGA Red (pin 1)                        |
| 27  | GPIO21     | VGA_R 1000 output                                                          | VGA Red (pin 1)                        |
| 28  | GND        | System ground                                                              | Connected to Apple II GND (pin 26)     |
| 29  | GPIO22     | VGA_R 500 output                                                           | VGA Red (pin 1)                        |
| 30  | RUN        | Reset Switch (optional)                                                    | System Reset                           |
| 31  | GPIO26     | φ0 (Apple II pin 40)                                                       | Connected to Apple II PHI0 (pin 40)    |
| 32  | GPIO27     | VSYNC VGA signal                                                           | VGA Vertical Sync pin 8                |
| 33  | AGND       | Analog ground                                                              | Connected to Apple II GND (pin 26)     |
| 34  | GPIO28     | HSYNC VGA signal                                                           | VGA Horizontal Sync pin 7              |
| 35  | ADC_VREF   | Not Connected                                                              | N/C                                    |
| 36  | 3V3        | 3.3v power output                                                          | Regulated 3.3v Output                  |
| 37  | 3V3_EN     | Not Connected                                                              | N/C                                    |
| 38  | GND        | System ground                                                              | Connected to Apple II GND (pin 26)     |
| 39  | VSYS       | DMG2305UX-7 sink                                                           |                                        |
| 40  | VBUS       | DMG2305UX-7 gate                                                           |                                        |


