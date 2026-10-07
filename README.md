# Fully documented source code for Elite on the NES

<details>
<summary>Links to my other software archaeology repositories</summary>
<hr>

**Elite sources:** [BBC Micro (cassette)](https://github.com/markmoxon/elite-source-code-bbc-micro-cassette) | [BBC Micro (disc)](https://github.com/markmoxon/elite-source-code-bbc-micro-disc) | [Elite Demonstration Disc](https://github.com/markmoxon/elite-demo-source-code-bbc-micro) | [Acorn Electron](https://github.com/markmoxon/elite-source-code-acorn-electron) | [6502 Second Processor](https://github.com/markmoxon/elite-source-code-6502-second-processor) | [Commodore 64](https://github.com/markmoxon/elite-source-code-commodore-64) | [Apple II](https://github.com/markmoxon/elite-source-code-apple-ii) | [BBC Master](https://github.com/markmoxon/elite-source-code-bbc-master) | [NES](https://github.com/markmoxon/elite-source-code-nes)

**Elite hacks:** [Elite-A](https://github.com/markmoxon/elite-a-source-code-bbc-micro) | [Two-player Elite](https://github.com/markmoxon/elite-two-player-6502-second-processor) | [Teletext Elite](https://github.com/markmoxon/teletext-elite) | [Elite 3D](https://github.com/markmoxon/elite-3d) | [Elite Universe Editor](https://github.com/markmoxon/elite-universe-editor) | [Flicker-free Commodore 64 Elite](https://github.com/markmoxon/c64-elite-flicker-free) | [Elite over Econet](https://github.com/markmoxon/elite-over-econet) | [!EliteNet](https://github.com/markmoxon/elite-over-econet-acorn-archimedes)

**Elite Compendium:** [BBC Master](https://github.com/markmoxon/elite-compendium-bbc-master) | [BBC Micro](https://github.com/markmoxon/elite-compendium-bbc-micro) | [BBC Micro B+](https://github.com/markmoxon/elite-compendium-bbc-micro-b-plus) | [Acorn Electron](https://github.com/markmoxon/elite-compendium-acorn-electron)

**Other sources:** [Aviator (BBC Micro)](https://github.com/markmoxon/aviator-source-code-bbc-micro) | [Revs (BBC Micro)](https://github.com/markmoxon/revs-source-code-bbc-micro) | [The Sentinel (BBC Micro)](https://github.com/markmoxon/the-sentinel-source-code-bbc-micro) | [Lander (Acorn Archimedes)](https://github.com/markmoxon/lander-source-code-acorn-archimedes)

**Other repositories:** [Scripts for generating bbcelite.com](https://github.com/markmoxon/bbcelite-scripts) | [Static content for bbcelite.com](https://github.com/markmoxon/bbcelite-websites) | [Elite source code library](https://github.com/markmoxon/elite-source-code-library) | [Elite Universe Editor library](https://github.com/markmoxon/elite-universe-editor-library) | [Elite over Econet file server menu](https://github.com/markmoxon/elite-over-econet-file-server-menu)

See [my profile](https://github.com/markmoxon) for more repositories to explore.
<hr>
</details>

![Screenshot of Elite on the NES](https://elite.bbcelite.com/images/github/nes-station.png)

This repository contains source code for Ian Bell and David Braben's classic game Elite on the Nintendo Entertainment System (NES), with every single line documented and (for the most part) explained. It has been reconstructed by hand from a disassembly of the original game binaries.

It is a companion to the [elite.bbcelite.com website](https://elite.bbcelite.com).

See the [introduction](#introduction) for more information, or jump straight into the [documented source code](1-source-files/main-sources).

## Contents

* [Introduction](#introduction)

* [Acknowledgements](#acknowledgements)

  * [A note on licences, copyright etc.](#user-content-a-note-on-licences-copyright-etc)

* [Browsing the source in an IDE](#browsing-the-source-in-an-ide)

* [Folder structure](#folder-structure)

* [Building NES Elite from the source](#building-neselite-from-the-source)

  * [Requirements](#requirements)
  * [Windows](#windows)
  * [Mac and Linux](#mac-and-linux)
  * [Build options](#build-options)
  * [Verifying the output](#verifying-the-output)
  * [Log files](#log-files)

* [Building different variants of NES Elite](#building-different-variants-of-nes-elite)

  * [Building the PAL variant](#building-the-pal-variant)
  * [Building the NTSC variant](#building-the-ntsc-variant)
  * [Building the NTSC hardware variant](#building-the-ntsc-hardware-variant)
  * [Differences between the variants](#differences-between-the-variants)

## Introduction

This repository contains source code for Elite on the NES, with every single line documented and (for the most part) explained.

You can build the fully functioning game from this source. [Two variants](#building-different-variants-of-nes-elite) are currently supported: the Imagineer PAL variant and the NTSC variant from Ian Bell's personal website.

This repository is a companion to the [elite.bbcelite.com website](https://elite.bbcelite.com), which contains all the code from this repository, but laid out in a much more human-friendly fashion. The links at the top of this page will take you to repositories for the other versions of Elite that are covered by this project.

* If you want to browse the source and read about how Elite works under the hood, you will probably find [the website](https://elite.bbcelite.com) a better place to start than this repository.

* If you would rather explore the source code in your favourite IDE, then the [annotated source](1-source-files/main-sources) is what you're looking for. It contains the exact same content as the website, so you won't be missing out (the website is generated from the source files, so they are guaranteed to be identical). You might also like to read the section on [browsing the source in an IDE](#browsing-the-source-in-an-ide) for some tips.

* If you want to build NES Elite from the source on a modern computer, to produce a working ROM image that can be loaded into a real NES or an emulator, then you want the section on [building NES Elite from the source](#building-nes-elite-from-the-source).

My hope is that this repository and the [accompanying website](https://elite.bbcelite.com) will be useful for those who want to learn more about Elite and what makes it tick. It is provided on an educational and non-profit basis, with the aim of helping people appreciate one of the most iconic games of the 8-bit era.

## Acknowledgements

NES Elite was written by Ian Bell and David Braben and is copyright &copy; D. Braben and I. Bell 1991/1992.

The code on this site has been reconstructed from a disassembly of the version released on [Ian Bell's personal website](http://www.elitehomepage.org/).

The commentary is copyright &copy; Mark Moxon. Any misunderstandings or mistakes in the documentation are entirely my fault.

Huge thanks are due to the original authors for not only creating such an important piece of my childhood, but also for releasing the source code for us to play with; to Paul Brink for his annotated disassembly; and to Kieran Connell for his [BeebAsm version](https://github.com/kieranhj/elite-beebasm), which I forked as the original basis for this project. You can find more information about this project in the [accompanying website's project page](https://elite.bbcelite.com/about_site/about_this_project.html).

The following archive from Ian Bell's personal website forms the basis for this project:

* [NES Elite, NTSC version](http://www.elitehomepage.org/archive/a/b7120500.zip)

### A note on licences, copyright etc.

This repository is _not_ provided with a licence, and there is intentionally no `LICENSE` file provided.

According to [GitHub's licensing documentation](https://docs.github.com/en/free-pro-team@latest/github/creating-cloning-and-archiving-repositories/licensing-a-repository), this means that "the default copyright laws apply, meaning that you retain all rights to your source code and no one may reproduce, distribute, or create derivative works from your work".

The reason for this is that my commentary is intertwined with the original source code for Elite, and the original source code is copyright. The whole site is therefore covered by default copyright law, to ensure that this copyright is respected.

Under GitHub's rules, you have the right to read and fork this repository... but that's it. No other use is permitted, I'm afraid.

My hope is that the educational and non-profit intentions of this repository will enable it to stay hosted and available, but the original copyright holders do have the right to ask for it to be taken down, in which case I will comply without hesitation. I do hope, though, that along with the various other disassemblies and commentaries of this source, it will remain viable.

## Browsing the source in an IDE

If you want to browse the source in an IDE, you might find the following useful.

* The main game's source code is split across eight different ROM banks, which you can find in the [main-sources](1-source-files/main-sources) folder. This is the motherlode and probably contains all the stuff you're interested in.

* It's probably worth skimming through the [notes on terminology and notations](https://elite.bbcelite.com/terminology/) on the accompanying website, as this explains a number of terms used in the commentary, without which it might be a bit tricky to follow at times (in particular, you should understand the terminology I use for multi-byte numbers).

* The accompanying website contains [a number of "deep dive" articles](https://elite.bbcelite.com/deep_dives/), each of which goes into an aspect of the game in detail. Routines that are explained further in these articles are tagged with the label `Deep dive:` and the relevant article name.

* There are loads of routines and variables in Elite - literally hundreds. You can find them in the source files by searching for the following: `Type: Subroutine`, `Type: Variable`, `Type: Workspace` and `Type: Macro`.

* If you know the name of a routine, you can find it by searching for `Name: <name>`, as in `Name: SCAN` (for the 3D scanner routine) or `Name: LL9` (for the ship-drawing routine).

* The entry point for the main game code is the `BEGIN` routine in [bank 7](1-source-files/main-sources/elite-source-bank-7.asm), which you can find by searching for `Name: BEGIN`. If you want to follow the program flow all the way from the title screen around the main game loop, then you can find a number of [deep dives on program flow](https://elite.bbcelite.com/deep_dives/) on the accompanying website.

* The source code is designed to be read at an 80-column width and with a monospaced font, just like in the good old days.

I hope you enjoy exploring the inner workings of NES Elite as much as I have.

## Folder structure

There are five main folders in this repository, which reflect the order of the build process.

* [1-source-files](1-source-files) contains all the different source files, such as the main assembler source files, image binaries, fonts and so on.

* [2-build-files](2-build-files) contains build-related scripts, such as the crc32 verification scripts.

* [3-assembled-output](3-assembled-output) contains the output from the assembly process, when the source files are assembled and the results processed by the build files.

* [4-reference-binaries](4-reference-binaries) contains the correct binaries for each variant, so we can verify that our assembled output matches the reference.

* [5-compiled-rom-images](5-compiled-rom-images) contains the final output of the build process: an iNES ROM image that contains the compiled game and which can be run on real hardware or in an emulator.

## Building NES Elite from the source

Builds are supported for both Windows and Mac/Linux systems. In all cases the build process is defined in the `Makefile` provided.

### Requirements

You will need the following to build NES Elite from the source:

* BeebAsm, which can be downloaded from the [BeebAsm repository](https://github.com/stardot/beebasm). Mac and Linux users will have to build their own executable with `make code`, while Windows users can just download the `beebasm.exe` file.

* Python. The build process has only been tested on 3.x, but 2.7 might work.

* Mac and Linux users may need to install `make` if it isn't already present (for Windows users, `make.exe` is included in this repository).

You may be wondering why we're using BeebAsm - a BBC Micro assembler - to build the NES version of Elite. This is because NES Elite is a conversion of BBC Master Elite, which itself is a direct descendant of the original 1984 release for the BBC Micro and Acorn Electron (and the same is true of the Commodore 64 and Apple II versions of Elite - they are all cut from the same cloth). All of the older 6502 versions of Elite were built and assembled on a BBC Micro, including the Commodore and Apple versions, so BeebAsm is a good modern assembler to use for the NES version as well. The 1991 NES version was actually developed on the PC-based PDS (Programmers' Development System), so the use of BeebAsm here isn't historically accurate, it's just a good fit for the source material.

For details of how the build process works, see the [build documentation on bbcelite.com](https://elite.bbcelite.com/about_site/building_elite.html).

Let's look at how to build NES Elite from the source.

### Windows

For Windows users, there is a batch file called `make.bat` which you can use to build the game. Before this will work, you should edit the batch file and change the values of the `BEEBASM` and `PYTHON` variables to point to the locations of your `beebasm.exe` and `python.exe` executables. You also need to change directory to the repository folder (i.e. the same folder as `make.bat`).

All being well, entering the following into a command window:

```
make.bat
```

will produce a file called `elite-pal.NES` in the `5-compiled-rom-images` folder that contains the PAL variant, which you can then load into an emulator, or into a real NES using a flash cart.

### Mac and Linux

The build process uses a standard GNU `Makefile`, so you just need to install `make` if your system doesn't already have it. If BeebAsm or Python are not on your path, then you can either fix this, or you can edit the `Makefile` and change the `BEEBASM` and `PYTHON` variables in the first two lines to point to their locations. You also need to change directory to the repository folder (i.e. the same folder as `Makefile`).

All being well, entering the following into a terminal window:

```
make
```

will produce a file called `elite-pal.NES` in the `5-compiled-rom-images` folder that contains the PAL variant, which you can then load into an emulator, or into a real NES using a flash cart.

### Build options

By default the build process will create a typical Elite game ROM with a standard commander and verified binaries. There are various arguments you can pass to the build to change how it works. They are:

* `variant=<name>` - Build the specified variant:

  * `variant=pal` (default)
  * `variant=ntsc`

* `commander=max` - Start with a maxed-out commander (specifically, this is the test commander file from the original source, which is almost but not quite maxed-out)

* `match=no` - Do not attempt to match the original game binaries (i.e. omit workspace noise)

* `verify=no` - Disable crc32 verification of the game binaries

So, for example:

`make variant=ntsc commander=max match=no verify=no`

will build an NTSC variant with a maxed-out commander, no workspace noise and no crc32 verification.

See below for more on the verification process.

### Verifying the output

The default build process prints out checksums of all the generated files, along with the checksums of the files from the original sources. You can disable verification by passing `verify=no` to the build.

The Python script `crc32.py` in the `2-build-files` folder does the actual verification, and shows the checksums and file sizes of both sets of files, alongside each other, and with a Match column that flags any discrepancies. If you are building an unencrypted set of files then there will be lots of differences, while the encrypted files should mostly match (see the Differences section below for more on this).

The binaries in the `4-reference-binaries` folder are those extracted from the released version of the game, while those in the `3-assembled-output` folder are produced by the build process. For example, if you don't make any changes to the code and build the project with `make`, then this is the output of the verification process:

```
Results for variant: pal
[--originals--]  [---output----]
Checksum    Size  Checksum    Size  Match  Filename
-----------------------------------------------------
6a32bd20   16384  6a32bd20   16384   Yes   bank0.bin
1840f774   16384  1840f774   16384   Yes   bank1.bin
e08fa78a   16384  e08fa78a   16384   Yes   bank2.bin
e07c0f21   16384  e07c0f21   16384   Yes   bank3.bin
731cd900   16384  731cd900   16384   Yes   bank4.bin
fee7480c   16384  fee7480c   16384   Yes   bank5.bin
500f28cd   16384  500f28cd   16384   Yes   bank6.bin
8e1162f8   16384  8e1162f8   16384   Yes   bank7.bin
4cf12d39  131088  4cf12d39  131088   Yes   elite.bin
eb5e8763      16  eb5e8763      16   Yes   header.bin
```

All the compiled binaries match the originals, so we know we are producing the same final game as the PAL variant.

### Log files

During compilation, details of every step are output in a file called `compile.txt` in the `3-assembled-output` folder. If you have problems, it might come in handy, and it's a great reference if you need to know the addresses of labels and variables for debugging (or just snooping around).

## Building different variants of NES Elite

This repository contains the source code for two different variants of NES Elite:

* The Imagineer PAL variant, which is the only official release of NES Elite

* The NTSC variant from Ian Bell's personal website

* An NTSC hardware variant, which is the NTSC variant with its NMI timings reworked so it runs on a real NTSC console (this is not an original release, so it has no reference binaries)

By default the build process builds the PAL variant, but you can build a specified variant using the `variant=` build parameter.

### Building the PAL variant

You can add `variant=pal` to produce the `elite-pal.NES` file that contains the PAL variant, though that's the default value so it isn't necessary. In other words, you can build it like this:

```
make.bat variant=pal
```

or this on a Mac or Linux:

```
make variant=pal
```

This will produce a file called `elite-pal.NES` in the `5-compiled-rom-images` folder that contains the PAL variant.

The verification checksums for this version are as follows:

```
Results for variant: pal
[--originals--]  [---output----]
Checksum    Size  Checksum    Size  Match  Filename
-----------------------------------------------------
6a32bd20   16384  6a32bd20   16384   Yes   bank0.bin
1840f774   16384  1840f774   16384   Yes   bank1.bin
e08fa78a   16384  e08fa78a   16384   Yes   bank2.bin
e07c0f21   16384  e07c0f21   16384   Yes   bank3.bin
731cd900   16384  731cd900   16384   Yes   bank4.bin
fee7480c   16384  fee7480c   16384   Yes   bank5.bin
500f28cd   16384  500f28cd   16384   Yes   bank6.bin
8e1162f8   16384  8e1162f8   16384   Yes   bank7.bin
4cf12d39  131088  4cf12d39  131088   Yes   elite.bin
eb5e8763      16  eb5e8763      16   Yes   header.bin
```

### Building the NTSC variant

You can build the NTSC variant by appending `variant=ntsc` to the `make` command, like this on Windows:

```
make.bat variant=ntsc
```

or this on a Mac or Linux:

```
make variant=ntsc
```

This will produce a file called `elite-ntsc.NES` in the `5-compiled-rom-images` folder that contains the NTSC variant.

The verification checksums for this version are as follows:

```
Results for variant: ntsc
[--originals--]  [---output----]
Checksum    Size  Checksum    Size  Match  Filename
-----------------------------------------------------
0560a52b   16384  0560a52b   16384   Yes   bank0.bin
c1239b33   16384  c1239b33   16384   Yes   bank1.bin
5e6c3bfb   16384  5e6c3bfb   16384   Yes   bank2.bin
54df916d   16384  54df916d   16384   Yes   bank3.bin
5953c5d4   16384  5953c5d4   16384   Yes   bank4.bin
0dd49e0c   16384  0dd49e0c   16384   Yes   bank5.bin
39255d4f   16384  39255d4f   16384   Yes   bank6.bin
26f0c7de   16384  26f0c7de   16384   Yes   bank7.bin
54386491  131088  54386491  131088   Yes   elite.bin
eb5e8763      16  eb5e8763      16   Yes   header.bin
```

### Differences between the variants

You can see the differences between the variants by searching the source code for `_PAL` (for features in the PAL variant) or `_NTSC` (for features in the NTSC variant). The main differences in the NTSC variant compared to the PAL variant are:

* The two versions count a different number of cycles in the NMI handler (7433 in the PAL version, 6797 in the NTSC version).

* The NTSC version is missing the Imagineer and Nintendo headings from the Start screen.

* The PAL version waits for longer before starting auto-play on the combat demo.

* Each version has its own unique checksum algorithm for the save slots.

* The internal version number is different (the PAL version is "<2.8>" while the NTSC version is "5.0")

* The copyright message hidden in bank 3 is different (the PAL message is "NES ELITE IMAGE 2.8 - 04 MAR 1992" while the NTSC message is "NES ELITE IMAGE 5.2 - 24 APR 1992"

* The first title in the combat demo scroll text is different (the PAL title is "IMAGINEER PRESENTS --- E L I T E --- (C)BRABEN & BELL 1991" while the NTSC title is "NTSC EMULATION --- E L I T E ---  (C)BELL & BRABEN 1991")

* A number of pixel y-coordinate constants in the PAL version are six pixels bigger than in the NTSC version, to cater for the taller screen height.

* The interrupt vectors in banks 0 to 6 that are used during initialisation are subtly different.

* The code for detecting double-taps of the B button when choosing buttons from the icon bar is a bit simpler in the NTSC version.

It's worth noting that the NTSC variant doesn't actually work on an NTSC machine. The NMI timings have been changed to work with some (but not all) emulators in NTSC mode, but it isn't a full NTSC conversion, it's an NTSC emulation (as per the scroll text).

### Building the NTSC hardware variant

You can build a version of the NTSC variant that runs on a real NTSC console by appending `variant=ntsc-hw` to the `make` command, like this on Windows:

```
make.bat variant=ntsc-hw
```

or this on a Mac or Linux:

```
make variant=ntsc-hw
```

This will produce a file called `ELITE-ntsc-hw.NES` in the `5-compiled-rom-images` folder. As this isn't an original release there are no reference binaries, so the build isn't verified. You can see all the changes by searching the source code for `_NTSC_HW`.

Add `shift=6` to move the picture down six scanlines, which makes scenes with big ships noticeably faster. On a CRT this pushes the bottom of the dashboard into the overscan area. See [README-NTSC-HW.md](README-NTSC-HW.md) for details.

The NMI handler in NES Elite blanks the screen, sends as much data to the PPU as its cycle budget allows, and then re-enables the screen at a fixed point. The PAL release has a 70-line VBlank and re-enables the screen on scanline 7 (hence the `YPAL` margin), while the NTSC variant's budget of 6797 cycles is nearly three times the length of a real NTSC VBlank (20 lines, about 2270 cycles), so on real hardware it re-enables the screen around scanline 50, which pushes the picture down the screen, breaks the icon bar split and loses the bottom of the dashboard.

The NTSC hardware variant makes these changes to fit into the NTSC VBlank:

* The NMI cycle budget is `NMI_CYCLES_NTSC_HW`, so the screen is re-enabled at the same point as the PAL release, near the start of scanline 7, which is in the overscan area of an NTSC TV. Measured on a cycle-accurate emulator across title, demo, combat, docked screens and flight, the restart always falls between line 6 dot 334 and line 7 dot 192, well inside the safe window of line 6 dot 257 to line 7 dot 255.

* `SetScrollNTSC` loads the PPU address register directly with a fine y-scroll of 6 before the screen is re-enabled, as the pre-render line has passed with rendering disabled. This means nametable row r appears on scanline r + 1, which is the screen layout that the NTSC variant's coordinates (`YPAL` = 0) are designed for, so the whole dashboard fits inside the visible area of an NTSC TV.

* `SendBarNamesNTSC` and `SendBarPattsNTSC` split the icon bar update across two VBlanks. In the original code, sending the icon bar's nametable entries and its first batch of patterns has to happen in a single VBlank, which doesn't fit into an NTSC VBlank, so the game would hang the first time it updated the icon bar.

* The point at which the NMI handler swaps the visible and hidden bitplanes is `NAME_FLIP_NTSC_HW` (12 batches rather than 48), as the swap relies on the rest of the nametable being sent in the same VBlank. With the original value, the space view shows a half-drawn frame for one VBlank on most redraws.

* `MakeSoundsNTSC` skips one in every six calls to the sound routines, and the note periods in `noteFrequency` are scaled by the ratio of the CPU clocks, so music and sound effects play at the same speed and pitch as the PAL release.

* The NMI timer counts 60 VBlanks per second rather than 50, so the combat demo timer reports real seconds.

An NTSC VBlank only gives the NMI handler about a third of the PPU time that it gets on PAL, and NES Elite only moves the game on once each frame has been sent to the PPU, so on its own, fitting into the NTSC VBlank would make the game run at about 70% of PAL speed in combat. The NTSC hardware variant gets this time back, so the game runs faster than PAL:

* In the space view, the NMI handler sends nametable rows 2 to 19 (576 bytes) for every frame, even though most rows are usually empty apart from the box edges, and this takes up most of the VBlank. `DrawEdgesScanNTSC` scans each frame's nametable buffer when it is handed over to the NMI handler, and `NamesRowCheck` only sends the rows that contain something in this frame, or contained something in the frame that was last sent to the same bitplane (so they get blanked out). The masks are reset whenever the nametables are written in other ways (`InvalidateRowMasks`) or a whole buffer is sent for a new view.

* `SendNamesTail` and `SendPattsTail` keep sending data in batches of eight nametable entries or one pattern when there isn't enough time left in the VBlank for a full batch of 32 entries or three patterns, rather than wasting the rest of the VBlank.

* The cycle costs that the NMI handler charges for the paths that are used in this variant have been measured with a cycle-accurate emulator and corrected where the original values were inaccurate, and `BurnFineNTSC` burns the last 0 to 31 cycles that the burn loop in `ClearBuffers` overshoots by, so the screen is re-enabled at a consistent point while the budget can be used right up to the end.

In the combat demo, measured as iterations of the main flight loop per second on a cycle-accurate emulator, the PAL release manages 13.9, the NTSC hardware variant without these improvements manages 9.7, and with them it manages 16.3 (117% of the PAL release).

To make room for the new code in the fixed bank, `DrawBoxEdges` and `FillMemory` are rewritten as loops (with `ClearSmallNTSC` replacing the computed jump into `FillMemory` that `ClearMemory` uses for blocks of fewer than 256 bytes), and the main-loop part of the row scanning lives in bank 3. The rest of the new code lives in bytes that are unused in the other variants, and the NMI handler code is at exactly the same addresses as in the NTSC variant. This is important because the cycle counts in the NMI handler depend on the exact layout of the code, as moving code can change page-crossing timings.

The changes were verified on the Mesen emulator core, patched to log the scanline and dot of every PPU register write and to check, byte for byte, that every nametable row the NMI handler finishes sending matches the frame that was handed over to it. Across around 15,000 handovers covering the title screen, the combat demo, every docked screen, launching and flight, there were no mismatches and no PPU writes while the screen was being drawn.

See the [accompanying website](https://elite.bbcelite.com/nes/releases.html) for a comprehensive list of differences between the variants.

---

Right on, Commanders!

_Mark Moxon_