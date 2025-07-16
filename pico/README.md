## Raspberry Pi Pico: SDK Setup and Firmware Build

### MacOS Homebrew
```shell
brew update
brew upgrade
brew install cmake gcc-arm-embedded
```

### Download the Pico SDK
```shell
git clone --recurse-submodules https://github.com/raspberrypi/pico-sdk.git
export PICO_SDK_PATH=$(realpath pico-sdk)
```

### Build the firmware for Apple II+ Video
```shell
git clone https://github.com/inindev/AppleII-VGA.git
cd ~/AppleII-VGA/pico
mkdir build
cd build
cmake -DAPPLE_MODEL=IIPLUS -DCMAKE_BUILD_TYPE=Release ..
make
$ ls *uf2
...
apple-iiplus-vga.uf2
```

### Build the firmware for Apple IIe Video
```shell
git clone https://github.com/inindev/AppleII-VGA.git
cd ~/AppleII-VGA/pico
mkdir build
cd build
cmake -DAPPLE_MODEL=IIE -DCMAKE_BUILD_TYPE=Release ..
make
$ ls *uf2
...
apple-iie-vga.uf2
```

### Install the firmware
Hold down the BOOTSEL button and connect the Raspberry Pi Pico to your PC via micro USB cable. Once Pico is
connected release the BOOTSEL button. Pi Pico should be connected to PC with USB mass storage device mode.

A disk volume called RPI-RP2 will appear on your computer. Drag and drop the apple-ii-vga.uf2 file to that volume.
RPI-RP2 will unmount and Pico will start the program.

### Stand-alone test pattern image
There's a test pattern image in the source code that one can set to display immediately at power on, and Pico
microcontroller can be powered entirely from USB. To enable test pattern just uncomment `RENDER_TEST_PATTERN`
flag in `render.h`:
```
// #define RENDER_TEST_PATTERN
```
![Test Pattern](../docs/images/test_pattern.jpg)
