# XEBook Button Bar

A panel-mount board carrying five tactile buttons for the
[FN32ROV-XEBook-KiCad](../FN32ROV-XEBook-KiCad) main board, which mounts the FujiNet
FN32ROV WiFi/SIO adapter permanently inside an XEBook laptop build.

On the main board, the buttons were moved off-PCB onto cable pigtails so they can be
panel-mounted separately (see that project's `CLAUDE.md`, "External button bar interface
(J_BTN1)"). This board is that panel-mounted button bar.

## What's on it

- SW1 ("-") / SW2 ("+") — volume down / volume up, wired to **J1**.
- SW3 ("swap") / SW4 ("bt") / SW6 ("rst") — the three FujiNet control buttons (swap
  device, Bluetooth, reset), wired to **J2**.
- All five are low-profile SMD tactile switches (E-Switch TL3342), active-low (each pulls
  its signal to GND when pressed).
- **J1** — 3-pin `PinHeader_1x03_P2.00mm_Vertical`, through-hole. Pin 1 = GND, pin 2 =
  volume down, pin 3 = volume up.
- **J2** — 4-pin `PinHeader_1x04_P2.00mm_Vertical`, through-hole. Pin 1 = GND, pin 2 =
  swap, pin 3 = Bluetooth, pin 4 = reset — matches J_BTN1 on the main board pin-for-pin.
  Cable wires are soldered directly to both connectors' pins rather than using a mating
  connector.

## Revision note: J2 upgraded from 3 pins to 4

The original J2 carried only the three FujiNet button signals with no dedicated ground
pin — those buttons' return path only worked because it shared a ground reference with
J1 through that connector's cable. J2 is now a proper 4-pin header with its own GND pin,
matching the main board's J_BTN1 connector so the two boards' cabling maps 1:1. See this
repo's `CLAUDE.md` for the technical details of the fix.

**This fix has not yet been re-fabricated.** The schematic and PCB footprint/net changes
are verified (ERC and DRC clean, no shorts), but the copper routing from J2's new pin
layout out to the existing traces needs to be finished by hand in the KiCad GUI before
ordering the next batch of boards.

## Status

Previously fabricated with the 3-pin J2 (see `production/`). The 4-pin J2 fix described
above is designed and verified but not yet routed or re-fabricated.

## Tooling

Built with KiCad 9. `kicad-cli sch erc` / `kicad-cli pcb drc --schematic-parity` are used
to verify the design.

## License

[CERN Open Hardware Licence Version 2 - Strongly Reciprocal](LICENSE)
(CERN-OHL-S-2.0), matching the main board.
