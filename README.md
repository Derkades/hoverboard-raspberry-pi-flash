This guide describes how to use a Raspberry Pi to flash custom firmware to a hoverboard motor controller. If you don't have a USB flasher (ST-Link) and you don't want to wait for one to arrive, this guide is for you!

## Step 1: Install an OS

Install [Raspberry Pi OS Lite](https://www.raspberrypi.com/software/operating-systems/) to your Raspberry Pi 1/2/3/4

## Step 2: Install OpenOCD

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
TODO

## Step 5: Unlocking
TODO

## Step 6: Flashing
TODO
