This guide describes how to use a Raspberry Pi to flash custom firmware to a hoverboard motor controller. If you don't have a USB flasher (ST-Link) and you don't want to wait for one to arrive, this guide is for you!

## Step 1: Install an OS

Install [Raspberry Pi OS Lite](https://www.raspberrypi.com/software/operating-systems/) to your Raspberry Pi 1/2/3/4

## Step 2: Install OpenOCD

There's an OpenOCD package, but it doesn't support SWD over GPIO pins (sysfsgpio). Instead, we need to compile OpenOCD from source.

Clone the repository:
```sh
git clone https://git.code.sf.net/p/openocd/code openocd-code
```

TODO: Dependency installation

Compile:
```sh
cd openocd-code
./bootstrap
./configure --enable-sysfsgpio --enable-bcm2835gpio  # Enable support for Raspberry Pi GPIO pins
make
sudo make install
```

## Step 3: Firmware

### Emanuel Feru's FOC firmware


The best firmware with many options and advanced motor control. Check if your board is supported: [your board is supported](https://github.com/EFeru/hoverboard-firmware-hack-FOC/wiki/Firmware-Compatibility).

```sh
git clone https://github.com/EFeru/hoverboard-firmware-hack-FOC
cd hoverboard-firmware-hack-FOC
sudo apt install gcc-arm-none-eabi
make
```

### Florian's gen2 firmware


Alternative firmware, for dual sideboard models (GD32). No field-oriented control (FOC) and way fewer features. Maximilian has a fork with PlatformIO build support (instead of the proprietary Keil development environment).

Install [PlatformIO Core (CLI)](https://docs.platformio.org/en/stable/core/index.html). They provide an easy installation script that installs PlatformIO in a virtual environment. Run the following command as a regular user:

```sh
python3 -c "$(curl -fsSL https://raw.githubusercontent.com/platformio/platformio/master/scripts/get-platformio.py)"
```

`pio` is now installed to `~/.platformio/penv/bin/pio`

```sh
git clone https://github.com/maxgerhardt/Hoverboard-Firmware-Hack-Gen2
cd Hoverboard-Firmware-Hack-Gen2/HoverBoardGigaDevice
~/.platformio/penv/bin/pio run
```

You may need to modify `platformio.ini` to manually specify a newer version of toolchain-gccarmnoneeabi, if the current version is not available for your architecture:
```
platform_packages = maxgerhardt/framework-spl@2.10301.0, toolchain-gccarmnoneeabi@~1.90301.0
```

## Step 4: Wiring
Connect SWD io and clock to Raspberry PI pins 11 and 25. Not sure about the order, try one and if it doesn't work, swap. them. Also connect ground. Do not connect 3v3, this will overload the Raspberry Pi's power supply. Instead, you should connect the board to a battery or other 30-40V power supply.

## Step 5: Unlocking
https://github.com/EFeru/hoverboard-firmware-hack-FOC/wiki/How-to-Unlock-MCU-Flash

Hold power button, run the command below. Release power button after command completed, and power cycle the board (disconnect battery, hold power button to discharge, connect battery).

```
openocd -f /usr/local/share/openocd/scripts/interface/sysfsgpio-raspberrypi.cfg -c "transport select swd" -f /usr/local/share/openocd/scripts/target/stm32f1x.cfg  -c init -c "reset halt" -c "stm32f1x unlock 0"
```

Example output:
```
Open On-Chip Debugger 0.12.0-rc2+dev-00022-g6ea1ccf3e (2022-12-02-10:06)
Licensed under GNU GPL v2
For bug reports, read
        http://openocd.org/doc/doxygen/bugs.html
SysfsGPIO nums: swclk = 11, swdio = 25

swd
Info : SysfsGPIO JTAG/SWD bitbang driver
Info : This adapter doesn't support configurable speed
Info : SWD DPIDR 0x1ba01477
Info : [stm32f1x.cpu] Cortex-M3 r2p1 processor detected
Info : [stm32f1x.cpu] target has 6 breakpoints, 4 watchpoints
Info : starting gdb server for stm32f1x.cpu on 3333
Info : Listening on port 3333 for gdb connections
Error: [stm32f1x.cpu] clearing lockup after double fault
Polling target stm32f1x.cpu failed, trying to reexamine
Info : [stm32f1x.cpu] Cortex-M3 r2p1 processor detected
Info : [stm32f1x.cpu] target has 6 breakpoints, 4 watchpoints
[stm32f1x.cpu] halted due to debug-request, current mode: Thread
xPSR: 00000000 p
c: 00000000 msp: 00000000
Info : device id = 0x13030410
Info : flash size = 64 KiB
stm32x unlocked.
INFO: a reset or power cycle is required for the new settings to take effect.
```

## Step 6: Flashing
Hold power button, run command, release power button.

Example for EFeru firmware:
```
openocd -f /usr/local/share/openocd/scripts/interface/sysfsgpio-raspberrypi.cfg -c "transport select swd" -f /usr/local/share/openocd/scripts/target/stm32f1x.cfg -c "program build/hover.bin verify reset exit 0x08000000"
```

Example for Florian firmware, PlatformIO:
```
openocd -f /usr/local/share/openocd/scripts/interface/sysfsgpio-raspberrypi.cfg -c "transport select swd" -f /usr/local/share/openocd/scripts/target/stm32f1x.cfg -c "program .pio/build/GD32F130C8T6/firmware.bin verify reset exit 0x08000000"
```

Example output:
```
Open On-Chip Debugger 0.12.0-rc2+dev-00022-g6ea1ccf3e (2022-12-02-10:06)
Licensed under GNU GPL v2
For bug reports, read
        http://openocd.org/doc/doxygen/bugs.html
SysfsGPIO nums: swclk = 11, swdio = 25

swd
Info : SysfsGPIO JTAG/SWD bitbang driver
Info : This adapter doesn't support configurable speed
Info : SWD DPIDR 0x1ba01477
Info : [stm32f1x.cpu] Cortex-M3 r2p1 processor detected
Info : [stm32f1x.cpu] target has 6 breakpoints, 4 watchpoints
Info : starting gdb server for stm32f1x.cpu on 3333
Info : Listening on port 3333 for gdb connections
Error: [stm32f1x.cpu] clearing lockup after double fault
Polling target stm32f1x.cpu failed, trying to reexamine
Info : [stm32f1x.cpu] Cortex-M3 r2p1 processor detected
Info : [stm32f1x.cpu] target has 6 breakpoints, 4 watchpoints
[stm32f1x.cpu] halted due to debug-request, current mode: Thread
xPSR: 0x01000000 pc: 0xfffffffe msp: 0xfffffffc
** Programming Started **
Info : device id = 0x13030410
Info : flash size = 64 KiB
Warn : Adding extra erase range, 0x08003790 .. 0x080037ff
** Programming Finished **
** Verify Started **
** Verified OK **
** Resetting Target **
shutdown command invoked
```
