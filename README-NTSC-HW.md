# NES Elite for real NTSC hardware

This is a modified version of Mark Moxon's annotated NES Elite source
([markmoxon/elite-source-code-nes](https://github.com/markmoxon/elite-source-code-nes))
that adds a new build variant, `ntsc-hw`, which runs on a stock NTSC NES.

The existing variants are unchanged and still build byte-for-byte identical to
the reference binaries:

| Variant | Build option | Runs on |
|---|---|---|
| PAL (official Imagineer release) | `variant=pal` (default) | PAL consoles |
| NTSC (Ian Bell's "NTSC emulation") | `variant=ntsc` | Some emulators only, not real NTSC hardware |
| **NTSC hardware (new)** | `variant=ntsc-hw` | **Real NTSC consoles** |

All the new code is wrapped in `IF _NTSC_HW` conditionals, so searching the
source for `_NTSC_HW` shows every change.

## Building

### Requirements

* [BeebAsm](https://github.com/stardot/beebasm) (tested with version 1.11). On
  Mac and Linux, build it by running `make code` in its `src` folder; on
  Windows, download `beebasm.exe`
* Python 3 (only used to verify the PAL and NTSC builds against the reference
  binaries)
* GNU Make on Mac or Linux; on Windows the repository includes `make.bat` and
  `2-build-files/make.exe`

### Mac and Linux

```
make variant=ntsc-hw
```

If `beebasm` isn't on your path, pass its location:

```
make BEEBASM=/path/to/beebasm variant=ntsc-hw
```

### Windows

Edit the paths to BeebAsm and Python at the top of `make.bat`, then run:

```
make.bat variant=ntsc-hw
```

### Output

The ROM image is written to `5-compiled-rom-images/ELITE-ntsc-hw.NES`. It is a
standard iNES file (mapper 1, MMC1, 128K PRG ROM, CHR RAM, battery-backed WRAM),
so it runs on a flash cart or an emulator set to NTSC.

As `ntsc-hw` is not an original release there are no reference binaries, so the
build skips checksum verification. To check that the original variants are
still intact:

```
make variant=pal
make variant=ntsc
```

Both should report `Yes` for all ten files.

The other build options (`commander=max`, `match=no`) work with `ntsc-hw` as
well.

## Why the NTSC variant doesn't work on real hardware

Each frame, the NMI handler blanks the screen, sends as much data to the PPU as
a cycle budget allows, and then re-enables the screen at a fixed point. Even the
PAL release overruns its 70-line VBlank and re-enables the screen on scanline 7,
so the picture starts a few lines down (which is what the `YPAL` margin is for).

The NTSC variant budgets 6797 cycles, but a real NTSC VBlank is only 20 lines
(about 2270 CPU cycles). On real hardware it re-enables the screen around
scanline 50, which pushes the picture down the screen, breaks the icon bar
split and cuts off the bottom of the dashboard.

## What changes in the `ntsc-hw` variant

The variant is based on the NTSC variant (`_NTSC` is also true), so it keeps
that variant's screen layout and text.

### Fitting into the NTSC VBlank

* **NMI budget** (`NMI_CYCLES_NTSC_HW`, bank 7 `NMI`): the screen is re-enabled
  near the start of scanline 7, as in the PAL release. This is inside the
  overscan area that NTSC TVs hide.

* **Scroll position** (`SetScrollNTSC`): the pre-render line goes by with
  rendering off, so the PPU never reloads its vertical scroll. Before
  re-enabling the screen, the full PPU address is loaded with fine y-scroll 6
  using the `$2006/$2005/$2005/$2006` sequence. Nametable row r then appears on
  scanline r + 1, which is the layout the NTSC coordinates (`YPAL` = 0) are
  designed for, so the whole dashboard is visible on an NTSC TV.

* **Icon bar hang** (`SendBarNamesNTSC`, `SendBarPattsNTSC`): the original code
  has to send the icon bar's nametable entries and its first batch of patterns
  in one VBlank. That never fits in an NTSC VBlank, so the game would hang the
  first time the icon bar changed. The update is now split over two VBlanks.

* **Half-drawn frames** (`NAME_FLIP_NTSC_HW`): the NMI swaps the visible
  bitplane once the rest of the frame should fit in the current VBlank. The
  threshold is lowered from 48 batches to 12 so it holds on NTSC.

* **Icon bar split timing** (`DrawEdgesScanNTSC`): the long-running row scan
  described below polls `SETUP_PPU_FOR_ICON_BAR` once per row, so the switch to
  the icon bar's nametable and pattern table still happens on time. Without
  this, the top of the dashboard flickered.

### 50 Hz to 60 Hz

* **Music and sound speed** (`MakeSoundsNTSC`): every sixth call to the sound
  routines is skipped, so music and effects run at 50 updates per second as on
  PAL. It uses `unusedVariable` as its counter.
* **Music pitch** (`noteFrequency`, bank 6): the note periods are scaled by the
  CPU clock ratio (1.789773 / 1.662607 = 1.0765). Without this the music would
  be about 1.3 semitones sharp.
* **Timer** (`UpdateNMITimer`, bank 0 combat demo): the timer counts 60 VBlanks
  per second, so the combat demo time is in real seconds.

### Getting the speed back

NES Elite only moves the game on once a frame has been sent to the PPU, and an
NTSC VBlank gives the NMI handler about a third of the PPU time it gets on PAL.
Fitting into the VBlank alone made the game run at about 70% of PAL speed in
combat. These changes recover it:

* **Row skipping** (`DrawEdgesScanNTSC` in bank 3, `NamesRowCheck` in bank 7):
  the space view sends nametable rows 2 to 19 (576 bytes) every frame, even
  though most rows are usually empty. When a frame is handed to the NMI, its
  buffer is scanned. A row is only sent if it contains something now, or
  contained something in the frame last sent to that bitplane, so that it gets
  blanked. The masks are kept in previously unused WRAM bytes. They are reset by
  `InvalidateRowMasks` (called from `ResetScreen` and `SendViewToPPU`) and by
  any full-buffer send for a new view. Other views always send every row.

* **Finer batches** (`SendNamesTail`, `SendPattsTail`): when a full batch of 32
  nametable entries or 3 patterns no longer fits, the NMI keeps going 8 entries
  or 1 pattern at a time instead of wasting the rest of the VBlank. 32-entry
  batches must start on a row boundary, because they only check for a page
  crossing at the end of a batch. A resume that isn't on a row boundary
  therefore goes through the tail until it is.

* **Accurate cycle accounting**: the cycle costs charged for the paths used in
  this variant were measured on a cycle-accurate emulator and corrected,
  including several original constants (`sbuf7` 283, `snam6` 359). Failed
  attempts are charged what they really cost. `BurnFineNTSC` burns off the 0 to
  31 cycles that the burn loop in `ClearBuffers` overshoots by. The amount added
  to the cycle count before the burn loop is 150, enough to cover the most a
  step can overdraw by.

### Making room in the fixed bank

The NMI code has to live in bank 7, which is full. Its cycle counts also depend
on page-crossing timings, so none of the existing NMI code is moved:

* `DrawBoxEdges` (80 unrolled stores) becomes a loop. It is only called once per
  frame.
* `FillMemory` (256 unrolled stores) becomes a loop of 16. `ClearMemory`'s
  computed `JMP (clearBlockSize)` into the middle of the unrolled code is
  replaced by `ClearSmallNTSC`, which charges its extra time to the cycle count.
* The new routines go into the freed space and into blocks the source marks as
  unused, padded back to their original sizes and checked with `ASSERT`.
* The main-loop row scanner and `InvalidateRowMasks` live in bank 3, which has
  free space. The scanner is called through the `DrawEdgesScan_b3` trampoline.

### Tuning constants (`elite-source-common.asm`)

| Constant | Value | Meaning |
|---|---|---|
| `NMI_CYCLES_NTSC_HW` | 2177 | NMI cycle budget, which sets where the screen is re-enabled |
| `NAME_FLIP_NTSC_HW` | 12 | Nametable batches left before swapping bitplanes |
| `NAME8_CYCLES` | 146 | Cost of 8 nametable entries in `SendNamesTail` |
| `PATT1_CYCLES` | 160 | Cost of 1 pattern in `SendPattsTail` |
| `ROW_SKIP_CYCLES` | 131 | Cost of skipping an empty row |
| `ROW_CHECK_CYCLES` | 73 | Extra cost of checking a row that gets sent |
| `ROW_FAIL_CYCLES` | 108 | Cost of a row check when there's no time left to skip |
| `ROW_FULL_CYCLES` | 34 | Row check cost during a full-buffer send |

If you change `NMI_CYCLES_NTSC_HW`, the screen must still be re-enabled
between line 6 dot 257 and line 7 dot 255. Any earlier or later, and the picture
jumps by a line for that frame.

## Results

These were measured on the Mesen emulator core (cycle-accurate PPU timing), run
headless with scripted input.

| | PAL release | First `ntsc-hw` build | Current `ntsc-hw` |
|---|---|---|---|
| Combat demo, flight-loop iterations per second | 13.9 | 9.7 | **16.3** |
| Relative to PAL | 100% | 69% | **117%** |

Combat runs a little faster than the PAL original, because NTSC gives the main
loop more CPU time per second.

These checks were run across the title screen, attract mode, the combat demo,
every docked screen, launching and flight:

* **Screen re-enable point**: always between line 6 dot 334 and line 7 dot 192,
  inside the safe window.
* **No PPU writes** to `$2005`, `$2006` or `$2007` while the screen is rendering.
* **Nametable correctness**: every frame the NMI finishes sending was compared
  byte for byte with the buffer the game handed over. There were no mismatches
  across about 15,000 frames.
* **Icon bar split**: on line 165 or earlier, the same as without the speed
  changes (line 166 in a handful of frames).
* **Tearing**: no half-drawn frames in the 3D view.

These were all emulator tests. Please report anything you see on real hardware.

## Known limitations

* The intro scroll text still says "NTSC EMULATION", as in Ian Bell's NTSC
  variant.
* The noise channel isn't retuned, because its period table is built into the
  APU and differs between the NTSC and PAL consoles.
* Sound effect pitches aren't rescaled, only the music note table.
* Game speed still varies with scene complexity, as in all versions of NES
  Elite. Busy scenes can run slower than the figures above.
