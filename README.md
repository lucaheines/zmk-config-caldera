# Caldera ZMK configuration

Firmware for the original [Caldera PCB design](https://github.com/christianselig/caldera-keyboard) with nice!nano v2 or pin-compatible controllers, using ZMK v0.3.

## Flash the normal firmware first

1. Open this repository's latest successful **Build ZMK firmware** run in GitHub Actions and download its **firmware** artifact.
2. Unzip the download. Use `caldera_right.uf2` for the right half and `caldera_left.uf2` for the left half.
3. Connect the right half with a USB data cable. Double-tap its reset button to enter the UF2 bootloader. A drive usually named `NICENANO` appears.
4. Copy `caldera_right.uf2` to that drive. The drive disappears when the controller restarts.
5. Repeat with the left half and `caldera_left.uf2`.
6. Power both halves and connect the left half to the computer over USB for the first typing test.

The left half is the central half and sends keys to the computer. Connecting only the right half over USB does not make it a standalone USB keyboard. Keep the right half powered to use its keys.

Firmware flashing does not intentionally erase saved Bluetooth pairings. If the whole right half is unresponsive after flashing, investigate power and split pairing before individual switches.

## Layout

The base layer follows the original Caldera keymap. Each half has 30 switches:

```text
 Left half                                  Right half
 GRAVE  1    2    3    4    5                 6    7    8    9    0    BACKSPACE
 TAB    Q    W    E    R    T                 Y    U    I    O    P    BACKSLASH
 CAPS   A    S    D    F    G                 H    J    K    L    ;    ENTER
 SHIFT  Z    X    C    V    B                 N    M    ,    .    /    '
 CTRL   ALT  GUI  SPC  ALT  GUI               SPC  LOW  LEFT DOWN UP   RIGHT
```

GUI is Command on macOS. LOW is the second key from the left on the right half's bottom row. It changes layers only while held; pressing it alone does not type a character. Modifier keys also need a second key to produce a visible effect.

On the lower layer, the left top-row keys in positions 2–6 select Bluetooth profiles 1–5; the right top-right key clears the selected profile. Parentheses are on left D/F positions; brackets, minus and equals are on right J/K/L/semicolon positions. Other positions fall through to the base layer.

## Optional debug firmware

The artifact also includes `caldera_left-debug.uf2` and `caldera_right-debug.uf2`. These have the same keymap and add USB serial logging. Flash the matching debug file only when diagnosing that half, then connect it by USB and open its serial port. The right half can provide serial logs even though it does not send keyboard input over USB.

On macOS, list available ports with:

```sh
ls /dev/cu.usbmodem*
```

Use a serial monitor with the actual port. Compare one working key with one failing key and look for `Row:`, `col:`, `position:`, and `pressed: true/false`.

- Correct press and release events mean the switch signal reaches that controller. Investigate key mapping or the split/host connection next.
- `Not found in transform` indicates a software coordinate-mapping problem.
- No event for a failing key, while a working key logs normally, can mean a switch, socket, diode, solder joint or wiring problem, or incorrect scan-pin configuration. It does not alone prove hardware failure.

Use normal firmware again after collecting logs to avoid the extra power use of logging.

If switches are hot-swappable, disconnect USB and battery power before swapping a failing switch with a working one. A failure that follows the switch identifies a switch problem; one that stays in the same position needs further investigation.

## Configuration details

- `build.yaml` selects Caldera left/right shields and produces normal plus debug firmware.
- `config/boards/shields/caldera/` contains the shield definitions.
- `config/caldera.keymap` has 60 bindings on every layer.
- The matrix transform has five rows and twelve columns, matching the original PCBs. There is no sixth scan row.
- The left half uses columns 3, 4, 5, 6, 7, 2; the right uses 2, 7, 6, 5, 4, 3 and a column offset of 6. Both use row pins 21, 20, 19, 18, 15 and col2row diodes. These are Pro Micro connector labels.
- Both the manifest and build workflow remain on ZMK v0.3.

References: [nice!nano flashing](https://nicekeyboards.com/docs/nice-nano/getting-started/), [ZMK split keyboards](https://zmk.dev/docs/features/split-keyboards), [USB logging](https://zmk.dev/docs/development/usb-logging), [hardware troubleshooting](https://zmk.dev/docs/troubleshooting/hardware-issues).
