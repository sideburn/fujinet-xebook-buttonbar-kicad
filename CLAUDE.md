# CLAUDE.md

This file provides guidance to Claude Code when working in this repository.

## What this repository is

A small KiCad panel-mount board carrying five tactile buttons for the
[FN32ROV-XEBook-KiCad](../FN32ROV-XEBook-KiCad) main board: two volume buttons and the
three FujiNet control buttons (swap, Bluetooth, reset). See `README.md` for the full
description.

This is an **older board** — it predates the LED/SD daughter-board pattern (and the
two-tier verification workflow) used elsewhere in this board family. It has already been
fabricated once with a 3-pin connector for the FujiNet buttons; that connector has since
been upgraded to 4 pins (see "Connector fix" below) to add a proper ground return, ahead
of the next fab run.

Status: schematic and PCB fixed, routed, and verified (`kicad-cli pcb drc`, full +
`--schematic-parity`: 0 unconnected, 0 shorts). Fab package (gerbers, drill, JLCPCB
BOM/CPL) generated in `Fab/`. Not yet re-fabricated with the fix.

A logo was imported to `B.SilkS` on 2026-08-02 as 68 `gr_poly` outlines (source artwork in
`Graphics/XEBOOKButtonBarLogo.ai`/`.svg`; the two board screenshots moved from `images/` to
`Graphics/` at the same time). `Fab/` was regenerated for it and, timestamps aside,
**`B_Silkscreen.gbo` is the only output that changed** — copper, mask, paste, drill and
Edge.Cuts are byte-identical, and footprints/tracks/vias/outline are untouched. Export
without `--subtract-soldermask` (`%LPC`=0), same as the SD board.

## Interface contract with the main board

Two connectors carry all five buttons off-board via cables, soldered directly to
through-hole pins (no mating connector — wires are soldered straight onto the pads):

- **J1 ("vol")** — 3-pin `PinHeader_1x03_P2.00mm_Vertical`. Pin 1 = GND, pin 2 = volume
  down (SW1, silkscreened "-"), pin 3 = volume up (SW2, silkscreened "+"). Self-contained;
  not part of the main board's documented interface contracts.
- **J2 ("fuji")** — 4-pin `PinHeader_1x04_P2.00mm_Vertical`, mates 1:1 with J_BTN1 on the
  main board (straight, non-crossed wiring):

  | Pin | Net | Switch on this board |
  |---|---|---|
  | 1 | GND | shared `GNDREF` return (previously only reachable via J1's cable) |
  | 2 | BTN_SWAP | SW3, silkscreened "swap" |
  | 3 | BTN_BT | SW4, silkscreened "bt" |
  | 4 | BTN_RST | SW6, silkscreened "rst" |

  See the main board's `CLAUDE.md`, "External button bar interface (J_BTN1)", for the
  GPIO/pull-up side of this contract.

All switches are active-low: each button ties its signal pin to `GNDREF` when pressed:
the main board pulls each signal high internally.

## Connector fix (J2: 3-pin → 4-pin)

J2 originally carried only the 3 button signals with no ground pin — those buttons'
return path only worked because J1's cable happened to also carry a GND wire on the same
local `GNDREF` net. This has been fixed: J2 is now a 4-pin `PinHeader_1x04_P2.00mm_Vertical`
(**not** a JST-PH connector — cable wires are soldered directly to the through-hole pins,
so the small/cheap pin header is preferred over a mating connector here).

Two things to know if touching this again:

- **Button-to-pin identity isn't obvious from pin order alone.** The schematic's pin
  numbering (`Net-(J2-Pin_N)`) doesn't reveal which physical button is on which pin — the
  PCB footprints' `Reference` text was hand-overridden to functional names ("swap", "bt",
  "rst") independent of the schematic reference designator (SW3/SW4/SW6). Always confirm
  button identity via those PCB silkscreen labels (or by tracing `path` UUIDs between the
  `.kicad_sch` and `.kicad_pcb`), not by assuming pin-number order carries any meaning.
- **Adding the 4th pin shifted where GND sits, not just appended it.** To make pin 1 =
  GND (matching J_BTN1's pin 1), the three existing signal wires each moved down one slot
  and GND took the vacated top slot — this is a s-expression connectivity edit, not a
  visual rearrangement, and it's easy to get the direction backwards (see the git history
  for the two false starts this took: one where pin geometry math was wrong, and one
  where the button-to-net mapping was assumed instead of verified against the PCB).

## Working conventions

Same as the main board:

- The user does manual placement/routing/symbol cleanup themselves in the KiCad GUI;
  prioritize correctness over polish when editing files directly.
- Verification is two-tier and both are required before calling the board correct:
  1. `kicad-cli sch erc` on the schematic.
  2. `kicad-cli pcb drc --schematic-parity` — PCB vs. schematic (references, footprints,
     net connectivity must match).
- This board predates the newer boards' clean parity baseline: `kicad-cli pcb drc
  --schematic-parity` reports 14 pre-existing `missing_footprint`/`extra_footprint`
  warnings because every PCB footprint's `Reference` text was overridden to a functional
  label ("vol", "swap", "bt", "rst", "+", "-") instead of the schematic reference
  designator (J1/J2/SW1-SW4/SW6). These are cosmetic (matching is still correct via
  `path` UUID) and pre-date this fix — confirmed identical against the pre-fix backup
  before treating any new parity output as a regression. Don't try to silence them by
  renaming the PCB reference text back to match the schematic; that would undo an
  intentional, presumably user-made labeling choice.
- No original DipTrace netlist exists for this board (unlike the main board) — its ground
  truth is the interface contract table above.

## Gotcha: `"-"`/`"+"` reference designators break Specctra DSN export

SW1 and SW2's PCB `Reference` text is literally `"-"` and `"+"` (see functional-label
note above). This is fine for KiCad itself (confirmed via `kicad-cli pcb export
ipcd356` — the underlying pad-to-net data is correct), but it breaks autorouting via
Specctra DSN (e.g. the Freerouting plugin): DSN pin identifiers are formatted as
`refdes-pinnumber`, and the file's own coordinate fields use bare `+`/`-` as sign
prefixes throughout. A refdes that IS `-` or `+` produces a token indistinguishable from
part of a signed number, so the exporter/importer silently drops that pin's ratsnest
instead of erroring — Freerouting's routed-board preview won't even show an airwire for
it, and will report the board fully routed while that connection stays open. This only
hits SW1/SW2; the other functional labels ("swap", "bt", "rst", "vol", "fuji") are plain
words and export fine.

The user has chosen to keep `-`/`+` as-is rather than rename them — if autorouting via
DSN/Freerouting, manually verify (or hand-route) SW1/SW2's connections afterward, and
always confirm the final result with `kicad-cli pcb drc` rather than trusting the
router's own "fully routed" report.
