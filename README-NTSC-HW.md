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

### Moving the picture down for more speed (`shift`)

```
make variant=ntsc-hw shift=6
```

`shift` moves the picture down by 0 to 7 scanlines (default 0). The screen is
then re-enabled that many lines later, so the NMI handler gets about 114 more
CPU cycles per line to send data to the PPU in every frame. See
[Picture shift](#picture-shift-shift) below for the trade-off.

Since the [early VBlank](#early-vblank-with-a-dmc-timer) was added, `shift`
makes little difference to speed: both use the same spare scanlines, and the
early VBlank already uses all of them without moving the picture. So the
default of 0 is the best choice for most people.

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

* **Column window** (`RowSpanNTSC` and `WindowNTSC` in bank 3, `WinRowNTSC` in
  bank 7): while scanning a space view frame, the main loop also finds the
  leftmost and rightmost non-empty columns in rows 2 to 19. Combined with the
  same span from the frame last sent to that bitplane (so old pixels get
  blanked), this gives a window of columns that can have changed. Each row that
  needs sending then only sends the entries inside the window, by jumping into
  the unrolled sends at `snam7` part-way through, and `WinFixNTSC` moves on to
  the start of the next row. Rows 0 and 1 (the view title) and any row after
  the PPU contents are unknown (a full-buffer send, or a new view) are still
  sent in full, as columns 0 and 1 hold the box edges. Each row is charged
  `WIN_ROW_K` plus 11 cycles per entry.

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
  to the cycle count before the burn loop is `NMI_MARGIN_NTSC_HW` (200), enough
  to cover the most a step can overdraw by (running out part-way through
  checking a row for the column window).

### Frame pacing

Without pacing, combat ran at about 117% of PAL speed, with each frame staying
on screen for an uneven 3 or 4 VBlanks. To match the feel of the original, the
space view is now paced (`FRAME_VBLANKS_NTSC_HW`, in `DrawEdgesScanNTSC`):

* Before handing a new frame to the NMI handler, the main loop waits until at
  least 4 VBlanks have passed since the last handover. The time is taken from
  `nmiCounter` and stored in `lastHandoverNMI`, a previously unused WRAM byte.
* The wait loop keeps polling `SETUP_PPU_FOR_ICON_BAR`, so the icon bar split
  isn't affected.
* 4 NTSC VBlanks is 66.7 ms, close to the PAL release's typical frame time in
  combat. When the ships are small, the game therefore runs at the original
  speed, but the frames are evenly spaced, where PAL alternates between 3 and 4
  PAL VBlanks (60 ms and 80 ms).
* The handover is also held back until at least `FLIP_GAP_NTSC_HW` (2) VBlanks
  have passed since the last bitplane flip. The flip happens in the NMI after a
  handover, but if the NMI is still busy sending the previous frame, it comes a
  VBlank late. Pacing from the handover alone then showed that frame for 5
  VBlanks and the next one for only 3. `FlipCheckNTSC` runs at the start of
  every NMI (in a fixed number of cycles, before the budget is set) and records
  the time of each flip in `lastFlipNMI`, so a late flip now delays the next
  handover too, and the frames stay 4 VBlanks apart.
* Scenes that already take longer than 4 VBlanks are unaffected. Only the space
  view is paced; other screens run as before.
* Pacing can't help when a ship fills a large part of the view (the rotating
  ship on the title screen, or a ship or station close up). Those frames need
  far more data than an NTSC VBlank can carry, so they still take longer than
  on PAL. See [Results](#results).

### Picture shift (`shift`)

A large ship needs about 90 new patterns and 15 to 18 nametable rows per frame,
over 1,000 bytes. Sending a byte costs at least 11 CPU cycles, and an NTSC VBlank
leaves only about 2,000 cycles for data once the sprite DMA and other fixed work
are done. A PAL VBlank leaves more than twice that. So when the ship is big, the
NTSC version is limited by how fast it can send data, not by the game code.

The only way to send more per frame is to keep the screen off for longer. The
picture starts on scanline 8 and ends on scanline 232, so there are a few unused
lines below it. `shift=N` moves the picture down by N lines and re-enables the
screen N lines later:

* The sprites are moved down by the same amount, using the `YPAL` margin that
  the PAL release uses for its own taller picture (`YPAL = _NTSC_HW_SHIFT`).
* `NMI_CYCLES_NTSC_HW` grows by 341 / 3 cycles per line, so rendering restarts
  on line 7 + N.
* The trade-off: the bottom N lines of the dashboard move into the area that
  NTSC CRT TVs usually hide (about the bottom 8 lines). The bottom row of the
  dashboard (the `LT` and `AL` indicators) is already close to that edge with
  `shift=0`. Emulators and upscalers that show all 240 lines show the whole
  picture, just a few lines lower.

On its own, `shift=6` makes the title screen's big ship about 45% faster. The
[early VBlank](#early-vblank-with-a-dmc-timer) below gets the same gain without
moving the picture, and the two don't add up, as they use the same spare lines.

### Early VBlank with a DMC timer

The picture ends on scanline 232, but the NMI only arrives at scanline 241, so
eight lines at the bottom of every frame go to waste. The MMC1 has no scanline
counter, so the NTSC hardware variant uses the APU's DMC channel as a timer to
start the VBlank routine on scanline 233 instead (`IRQHandlerNTSC`). This gives
about 880 more cycles per frame for sending data to the PPU.

How it works:

* The DMC plays a silent sample (`dmcSilence`, 49 zero bytes) at its fastest
  rate. It fetches one byte every 432 CPU cycles, on a fixed grid, and raises an
  interrupt when it fetches the last byte.
* **Sync**: a 49-byte sample ends just before the sprite 0 hit at the top of the
  icon bar. The interrupt handler polls `PPU_STATUS` until the hit, which tells
  it exactly where the DMC grid is relative to the screen. It then does the
  icon bar split (on scanline 162, a few lines later than the main loop's
  polling usually manages) and works out the rest of the frame's timing.
* **Hops**: a 17-byte sample, followed by two or three one-byte samples
  (432 cycles each), takes the timer to just before scanline 233.
* **Final**: the last interrupt starts the next frame's 49-byte sample, waits
  for the remaining 0 to 431 cycles, turns off rendering on scanline 233 (at
  around dot 150, where switching it off doesn't upset the sprite memory), and
  runs the VBlank routine with a larger budget (`NMI_CYCLES_EARLY`). The real
  NMI arrives part-way through and is ignored (`NMIEntryNTSC`). The screen is
  re-enabled on scanline 7 as usual.

Safety checks:

* The chain only runs in the space view. Other screens are unchanged.
* When the chain starts (from a normal NMI, `StartChainNTSC`), we don't yet know
  where the DMC grid is. So the first sync interrupt is placed up to seven lines
  before the sprite 0 hit, and the early VBlank only starts after five more
  syncs agree with each other (`IRQ_DRY_FRAMES`).
* Each sync must land within four poll iterations of where the previous one
  predicted (the grid drifts 27.33 cycles a frame relative to the screen,
  `irqDrift`). The sprite 0 hit moves for a few frames while the screen is
  being set up, and this check catches it. The chain then stops, and the
  next NMI is handled normally and restarts it.
* No early VBlank is started if NMIs are disabled, so screen setup code is
  never interrupted.
* The game used to run with interrupts disabled, as it didn't use them, so
  `DrawEdgesScanNTSC` now enables them. If they were disabled (after a game
  restart), the chain is stopped first, so no stale interrupt arrives at the
  wrong time.
* A DMC sample fetch during a controller read can make the controller skip a
  button, so `ReadControllersNTSC` reads each controller until it gets the same
  result twice in a row.

Making room: the line images (`lineImage`, 232 bytes) and `SendInventoryToPPU`
are only used by bank 3, so they now live there, and their space in bank 7 holds the DMC sample and the
interrupt handler. The rest of the new code goes into the spare bytes in the
`DrawBoxEdges`, `FillMemory` and `SendBarNamesNTSC` blocks and in padding
elsewhere. The NMI code itself doesn't move.

### Sprite DMA at the end of the NMI

The PPU's sprite memory (OAM) is dynamic RAM that is only refreshed while the
screen is being drawn. On an NTSC PPU it is only guaranteed to hold its contents
for about a normal VBlank (around 1.3 ms), and this variant keeps the screen off
for longer than that (from line 241 to line 7, or later with `shift`). So the
sprite DMA now happens at the very end of the NMI, just before the screen is
re-enabled (`RestartScreenNTSC`), instead of at the start. The DMA takes the
same time either way, so this costs nothing.

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
* `DV41` and `DVID4` (142 bytes) are only called from bank 1, so they now live
  there, and their space in bank 7 (`dvidNTSC`) holds `FlipCheckNTSC`,
  `TailEntryNTSC`, `BarFixNTSC`, `WinFailNTSC` and `WinBailNTSC`.

### Tuning constants (`elite-source-common.asm`)

| Constant | Value | Meaning |
|---|---|---|
| `NMI_CYCLES_NTSC_HW` | 1969 + 113.67 &times; shift &minus; shift | NMI cycle budget, which sets where the screen is re-enabled |
| `NMI_CYCLES_EARLY` | `NMI_CYCLES_NTSC_HW` + 889 &minus; 113.67 &times; shift | Cycle budget when the VBlank routine starts early on line 233 + shift |
| `NMI_MARGIN_NTSC_HW` | 200 | Cycles added before the burn loop, so an overdrawn count still re-enables the screen on time |
| `IRQ_SYNC_K` | 1095 | Cycles from the end of the 17-byte sample to the early VBlank, less 11 per poll iteration |
| `IRQ_START_HOPS` | 14 | One-byte hops after the 33-byte sample when starting the chain |
| `IRQ_DRY_FRAMES` | 5 | Agreeing syncs needed before the first early VBlank |
| `IRQ_SKIP_DELAY` | 11 | Delay in `StartChainNTSC` that keeps its timing the same when the chain is already running |
| `IRQ_SPLIT_DELAY` | 70 | Delay between the sprite 0 hit and the icon bar split |
| `FRAME_VBLANKS_NTSC_HW` | 4 | Minimum VBlanks per space view frame (game pacing) |
| `FLIP_GAP_NTSC_HW` | 2 | Minimum VBlanks between the last bitplane flip and the next handover |
| `NAME_FLIP_NTSC_HW` | 12 | Nametable batches left before swapping bitplanes |
| `NAME8_CYCLES` | 146 | Cost of 8 nametable entries in `SendNamesTail` |
| `PATT1_CYCLES` | 160 | Cost of 1 pattern in `SendPattsTail` |
| `ROW_SKIP_CYCLES` | 146 | Cost of skipping an empty row |
| `ROW_CHECK_CYCLES` | 114 | Extra cost of checking a row that gets sent in full |
| `ROW_FAIL_CYCLES` | 117 | Cost of a row check when there's no time left to skip |
| `ROW_FULL_CYCLES` | 47 | Row check cost during a full-buffer send |
| `WIN_ROW_K` | 200 | Cost of a row sent through the column window, plus 11 per entry |
| `WIN_FIX_CYCLES` | 55 | Cost of moving on to the next row after a window (`WinFixNTSC`) |
| `WIN_FAIL_CYCLES` | 156 | Cost of the checks when a row doesn't fit through the window |
| `WIN_BAIL_MIN` | 64 | Stop sending if fewer cycles than this are left when a window row starts |
| `WIN_BAIL_CYCLES` | 115 | Cost of the checks when we stop there |
| `TAIL_ENTRY_CYCLES` | 50 | Cost of resuming a part-sent row before `SendNamesTail` |
| `BAR_FIX_CYCLES` | 10 | Cycles that `SendBarNamesToPPU` overcharges when no pattern batch fits |

If you change `NMI_CYCLES_NTSC_HW`, the screen must still be re-enabled
between line 6 + shift dot 257 and line 7 + shift dot 255. Any earlier or later,
and the picture jumps by a line for that frame.

`NamesRowCheck` stops straight away if the cycle count is already negative, and
`WinRowNTSC` stops if fewer than `WIN_BAIL_MIN` cycles are left. Otherwise a row
check that can't succeed could overdraw the margin in `SendScreenToPPU`, which
makes the screen come back early. A few original constants were also adjusted
for this variant, using the same number of bytes: `snam1` adds back 19 cycles
instead of 58, as that path takes longer than the original count allows for.

## Results

### How these were measured

The PAL release and the `ntsc-hw` build are run side by side on the Mesen
emulator core (cycle-accurate PPU timing), headless, from the same START press.
For every VBlank the test records which bitplane is on screen, so it can count
how often the 3D view gets a new frame, in real time. It also renders a
real-time side-by-side video, like a MesenCE screen recording. Two scenarios are
used:

* **Title**: the rotating ships on the title screen, 5 to 55 seconds after
  START. This is the worst case, because the ships are drawn large.
* **Combat**: the combat demo, 25 to 85 seconds after START.

"3D updates/s" is how many new frames the 3D view shows per second (higher is
smoother). "Slow frames" is the 90th percentile of how long a frame stays on
screen.

| | PAL release | Before (shift=0) | shift=6 | Early VBlank |
|---|---|---|---|---|
| Title, 3D updates/s | 14.9 | 6.5 | 9.5 | 9.6 |
| Title, median frame time | 60 ms | 183 ms | 100 ms | 83 ms |
| Title, slow frames (p90) | 100 ms | 216 ms | 150 ms | 150 ms |
| Combat, 3D updates/s | 12.4 | 10.3 | 11.1 | 11.2 |
| Combat, median frame time | 60 ms | 67 ms | 67 ms | 67 ms |
| Combat, slow frames (p90) | 120 ms | 200 ms | 150 ms | 150 ms |

"Before" is the build without the early VBlank or a picture shift. When the
ships are small, all the NTSC builds run at the paced 4 VBlanks (67 ms) per
frame, which matches PAL. When a ship is big, PAL can still send more than twice
as much data per second, so the NTSC builds fall behind. The early VBlank closes
about half of that gap without moving the picture.

The column window and flip pacing were measured over a longer window (5 to 65
seconds after START), and over four combat runs, as the combat demo plays out
differently from run to run:

| | PAL release | Early VBlank | **Window + flip pacing** |
|---|---|---|---|
| Title, 3D updates/s | 15.1 | 10.3 | **10.3** |
| Title, median frame time | 60 ms | 83 ms | **67 ms** |
| Combat (4 runs), 3D updates/s | 10.9 to 11.3 | 10.2 to 11.1 (mean 10.5) | **10.0 to 10.4 (mean 10.2)** |
| Combat, frames shown for 4 VBlanks | | 59% | **63%** |
| Combat, frames shown for 3 VBlanks or fewer | | 2.3% | **0.6%** |
| Combat, a 5-VBlank frame followed by a 3-VBlank one | | 10 | **0** |

So the window doesn't make the game faster overall. It lets more rows fit into
each VBlank, but most of that is used up by the larger margin it needs, so that
the screen is always re-enabled on time. The flip pacing makes the frame times
steadier. Together they cost about 3% in combat, which is within the run-to-run
variation of the combat demo.

These checks were run across the title screen, attract mode, the combat demo,
every docked screen, launching and flight, with the emulator's OAM corruption
emulation turned on:

* **Screen re-enable point**: always inside the safe window, both after an
  early VBlank (from 46 dots before the start of line 7 to dot 224 of line 7)
  and after a normal NMI (from 48 dots before to dot 224). With `shift=6`, a few normal NMIs on the docked screens
  re-enable it up to 20 dots too early, so that build can jump by a line now
  and then; the default build doesn't.
* **Early VBlank start**: always on line 233 between dots 115 and 203.
* **No PPU writes** to `$2005`, `$2006` or `$2007` while the screen is rendering.
* **Nametable correctness**: every frame the NMI finishes sending was compared
  byte for byte with the buffer the game handed over. There were no mismatches
  across about 1,500 checked frames per test for each build, including this one.
* **Icon bar split**: on line 162 when the early VBlank is running, and on lines
  158 to 166 otherwise, as before.

These were all emulator tests. Please report anything you see on real hardware.

## Known limitations

* The intro scroll text still says "NTSC EMULATION", as in Ian Bell's NTSC
  variant.
* The noise channel isn't retuned, because its period table is built into the
  APU and differs between the NTSC and PAL consoles.
* Sound effect pitches aren't rescaled, only the music note table.
* Game speed still varies with scene complexity, as in all versions of NES
  Elite. When a ship fills a large part of the view, the NTSC hardware can't
  send the data as fast as a PAL console, so those scenes are still slower than
  PAL.
* About one frame in 160 in combat is still shown for 3 VBlanks or fewer, when
  something other than the usual handover causes the flip.
* After entering the space view, it takes about 6 to 30 frames before the early
  VBlank starts. It only uses the DMC channel in the space view, and the
  sample it plays is silent.
* The DMC timer depends on the console's DMC timing, which is well documented
  and matches the emulator, but this has only been tested in an emulator. If
  it misbehaves on your console, the previous ROM without the early VBlank is
  the fallback.
