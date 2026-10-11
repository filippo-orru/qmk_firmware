# Lily58L
<img src="https://keycapsss.com/media/image/21/2b/68/lily58l-split-keyboard-rgb-led-1.jpg" width="400">

***The layout was adapted by me ([ffactory](https://github.com/ffactory-ofcl)).***
I tried to implement the [bone layout](https://git.neo-layout.org/neo/neo-layout/wiki/Bone) in 50% hardware and 50% software. Designed for use on Mac.
You need to insert this folder in the `keyboards/lily58/keymaps` folder.

A modified Lily58 pcb, with underglow, per key rgb light and rotary encoder support.
- SK6812 Mini-E per key led's (58x) for easy soldering
- 6x SK6812 Mini led's per side for underglow
- Support for 1 rotary encoder on each side

Left encoder: volume up/down, next/previous track on RAISE layer  
Right encoder: cursor down/up, right/left on LOWER layer

* Keyboard Maintainer: BenRoe [GitHub](https://github.com/BenRoe) / [Twitter](https://twitter.com/keycapsss)
* Hardware Supported: Pro Micro, or Elite-C
* Hardware Availability: [Keycapsss.com](https://keycapsss.com)

## Personal hardware and configuration

- Left half: Elite-C, normally using the `atmel-dfu` bootloader.
- Right half: Pro Micro, normally using the `caterina` bootloader.
- Target: `lily58/light`, keymap: `lily58l`.
- Layers: `_QWERTY`, `_BONE2_4` (navigation/numpad), `_MOUSE_L`.
- Handedness is stored in each half's EEPROM (`EE_HANDS`).
- The layout relies on host-side layout customization on macOS. Preserve that configuration too.

## Fixed left USB master

**Always connect the keyboard's USB cable to the left Elite-C half.**
The right half cannot act as a USB keyboard with this firmware, although its
bootloader remains available over USB for backups and flashing.

`EE_HANDS` stores left/right identity. This keymap overrides QMK's weak
`is_keyboard_master_impl()` function to use that identity for master selection:
left is always master, and right is always slave with USB disabled. It reads
EEPROM directly because QMK initializes the master role before the cached
`is_keyboard_left()` value. `MASTER_LEFT`/`MASTER_RIGHT` alone would not force
the USB master role.

This avoids USB/VBUS detection entirely. The left half keeps USB enabled even
when the monitor dock supplies power without a laptop connected, so it can
enumerate when a laptop becomes available. USB detection timeouts and the split
watchdog are not enabled; neither half periodically resets while waiting for a
host.

Both halves must have correct EEPROM handedness: left set to left, right set to
right. Back up flash and EEPROM before flashing both halves with their respective
bootloader builds, and preserve those handedness flags. Incorrect flags can make
both halves masters or both slaves.

See [QMK split configuration](https://docs.qmk.fm/features/split_keyboard) for
handedness and standard USB detection details.

## Build without flashing

After setting up QMK's build tools:

```sh
# Right Pro Micro
qmk compile -kb lily58/light -km lily58l -e BOOTLOADER=caterina
# Left Elite-C
qmk compile -kb lily58/light -km lily58l -e BOOTLOADER=atmel-dfu
```

Both commands produce `lily58_light_lily58l.hex`. Save each result under a
controller-specific name before building the other, since it overwrites the same file.

For build-only use on macOS, Docker avoids installing the AVR toolchain:

```sh
docker run --rm -v "$PWD:/qmk_firmware" -w /qmk_firmware \
  ghcr.io/qmk/qmk_cli qmk compile -kb lily58/light -km lily58l \
  -e BOOTLOADER=caterina
```

Use `BOOTLOADER=atmel-dfu` instead for the left half. Docker on macOS does not
provide direct access to the keyboard's USB bootloader; use native tools for readback and flashing.

## Backup before flashing

Do not flash or erase either half until its current flash and EEPROM have been
read back and verified. A fresh build from this repository is not a backup of
the currently installed firmware.

1. Confirm the bootloader on each controller. Entering it requires the PCB reset button.
2. Connect each half directly over USB, one at a time. Disconnect USB power before
   connecting or disconnecting the TRRS cable.
3. Read flash and EEPROM using native tools (`dfu-programmer` for the Elite-C's
   usual Atmel DFU bootloader, `avrdude` for the Pro Micro's usual Caterina bootloader).
   Readback may be blocked by controller protection; never erase to unlock it.
4. Save separate left/right backups, repeat reads and compare them, and record
   SHA-256 checksums, controller/bootloader details, and restore instructions.
5. Keep backups outside the build output and preserve EEPROM handedness when flashing.

The pre-update source is retained on branch `backup/pre-upstream-2026-04-27`.
It contained a `COLON:` syntax error, so do not assume that source matches the
working firmware installed on the keyboard.

See the [QMK documentation](https://docs.qmk.fm/) for build setup and flashing guidance.
