 [![forthebadge](https://forthebadge.com/images/badges/built-with-love.svg)](https://forthebadge.com) [![forthebadge](https://forthebadge.com/images/badges/powered-by-electricity.svg)](https://forthebadge.com) [![forthebadge](https://forthebadge.com/images/badges/mom-made-pizza-rolls.svg)](https://forthebadge.com) [![forthebadge](https://forthebadge.com/images/badges/reading-6th-grade-level.svg)](https://forthebadge.com)

# Hardware design for the RP2040/Raspi Pico

This guide requires you to have a small amount of prerequisite knowledge about PCB design and the RP2040.

I have an open source board using it in my repo [HERE](https://github.com/Sleepdealr/RPAD) Which is tested and working

## Kicad Version

To open these files, you will need a recent stable version of KiCad (6.0 or later).

## Libraries

You might want to import and clone the symbols/fps to your own libs.

- Symbol:
  - Created by me
  - `/PCB/Libraries/Sleep-lib`
- Footprint:
  - Taken from Raspi's implementation
  - `/PCB/Libraries/Footprints/RP2040-QFN-56.kicad_mod`
  - 3D Model is included

Images of the 3dview, EEschema, and PCBNew in the `/img` folder.

## Extra Resources

All the long-form information I found about the chip is in in the `/Pico-Resources` Folder

## Schematic

[PDF of schematic](PCB/RP2040-Guide.pdf)

THIS IMAGE WILL BE OUT OF DATE. LOOK AT THE PDF FOR THE LATEST VERSION.
![Schematic](img/eeschema.png)

If you want to know more about the comoponents, please read Raspberry's hardware design example.
The PDF is `/Pico-Resources/hardware-design-with-rp2040.pdf`.

Breakout for SWD/GPIO is NOT required.

## PCB

THIS IMAGE WILL BE OUT OF DATE. LOOK AT THE KiCad FILES FOR THE LATEST VERSION.
![PCB](/img/pcbnew.png)

### Placement / Routing

- Fills inside of the MCU were taken from the design example. You can just copy paste them out. Make sure to set the correct zone priority and clearances, or your fills will be all messed up. Zones are filled from high -> low numbers.

- Try and keep the decoupling capacitors close to their respective pins. You might want to break up and label them in the schematic so it's easier to handle.

- Keep the flash and crystal as close as possible to the MCU

### BOM

- Everything on the PCB can be assembled by JLC. Cost for a few units is a bit high due to all the extended components, but it scales nicely :D

- There are a few different flash sizes and packages to choose from. I would personally use the cheapest one that's in stock that fits.

## Firmware

- QMK
  - KarlK90's PR: <https://github.com/qmk/qmk_firmware/pull/14877>
  - KarlK90's Branch: <https://github.com/KarlK90/qmk_firmware/tree/feature/raspberry-pi-rp2040-support>
- VIAL Branch
  - <https://github.com/vial-kb/vial-qmk/tree/rp2040>
- KMK
  - <https://github.com/KMKfw/kmk_firmware>
- Keyberon
  - <https://github.com/TeXitoi/keyberon>

To flash, hold the USB-BOOT button down as you plug in the keyboard (like Bootmagic reset)
## Flashing QMK on Linux from CLI

You will need to install picotool with your distro's package manager of choice, or [build it manually](https://github.com/raspberrypi/picotool#building).

In order to be able to flash properly, you will need to configure a custom udev rule.

Open a terminal and execute this script.

```bash
sudo mkdir -p /etc/udev/rules.d/
echo '# /etc/udev/rules.d/99-pico.rules

# Make an RP2040 in BOOTSEL mode writable by all users, so you can `picotool`
# without `sudo`.
SUBSYSTEM=="usb", ATTRS{idVendor}=="2e8a", ATTRS{idProduct}=="0003", MODE="0666"

# Symlink an RP2040 running MicroPython from /dev/pico.
#
# Then you can `mpr connect $(realpath /dev/pico)`.
SUBSYSTEM=="tty", ATTRS{idVendor}=="2e8a", ATTRS{idProduct}=="0005", SYMLINK+="pico"", TAG+="uaccess", TAG+="udev-acl"' | sudo tee /etc/udev/rules.d/99-pico.rules
sudo udevadm control --reload-rules
sudo udevadm trigger
```

Command borrowed from xyz's VIAL docs

## Extra info

### EEPROM

While the Raspberry Pi Pico RP2040 does not come with an EEPROM onboard, we simulate one by using a single 4K chunk of flash at the end of flash space.

**Note that this is a simulated EEPROM and will only support the number of writes as the onboard flash chip, not the 100,000 or so of a real EEPROM.** Therefore, do not frequently update the EEPROM or you may prematurely wear out the flash.

Source: <https://arduino-pico.readthedocs.io/en/latest/eeprom.html>

This isn't really an issue, as the flash chip I used is rated for 100k cycles [link to datasheet](https://www.winbond.com/resource-files/w25q128jv_dtr%20revc%2003272018%20plus.pdf)
