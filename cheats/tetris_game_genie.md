# Tetris (NES) Game Genie codes

These codes were made from the US release that the disassembly rebuilds
byte for byte. They have not been checked against any other revision (PAL,
NWC 1990). The game keeps all 32KB of PRG mapped at once, so no code needs
an other-bank check. Every code is the 8-letter form with a compare byte.
All effects come from the disassembly. They have not been tested in play
unless an entry says so.

## Auto-repeat rate is 1 column per 4 frames (normal is 6)

**Codes:** `GEKESOZE`
**Raw form:** `89CD:0C:0A`
(address:new-byte:compare-byte, one 32KB bank)
**Recorded:** 2026-11-01 23:28:40

Tags: `demo-ready` `curated` `play-start`

**Mechanism:** This was the optional code of "Shorter auto-shift delay".
When a held LEFT or RIGHT reaches `DAS_RESET` (`$10`), `shift_tetrimino`
moves the piece and sets `autorepeatX` to `DAS_DELAY` (`$0A`). Six more
frames then pass before the piece moves again:

```
        inc     autorepeatX
        lda     autorepeatX
        cmp     #DAS_RESET
        bmi     @ret
        lda     #DAS_DELAY
        sta     autorepeatX
        jmp     @buttonHeldDown
```

The code changes the `lda #DAS_DELAY` to `lda #$0C`. After each repeat,
`autorepeatX` is 4 steps short of `DAS_RESET`, so the piece moves again 4
frames later. A held button moves the piece one column every 4 frames
(15 columns a second, instead of 10). The rate in frames is `$10` minus
the new byte. "Auto-repeat rate is 1 column per 3 frames" uses `$0D` and
"Auto-repeat rate is 1 column per frame" uses `$0F`.

**Evidence:** `tetris_original.nes` has
`E6 46 A5 46 C9 10 30 3D A9 0A 85 46 4C D7 89` at `$89C4` (`autorepeatX`
is `$46`). `autorepeatX` is only used in `shift_tetrimino`. All 32KB of
PRG is always mapped, so no other bank can match.

**Revisions:** US only. Other revisions were not checked.

**Caveats:**

- Not tested.
- It changes a different byte from "Shorter auto-shift delay" and works
  with it or without it. Without it, the first repeat still waits 16
  frames.
- Works in 1-player, 2-player and the demo. The demo was recorded with
  the normal timing, so pieces in the demo may end up in other columns.
- Holding DOWN still stops sideways movement, as before.
- "Auto-repeat left/right disabled" does nothing
  with this code on, since it turns off the auto-repeat this code speeds
  up.
- It uses the same byte as the other auto-repeat rate entries, so use
  only one of them.

## Shorter auto-shift delay

**Codes:** `PESAKPAE`
**Raw form:** `89D4:09:00`
(address:new-byte:compare-byte, one 32KB bank)
**Recorded:** 2026-11-01 22:09:01

Tags: `demo-ready` `curated` `play-start`

**Mechanism:** This is the Tetris version of the Metroid code "No delay
before auto-fire starts". `shift_tetrimino` moves the piece one column on
the frame LEFT or RIGHT is first pressed, and sets `autorepeatX` to 0.
Each frame the button is held, it adds 1, and the piece moves again when
`autorepeatX` reaches `DAS_RESET` (`$10`). Then it sets `autorepeatX` to
`DAS_DELAY` (`$0A`), so the piece moves again every 6 frames:

```
        inc     autorepeatX
        lda     autorepeatX
        cmp     #DAS_RESET
        bmi     @ret
        lda     #DAS_DELAY
        sta     autorepeatX
        jmp     @buttonHeldDown

@resetAutorepeatX:
        lda     #$00
        sta     autorepeatX
```

The code changes the `lda #$00` to `lda #$09`. A new press now sets
`autorepeatX` 7 steps short of `DAS_RESET`, so the piece moves on the
first frame, again 7 frames later, and then every 6 frames. The wait
before auto-repeat starts (the delayed auto shift, DAS) drops from 16
frames to 7. The delay in frames is `$10` minus the new byte, so other
bytes give other delays (`$0F` is 1 frame, `$0C` is 4, `$0A` is 6).

For faster repeats after the delay, see "Auto-repeat rate is 1 column
per 3 frames" and "Auto-repeat rate is 1 column per frame".

**Evidence:** `tetris_original.nes` has
`E6 46 A5 46 C9 10 30 3D A9 0A 85 46 4C D7 89 A9 00 85 46` at `$89C4`
(`autorepeatX` is `$46`). `autorepeatX` is only used in `shift_tetrimino`.
All 32KB of PRG is always mapped, so no other bank can match.

**Revisions:** US only. Other revisions were not checked.

**Caveats:**

- Tested in play by the user. 7 frames was chosen after trying 4, 6 and 7.
- Works in 1-player, 2-player and the demo. The demo was recorded with
  the normal timing, so pieces in the demo may end up in other columns.
- A tap still moves the piece only one column. Holding for 7 frames moves
  it two columns.
- Holding DOWN still stops sideways movement, as before.

## Auto-repeat left/right disabled

**Codes:** `AEKAOPLA`
**Raw form:** `89C1:00:03`
(address:new-byte:compare-byte, one 32KB bank)
**Recorded:** 2026-11-01 22:09:32

**Mechanism:** `shift_tetrimino` moves the piece once when LEFT or RIGHT
is first pressed. On later frames it checks whether either button is
still held, and if so counts up `autorepeatX` to move the piece again:

```
        lda     heldButtons
        and     #BUTTON_LEFT+BUTTON_RIGHT
        beq     @ret
        inc     autorepeatX
```

The code changes `and #BUTTON_LEFT+BUTTON_RIGHT` (`#$03`) to `and #$00`.
The result is always 0, so `beq @ret` always returns, as if no button
were held. Only a new press moves the piece.

**Evidence:** `tetris_original.nes` has `A5 B6 29 03 F0 45 E6 46` at
`$89BE` (`heldButtons` is `$B6`, `autorepeatX` is `$46`). The new-press
check before it (`A5 B5 29 03 D0 15` at `$89B8`) is not changed. All 32KB
of PRG is always mapped, so no other bank can match.

**Revisions:** US only. Other revisions were not checked.

**Caveats:**

- Not tested.
- Works in 1-player, 2-player and the demo. The demo was recorded with
  auto-repeat, so pieces in the demo may end up in other columns.
- To move a piece several columns, tap the button once for each column.
- "Shorter auto-shift delay" (`PESAKPAE`) does nothing with
  this code on, since it changes the auto-repeat this code turns off.

## Level 0 speed on every level

**Codes:** `ALNAKPPA` + `AANEEOIP` (use both)
**Raw form:** `8974:30:01` + `8978:00:1D`
(address:new-byte:compare-byte, one 32KB bank)
**Recorded:** 2026-11-01 22:10:18

Tags: `demo-ready` `play-start`

**Mechanism:** `drop_tetrimino` looks up how many frames a piece waits
before it falls one row. Levels 0-28 read `framesPerDropTable` (48 frames
at level 0, down to 2 frames from level 19). Level 29 and up skip the
table and use 1 frame:

```
@lookupDropSpeed:
        lda     #$01
        ldx     levelNumber
        cpx     #$1D
        bcs     @noTableLookup
        lda     framesPerDropTable,x
@noTableLookup:
        sta     dropSpeed
        lda     fallTimer
        cmp     dropSpeed
        bpl     @drop
```

The first code changes `lda #$01` to `lda #$30` (48). The second changes
`cpx #$1D` to `cpx #$00`, so `bcs @noTableLookup` is always taken and the
table is never read. Every level now uses 48 frames per row, the level 0
speed.

For half the level 0 speed, see "Half of Level 0 speed on every level".

**Evidence:** `tetris_original.nes` has
`A9 01 A6 44 E0 1D B0 03 BD 8E 89 85 AF A5 45 C5 AF 10 D2` at `$8973`
(`framesPerDropTable` is `$898E`, `dropSpeed` is `$AF`, `fallTimer` is
`$45`). `dropSpeed` is only set here and cleared at the start of a game.
All 32KB of PRG is always mapped, so no other bank can match.

**Revisions:** US only. Other revisions were not checked.

**Caveats:**

- Not tested.
- The level number, the level colors and the score
  still change as normal. Only the fall speed stays the same.
- Holding DOWN still drops the piece fast, and still scores the
  soft-drop points.
- Works in 2-player and the demo. The demo pieces fall more slowly, so
  the demo plays out differently.
- It changes the same bytes as "Half of Level 0 speed on every level", so
  use only one of the two.

## The legal screen goes by in a moment

**Codes:** `PAUAYXNY` + `PAUEGXNY` (use both)
**Raw form:** `8237:01:FF` + `823C:01:FF`
(address:new-byte:compare-byte, one 32KB bank)
**Recorded:** 2026-11-01 22:11:34

Tags: `play-start` `curated` `utility`

**Mechanism:** This is the Tetris version of the "skip the intro" codes
in the SNES files (ALTTP, Super Metroid, EarthBound). After it draws the
legal screen, `gameMode_legalScreen` waits 255 frames that can't be
skipped, then up to 255 more frames that START can skip:

```
        lda     #LEGAL_SLEEP_TIME
        jsr     sleep_for_a_vblanks
        lda     #LEGAL_SLEEP_TIME
        sta     generalCounter
@waitForStartButton:
        lda     newlyPressedButtons_player1
        cmp     #BUTTON_START
        beq     @continueToNextScreen
        jsr     updateAudioWaitForNmiAndResetOamStaging
        dec     generalCounter
        bne     @waitForStartButton
```

The two codes change both `lda #LEGAL_SLEEP_TIME` (`#$FF`) to `lda #$01`,
so each wait lasts one frame. The title screen comes up about 8.5 seconds
sooner.

**Evidence:** `tetris_original.nes` has
`A9 FF 20 59 A4 A9 FF 85 A8 A5 F5 C9 10 F0 07` at `$8236`
(`sleep_for_a_vblanks` is `$A459`, `generalCounter` is `$A8`,
`newlyPressedButtons_player1` is `$F5`). All 32KB of PRG is always
mapped, so no other bank can match.

**Revisions:** US only. The PAL release uses `$CC` here and was not
checked.

**Caveats:**

- Not tested.
- The legal screen still flashes up for a few frames. A code that skips
  it fully would have to start the game in `gameMode` 1, and the only
  one-byte way to do that at reset also changes `gameModeState`, which
  the main loop compares with each frame.
- With only the first code, the screen still waits about 4 seconds, but
  START skips it at once.

## Only I pieces

**Codes:** `SZEPUPAX` + `GIEPKPYA` (use both)
**Raw form:** `9903:AD:20` + `9904:54:07`
(address:new-byte:compare-byte, one 32KB bank)
**Recorded:** 2026-11-01 22:13:43

Tags: `demo-ready` `play-start` `video-ready`

**Mechanism:** This works the same way as "Only S pieces". The two codes
change `jsr @realStart` (`20 07 99`) at the start of
`pickRandomTetrimino` to `lda $9954` (`AD 54 99`). `$9954` is the
`iHoriz` byte of `spawnTable`. Every new piece is now an I (the long
bar), and the random-number code is never run.

**Evidence:** `tetris_original.nes` has `20 07 99 60` at `$9903`
(`@realStart` is `$9907`) and `02 07 08 0A 0B 0E 12` at `$994E`
(`spawnTable`; `iHoriz` is `$12`). All 32KB of PRG is always mapped, so
no other bank can match.

**Revisions:** US only. Other revisions were not checked.

**Caveats:**

- Not tested.
- Covers the first piece, the next-piece preview and both players in
  2-player. The demo has its own piece list and is not changed.
- The statistics count every piece as an I.
- Shares `$9903` with "Only O pieces" and "Only S pieces". Use only one
  of the three.

## Only O pieces

**Codes:** `SZEPUPAX` + `PIEPKPYA` (use both)
**Raw form:** `9903:AD:20` + `9904:51:07`
(address:new-byte:compare-byte, one 32KB bank)
**Recorded:** 2026-11-01 23:49:17

Tags: `demo-ready` `play-start` `video-ready`

**Mechanism:** This works the same way as "Only S pieces". The two codes
change `jsr @realStart` (`20 07 99`) at the start of
`pickRandomTetrimino` to `lda $9951` (`AD 51 99`). `$9951` is the
`oFixed` byte of `spawnTable`. Every new piece is now an O (the square),
and the random-number code is never run.

**Evidence:** `tetris_original.nes` has `20 07 99 60` at `$9903`
(`@realStart` is `$9907`) and `02 07 08 0A 0B 0E 12` at `$994E`
(`spawnTable`; `oFixed` is `$0A`). All 32KB of PRG is always mapped, so
no other bank can match.

**Revisions:** US only. Other revisions were not checked.

**Caveats:**

- Not tested.
- Covers the first piece, the next-piece preview and both players in
  2-player. The demo has its own piece list and is not changed.
- The statistics count every piece as an O.
- Shares `$9903` with "Only I pieces" and "Only S pieces". Use only one
  of the three.

## Only S pieces

**Codes:** `SZEPUPAX` + `ZIEPKPYA` (use both)
**Raw form:** `9903:AD:20` + `9904:52:07`
(address:new-byte:compare-byte, one 32KB bank)
**Recorded:** 2026-11-01 22:13:43

Tags: `demo-ready` `play-start` `video-ready` `challenge`

**Mechanism:** Outside the demo, `chooseNextTetrimino` gets every new
piece from `pickRandomTetrimino`, which returns the piece's spawn
orientation from `spawnTable`:

```
pickRandomTetrimino:
        jsr     @realStart
        rts
...
spawnTable:
        .byte   tDown, jDown, zHoriz, oFixed, sHoriz, lDown, iHoriz
```

The two codes change `jsr @realStart` (`20 07 99`) to `lda $9952`
(`AD 52 99`). `$9952` is the `sHoriz` byte of `spawnTable`. Every new
piece is now an S, and the random-number code is never run.

**Evidence:** `tetris_original.nes` has `20 07 99 60` at `$9903`
(`@realStart` is `$9907`) and `02 07 08 0A 0B 0E 12` at `$994E`
(`spawnTable`; `sHoriz` is `$0B`). `@realStart` and `spawnTable` are in
the same page, so only the opcode and the low byte change. All 32KB of
PRG is always mapped, so no other bank can match.

**Revisions:** US only. Other revisions were not checked.

**Caveats:**

- Not tested.
- Covers the first piece, the next-piece preview and both players in
  2-player. The demo has its own piece list and is not changed.
- The statistics count every piece as an S.
- Lines are hard to clear with only S pieces, so games end quickly.
- Shares `$9903` with "Only I pieces" and "Only O pieces". Use only one
  of the three.

## Piece order is always T J Z O S L I S, repeating

**Codes:** `APSZVUAU`
**Raw form:** `AB56:18:38`
(address:new-byte:compare-byte, one 32KB bank)
**Recorded:** 2026-11-01 22:14:48

Tags: `interesting`

**Mechanism:** This is the Tetris version of the SMB1 code "PRNG
feedback tap A (kill feedback)" and the SMW code "Nothing in the game is
random any more". `generateNextPseudorandomNumber` shifts the 16-bit
`rng_seed` right by one bit. The new top bit is bit 1 of the first byte
XOR bit 1 of the second byte:

```
        lda     tmp1,x
        and     #$02
        sta     tmp1
        lda     tmp2,x
        and     #$02
        eor     tmp1
        clc
        beq     @updateNextByteInSeed
        sec
@updateNextByteInSeed:
        ror     tmp1,x
```

The code changes `sec` to `clc`, so the new bit is always 0. The NMI
runs this routine every frame, so 16 frames after reset `rng_seed` is 0
and stays 0.

`pickRandomTetrimino` adds `spawnCount` (which goes up by 1 for each
piece) to `rng_seed` and uses the low 3 bits as an index into
`spawnTable`. With `rng_seed` at 0, the index just counts 0-7. Index 7
is not valid, so it falls back to `(rng_seed AND 7) + spawnID` mod 7,
which after an I piece is 4, an S. The pieces repeat in this order:
T, J, Z, O, S, L, I, S.

**Evidence:** `tetris_original.nes` has
`B5 00 29 02 85 00 B5 01 29 02 45 00 18 F0 01 38 76 00 E8 88` at `$AB47`
(`generateNextPseudorandomNumber`). All 32KB of PRG is always mapped, so
no other bank can match.

**Revisions:** US only. Other revisions were not checked.

**Caveats:**

- Not tested.
- The first piece of a game depends on `spawnCount`, which is never
  reset, so each game starts at a different place in the order.
- B-type garbage reads `rngTable` with `rng_seed AND 7`, which is now
  always `tileEmpty`. A B-type game with a height has no garbage.
- The demo has its own piece list and is not changed.
- An "alternate seed" code was not made. The seed is set once at reset
  (`ldx #$89`), but the NMI steps the random numbers every frame, so the
  pieces already depend on when START is pressed and a new seed would
  not be seen.
- With "Only S pieces" or "Only I pieces" on, this code doesn't change
  the pieces, since those skip `pickRandomTetrimino`'s random code.

## Pieces don't fall unless you hold Down

**Codes:** `AEEASPXI`
**Raw form:** `8985:00:D2`
(address:new-byte:compare-byte, one 32KB bank)
**Recorded:** 2026-11-01 22:17:07

Tags: `demo-ready` `play-start`

**Mechanism:** This is a Tetris take on the "time moves only while a
button is held" codes (Metroid, SMB1, Zelda, Super Metroid).
`drop_tetrimino` has two ways to move the piece down a row. Holding DOWN
goes straight to `@drop`. Otherwise, gravity drops the piece once
`fallTimer` reaches the level's `dropSpeed`:

```
@noTableLookup:
        sta     dropSpeed
        lda     fallTimer
        cmp     dropSpeed
        bpl     @drop
        jmp     @ret
```

The code sets the `bpl @drop` offset to `$00`, so the branch goes to the
next instruction (`jmp @ret`) either way. Gravity never moves the piece.
Only DOWN does.

**Evidence:** `tetris_original.nes` has `A5 45 C5 AF 10 D2 4C` at `$8980`
(`fallTimer` is `$45`, `dropSpeed` is `$AF`). The branch goes back to
`@drop` at `$8958`. All 32KB of PRG is always mapped, so no other bank
can match.

**Revisions:** US only. Other revisions were not checked.

**Caveats:**

- Not tested.
- A piece doesn't lock until DOWN pushes it onto the stack, so you have
  as long as you like to move and turn it.
- Soft drop works as before, one row every 2 frames, and still scores
  soft-drop points.
- The demo pieces only move down when the recorded input holds DOWN, so
  the demo plays out differently.
- "Level 0 speed on every level" does nothing with this
  code on.
- A "time moves only while a button is held" or "half speed" code was not
  made. The NMI reads the controller every frame and keeps new presses
  for one frame only, so if the game skipped frames, presses in the
  skipped frames would be lost.

## Music plays a whole tone higher

**Codes:** `APUVNPGO` + `YPKTSPLP` (use both)
**Raw form:** `E93F:18:14` + `E945:17:13`
(address:new-byte:compare-byte, one 32KB bank)
**Recorded:** 2026-11-01 22:19:07

Tags: `interesting` `play-start`

**Mechanism:** This is the Tetris version of the "music and sound
effects play higher" codes (SMB1, Zelda, ALTTP). Each note in
the music data is an index into `noteToWaveTable`, a table of 2-byte
square/triangle timer values, one entry per semitone. The music engine
adds the song's note offset and then reads the entry:

```
@noteOffsetApplied:
        tay
        lda     noteToWaveTable+1,y
        sta     musicStagingSq1Lo,x
        lda     noteToWaveTable,y
        ora     #$08
        sta     musicStagingSq1Hi,x
```

The two codes change the low bytes of both table addresses by 4
(`noteToWaveTable+5` and `noteToWaveTable+4`), so every note reads the
entry two places later, a whole tone up. (Adding 2 would be one semitone
up.)

The rest check before this (`lda noteToWaveTable+1,y` / `beq` at
`$E924`) reads the note before the song offset is added, and is not
changed, so rests stay rests.

**Evidence:** `tetris_original.nes` has
`A8 B9 14 EB 9D 80 06 B9 13 EB 09 08 9D 81 06` at `$E93E`
(`noteToWaveTable` is `$EB13`). The table is chromatic from `$04` (C2)
to `$86` (F7). After the song offsets, the music uses notes `$0E` to
`$7E`, so the shifted notes (at most `$82`) stay in the chromatic part.
All 32KB of PRG is always mapped, so no other bank can match.

**Revisions:** US only. Other revisions were not checked.

**Caveats:**

- Not tested.
- Only the music changes. The sound effects don't use `noteToWaveTable`
  and play as before.
- An octave-higher code was not made. Shifting by 12 entries would put
  the highest notes of Music 1 and the title music on the odd entries at
  `$88` and up (a timer value of 1, and notes out of order).

## All music plays with one instrument

**Codes:** `SXETINOU` + `SXEVZNOU` (use both)
**Raw form:** `E785:AD:B9` + `E78A:AD:B9`
(address:new-byte:compare-byte, one 32KB bank)
**Recorded:** 2026-11-01 22:20:10

Tags: `interesting` `play-start`

**Mechanism:** This is the Tetris version of the "all music plays with
one instrument" codes (SMB1, SMB2, SMB3, Zelda). The `$9F` music command
sets `musicChanControl`, and its low 5 bits pick an instrument: a volume
envelope from `musicChanVolControlTable`. Each frame,
`updateMusicFrame_setChanVol` turns the instrument number into a table
offset in Y and reads the envelope's address:

```
        ldy     #$00
@controlMinus1Times2_storeToY:
        dec     AUDIOTMP2
        beq     @loadFromTable
        iny
        iny
        bne     @controlMinus1Times2_storeToY
@loadFromTable:
        lda     musicChanVolControlTable,y
        sta     AUDIOTMP3
        lda     musicChanVolControlTable+1,y
        sta     AUDIOTMP4
```

The two codes change both `lda ...,y` (`B9`) to plain `lda` (`AD`), so
the reads ignore Y and always get the first entry, `LEA76`. Every square
note now uses instrument 1, a short attack that fades out.

For a code that keeps each song's own instrument changes but shifts every
instrument, see "Each instrument plays as the next one".

**Evidence:** `tetris_original.nes` has
`A0 00 C6 E1 F0 04 C8 C8 D0 F8 B9 4A EA 85 E2 B9 4B EA 85 E3` at `$E77B`
(`musicChanVolControlTable` is `$EA4A`, `AUDIOTMP2` is `$E1`). All 32KB
of PRG is always mapped, so no other bank can match.

**Revisions:** US only. Other revisions were not checked.

**Caveats:**

- Not tested.
- Only the two square channels use these envelopes. The triangle and
  noise (drum) channels sound as before.
- Notes set with instrument 0 (`$9F,$00`) skip the envelope and are not
  changed.
- The duty cycle (tone colour) still comes from each `$9F` command. Only
  the volume shape changes.
- Sound effects are not changed.
- "Each instrument plays as the next one" does nothing with these codes
  on, since they make the table reads ignore Y.

## Holding A while starting a game adds 20 levels

**Codes:** `GPVEPKZA`
**Raw form:** `8469:14:0A`
(address:new-byte:compare-byte, one 32KB bank)
**Recorded:** 2026-11-02 00:15:28

Tags: `curated` `play-start` `utility` `demo-ready`

**Mechanism:** This was the optional code of "Every game starts at level
28". The level menu offers levels 0-9, and holding A while pressing START
adds 10:

```
        lda     player1_startLevel
        clc
        adc     #$0A
        sta     player1_startLevel
```

The code changes `adc #$0A` to `adc #$14` (20). The menu still works as
normal, and A+START now starts 20 levels higher, so level 9 gives level
29 (and level 0 gives 20).

**Evidence:** `tetris_original.nes` has `A5 67 18 69 0A 85 67` at `$8465`
(`player1_startLevel` is `$67`). `levelDisplayTable` and
`framesPerDropTable` both have entries up to level 29, so level 29 (the
highest this code can start) shows as "29" and falls at 1 frame per row.
All 32KB of PRG is always mapped,
so no other bank can match.

**Revisions:** US only. Other revisions were not checked.

**Caveats:**

- Not tested.
- Without A held, the game starts at the level picked on the menu, as
  before.
- The demo is not changed.
- The game moves up a level when `lines / 10` is more than the level
  number, but it compares the binary level number with `lines / 10` in
  BCD. At 200 lines that is `$20` (32), so level 29 (1 frame per row)
  comes at 200 lines, not 290.
- "Every game starts at level 28" replaces whatever the menu picked, so
  this code does nothing useful with it on.

## Every game starts at level 28

**Codes:** `OXXAITSX` + `GOXATTYV` (use both)
**Raw form:** `86A5:A9:A5` + `86A6:1C:67`
(address:new-byte:compare-byte, one 32KB bank)
**Recorded:** 2026-11-01 22:21:16

Tags: `challenge`

**Mechanism:** This is the Tetris version of the "start a new game with
..." codes (Metroid). When a game starts,
`gameModeState_initGameBackground_finish` copies the level picked on the
menu into the level number:

```
        lda     player1_startLevel
        sta     player1_levelNumber
```

The two codes change `lda player1_startLevel` (`A5 67`) to `lda #$1C`
(`A9 1C`), so every game starts at level 28 (`$1C`), whatever level was
picked.

For a code that adds 20 levels when A is held, see "Holding A while
starting a game adds 20 levels".

**Evidence:** `tetris_original.nes` has `A5 67 85 64 A5 87 85 84` at
`$86A5` (`player1_startLevel` is `$67`, `player1_levelNumber` is `$64`).
`levelDisplayTable` and `framesPerDropTable` both have entries up to
level 29, so level 28 shows as "28" and falls at 2 frames per row. All 32KB of PRG is always mapped,
so no other bank can match.

**Revisions:** US only. Other revisions were not checked.

**Caveats:**

- Not tested.
- The level picked on the menu is ignored, in both A-type and B-type.
- The codes also start the demo at level 28, so its pieces fall fast and
  the demo plays out differently.
- The game moves up a level when `lines / 10` is more than the level
  number, but it compares the binary level number with `lines / 10` in
  BCD. At 200 lines that is `$20` (32), so level 29 (1 frame per row)
  comes at 200 lines, not 290.
- "Holding A while starting a game adds 20 levels" does nothing useful
  with these codes on, since they replace whatever the menu picked.
- Shares `$86A5` with "Every game starts at level 29". Use only one of
  the two.

## Every game starts at level 29

**Codes:** `OXXAITSX` + `IOXATTYV` (use both)
**Raw form:** `86A5:A9:A5` + `86A6:1D:67`
(address:new-byte:compare-byte, one 32KB bank)
**Recorded:** 2026-11-02 00:19:20

Tags: `challenge`

**Mechanism:** This works the same way as "Every game starts at level
28". The two codes change `lda player1_startLevel` (`A5 67`) in
`gameModeState_initGameBackground_finish` to `lda #$1D` (`A9 1D`), so
every game starts at level 29 (`$1D`), whatever level was picked.

Level 29 is the first level that skips `framesPerDropTable`
(`cpx #$1D` / `bcs @noTableLookup`), so pieces fall one row every frame.

**Evidence:** `tetris_original.nes` has `A5 67 85 64 A5 87 85 84` at
`$86A5` (`player1_startLevel` is `$67`, `player1_levelNumber` is `$64`).
`levelDisplayTable` has an entry for level 29, so it shows as "29". All
32KB of PRG is always mapped, so no other bank can match.

**Revisions:** US only. Other revisions were not checked.

**Caveats:**

- Not tested.
- The level picked on the menu is ignored, in both A-type and B-type.
- At 1 frame per row, a piece reaches the bottom almost at once, so the
  game is very hard to play.
- The codes also start the demo at level 29, so its pieces fall fast and
  the demo plays out differently.
- "Holding A while starting a game adds 20 levels" does nothing useful
  with these codes on, since they replace whatever the menu picked.
- Shares `$86A5` with "Every game starts at level 28". Use only one of
  the two.

## SELECT throws away the falling piece and brings in the next one

**Codes:** `OXXAXASX` + `PEXAUENS` + `AVXANENS` (use all 3)
**Raw form:** `88A2:A9:A5` + `88A3:09:DF` + `88A7:68:DF`
(address:new-byte:compare-byte, one 32KB bank)
**Recorded:** 2026-11-01 22:22:39

Tags: `demo-ready` `curated` `play-start` `video-ready`

**Mechanism:** This is the Tetris version of the "Select does ..." codes
(Zelda, SMB3, SMW). In the normal game, SELECT turns the next-piece
preview on and off, in `gameModeState_updateCountersAndNonPlayerState`:

```
@checkSelectButtonPressed:
        lda     newlyPressedButtons_player1
        and     #BUTTON_SELECT
        beq     @ret
        lda     displayNextPiece
        eor     #$01
        sta     displayNextPiece
```

The codes change `lda displayNextPiece` to `lda #$09` and
`sta displayNextPiece` to `sta player1_playState`. The `eor #$01` is
kept, so SELECT now stores `$08` in `player1_playState`, which is
`playState_spawnNextTetrimino`. On the next frame the next piece appears
at the top of the well, and a new next piece is picked.

The falling piece is not part of the playfield until it locks, so it
just goes away.

**Evidence:** `tetris_original.nes` has
`A5 F5 29 20 F0 06 A5 DF 49 01 85 DF` at `$889C`
(`newlyPressedButtons_player1` is `$F5`, `displayNextPiece` is `$DF`).
`player1_playState` is `$68`. This code runs before `makePlayer1Active`
copies the player data to `$40`, so `$68` holds the live value. All 32KB
of PRG is always mapped, so no other bank can match.

**Revisions:** US only. Other revisions were not checked.

**Caveats:**

- Not tested.
- Press SELECT only while a piece is falling. The code changes the play
  state at any time, so a press while rows are being cleared can skip the
  rest of the line clear, and a press during the game over curtain brings
  in a new piece, which starts the game over again when it locks.
- SELECT no longer hides the next-piece preview.
- The statistics count both the thrown-away piece and the new one.
- The new piece starts at the top, so this can also be used to get more
  time.

## B-type is won after 1 line

**Codes:** `PASEGYIZ`
**Raw form:** `875C:01:25`
(address:new-byte:compare-byte, one 32KB bank)
**Recorded:** 2026-11-01 22:24:47

Tags: `demo-ready` `video-ready` `interesting`

**Mechanism:** In a B-type game, `lines` counts down from 25, and
`playState_bTypeGoalCheck` plays the B-type ending when it reaches 0.
`gameModeState_initGameState` sets the starting count:

```
        lda     gameType
        beq     @skipTypeBInit
        lda     #$25
        sta     player1_lines
        sta     player2_lines
```

The code changes `lda #$25` to `lda #$01`, so a B-type game starts with
1 line to go. The first line cleared brings the count to 0, and the game
goes on to the B-type ending.

**Evidence:** `tetris_original.nes` has
`A5 C1 F0 06 A9 25 85 70 85 90` at `$8757` (`gameType` is `$C1`,
`player1_lines` is `$70`). The countdown in
`playState_updateLinesAndStatistics` stops at 0, so clearing 2-4 lines
at once also works. All 32KB of PRG is always mapped, so no other bank
can match.

**Revisions:** US only. Other revisions were not checked.

**Caveats:**

- Not tested.
- A-type games are not changed. See "A-type ends with the rocket after
  the first line" for those.
- The ending you see depends on the level and height picked, as in the
  normal game. The level and height bonuses are still added.

## A-type ends with the rocket after the first line

**Codes:** `AENZILIL` + `ENNXALEI` (use both)
**Raw form:** `A3F5:00:35` + `A3F8:F0:D0`
(address:new-byte:compare-byte, one 32KB bank)
**Recorded:** 2026-11-01 22:25:00

Tags: `interesting` `play-start`

**Mechanism:** After each piece locks, `playState_bTypeGoalCheck` checks
for the end of a B-type game:

```
playState_bTypeGoalCheck:
        lda     gameType
        beq     playState_bTypeGoalCheck_ret
        lda     lines
        bne     playState_bTypeGoalCheck_ret
@gameOver:
        lda     #$02
        jsr     setMusicTrack
```

The first code sets the `beq` offset to `$00`, so A-type games also go on
to the `lines` check. The second changes `bne` to `beq`, so the check
passes once `lines` is not 0. In an A-type game, `lines` counts up, so
the first line cleared ends the game. The game plays the B-type success
jingle and graphic, then calls `endingAnimation`, which shows the A-type
rocket ending because `gameType` is 0. After that comes the usual game
over and high score entry.

**Evidence:** `tetris_original.nes` has
`A5 C1 F0 35 A5 50 D0 31 A9 02 20 07 9E` at `$A3F2` (`gameType` is
`$C1`, `lines` is `$50`). All 32KB of PRG is always mapped, so no other
bank can match.

**Revisions:** US only. Other revisions were not checked.

**Caveats:**

- Not tested.
- `selectEndingScreen` picks the rocket from the score. With a score
  under 50,000 it shows the smallest rocket (the same one as for 30,000).
- Use only in A-type. In B-type, `lines` starts at 25, so the game ends as
  soon as the first piece locks.
- Don't use with "B-type is won after 1 line" in a B-type game, for the
  same reason.

## Level palettes are read from past the end of color table

**Codes:** `IYXONEIG` + `TYUPSETG` + `YYUOUEYG` (use all 3)
**Raw form:** `982F:75:4D` + `9835:76:4E` + `983B:77:4F`
(address:new-byte:compare-byte, one 32KB bank)
**Recorded:** 2026-11-02 00:32:16

Tags: `demo-ready` `play-start` `curated` `video-ready`

**Mechanism:** This works the same way as "Level 0 has the Dusk colors and
level 2 has the Charcoal colors". `updatePaletteForLevel` takes the level
number mod 10, times 4, as an offset into `colorTable`, and copies 4 bytes
into background palette 2 and sprite palette 2, which the blocks in the
well and the falling piece use:

```
        asl     a
        asl     a
        tax
...
        lda     colorTable,x
        sta     PPUDATA
        lda     colorTable+1,x
        sta     PPUDATA
        lda     colorTable+1+1,x
        sta     PPUDATA
        lda     colorTable+1+1+1,x
        sta     PPUDATA
```

`colorTable` has 10 entries of 4 bytes (40 bytes, `$984C` to `$9873`). The
first 4 bytes after it, at `$9874`, are `60 E6 69 A5`: the `rts` and the
start of the code that follows the table.

The three codes move the last three reads `$28` bytes (10 entries) later
(`colorTable+$29`, `+$2A` and `+$2B`). Level 0 (offset `$00`) now reads
the entry at `$9874`. The first read is color 0 of the palette, which is
never shown, so it is left alone. The PPU drops the top 2 bits of each
palette byte, so level 0 gets the colors `$26` (red), `$29` (green) and
`$25` (pink).

**Evidence:** `tetris_original.nes` has
`BD 4C 98 8D 07 20 BD 4D 98 8D 07 20 BD 4E 98 8D 07 20 BD 4F 98` at
`$9828` (`colorTable` is `$984C`, `PPUDATA` is `$2007`) and
`60 E6 69 A5 69 C9 14 30 04 A9 20 85 69 E6 89 A5 89 C9 14 30 04 A9 20 85
89 60 A5 49 C9 20 30 56 A5 BE C9 01 F0 20 A5 A4` at `$9874`. All 32KB of
PRG is always mapped, so no other bank can match.

**Revisions:** US only. Other revisions were not checked.

**Caveats:**

- Not tested.
- Every level moves, not just level 0. Level N (mod 10) reads the entry
  at `$9874` + 4 * N:

  | Level | Entry at | Colors |
  |---|---|---|
  | 0 | `$9874` | `$26 $29 $25` |
  | 1 | `$9878` | `$09 $14 $30` |
  | 2 | `$987C` | `$29 $20 $05` |
  | 3 | `$9880` | `$26 $09 $25` |
  | 4 | `$9884` | `$09 $14 $30` |
  | 5 | `$9888` | `$29 $20 $05` |
  | 6 | `$988C` | `$20 $25 $09` |
  | 7 | `$9890` | `$20 $30 $16` |
  | 8 | `$9894` | `$3E $09 $01` |
  | 9 | `$9898` | `$20 $25 $24` |

  Levels 10 to 29 use the same entries as levels 0 to 9.
- The colors replace the white of each block too, so the blocks have no
  white highlight.
- "Level 0 has the Dusk colors and level 2 has the Charcoal colors"
  changes the same three bytes. Use only one of the two.

## Level 0 has the Dusk colors and level 2 has the Charcoal colors

**Codes:** `SPXONEIG` + `VPUPSETG` + `NPUOUEYG` (use all 3)
**Raw form:** `982F:95:4D` + `9835:96:4E` + `983B:97:4F`
(address:new-byte:compare-byte, one 32KB bank)
**Recorded:** 2026-11-01 22:34:39

Tags: `demo-ready` `play-start` `video-ready`

**Mechanism:** "Dusk" (level 146) and "Charcoal" (level 148) are the two
darkest of the glitched level colors that players see past level 137.
`updatePaletteForLevel` takes the level number mod 10, times 4, as an
offset into `colorTable`:

```
updatePaletteForLevel:
        lda     player1_levelNumber
@mod10: cmp     #$0A
        bmi     @copyPalettes
        sec
        sbc     #$0A
        jmp     @mod10

@copyPalettes:
        asl     a
        asl     a
        tax
...
        lda     colorTable,x
        sta     PPUDATA
        lda     colorTable+1,x
        sta     PPUDATA
        lda     colorTable+1+1,x
        sta     PPUDATA
        lda     colorTable+1+1+1,x
        sta     PPUDATA
```

`bmi` is a signed test, so from level 128 on the mod 10 is skipped, and
`asl` drops the top bits. Level 146 reads offset `$48` and level 148
reads offset `$50`, both past the end of the 40-byte table, in the code
that follows it.

The three codes move the last three reads `$48` bytes later
(`colorTable+$49`, `+$4A` and `+$4B`). Level 0 (offset `$00`) now reads
the colors at `$48`, the same as level 146, and level 2 (offset `$08`)
reads the colors at `$50`, the same as level 148. The first read is
color 0 of the palette, which is never shown, so it is left alone.

**Evidence:** `tetris_original.nes` has
`BD 4C 98 8D 07 20 BD 4D 98 8D 07 20 BD 4E 98 8D 07 20 BD 4F 98` at
`$9828` (`colorTable` is `$984C`, `PPUDATA` is `$2007`). The bytes at
`$9894` are `A5 BE C9 01` (Dusk: colors `$3E`, `$09`, `$01` once the PPU
drops the top 2 bits) and at `$989C` are `C9 00 D0 0E` (Charcoal: colors
`$00`, `$10`, `$0E`). All 32KB of PRG is always mapped, so no other bank
can match.

**Revisions:** US only. Other revisions were not checked.

**Caveats:**

- Not tested.
- Every level moves, not just levels 0 and 2. Level N (mod 10) gets the
  colors of level 146 + N:

  | Level | Colors of | Colors |
  |---|---|---|
  | 0 | 146 (Dusk) | `$3E $09 $01` |
  | 1 | 147 | `$20 $25 $24` |
  | 2 | 148 (Charcoal) | `$00 $10 $0E` |
  | 3 | 149 | `$24 $25 $37` |
  | 4 | 150 | `$25 $20 $2B` |
  | 5 | 151 | `$05 $26 $0C` |
  | 6 | 152 | `$18 $25 $25` |
  | 7 | 153 | `$37 $10 $36` |
  | 8 | 154 | `$24 $09 $1C` |
  | 9 | 155 | `$30 $29 $00` |

  Levels 10, 12, 20 and 22 also get Dusk or Charcoal.
- The colors replace the white of each block too, as in the real glitch.
  Dusk is very hard to see.
- "Level palettes are read from past the end of color table" changes the
  same three bytes. Use only one of the two.

## The level goes up with every piece

**Codes:** `VVUOKASA` + `GKUOSAKZ` + `ZKOOVPAG` (use all 3)
**Raw form:** `98BC:E6:85` + `98BD:44:A4` + `999E:42:40`
(address:new-byte:compare-byte, one 32KB bank)
**Recorded:** 2026-11-01 22:31:10

Tags: `demo-ready` `play-start` `curated` `interesting` `video-ready` `challenge`

**Mechanism:** After a piece locks, `playState_spawnNextTetrimino`
brings in the next one:

```
@spawnPiece:
        lda     #$00
        sta     twoPlayerPieceDelayCounter
        sta     fallTimer
        sta     tetriminoY
```

`twoPlayerPieceDelayCounter` is only used in 2-player games and is
always 0 in a 1-player game. The first two codes change
`sta twoPlayerPieceDelayCounter` (`85 A4`) to `inc levelNumber`
(`E6 44`), so each new piece adds 1 to the level. The level's fall speed
is read from `levelNumber` on every frame, so it takes effect at once.

`@spawnPiece` then calls `incrementPieceStat`, which marks the
statistics for redrawing:

```
        lda     outOfDateRenderFlags
        ora     #RENDER_STATS
        sta     outOfDateRenderFlags
```

The third code changes `ora #RENDER_STATS` (`#$40`) to
`ora #RENDER_STATS+RENDER_LEVEL` (`#$42`), so the level number and the
level colors are also redrawn for each new piece.

The normal level-up from clearing lines still works too.

**Evidence:** `tetris_original.nes` has `A9 00 85 A4 85 45 85 41` at
`$98BA` (`twoPlayerPieceDelayCounter` is `$A4`, `levelNumber` is `$44`
while the player's data is copied to `$40`) and `A5 A3 09 40 85 A3` at
`$999C` (`outOfDateRenderFlags` is `$A3`). All 32KB of PRG is always
mapped, so no other bank can match.

**Revisions:** US only. Other revisions were not checked.

**Caveats:**

- Not tested.
- Level 29 (1 frame per row) comes after 29 pieces from level 0.
- Past level 29 the level display shows odd numbers and tiles, and past
  level 137 the colors glitch, as in the real game. The level goes back
  to 0 after 255.
- The first piece of a game doesn't count, since it doesn't come from
  `@spawnPiece`.
- Use all 3 codes. With only the first code, the new instruction is
  `inc twoPlayerPieceDelayCounter`, and the level doesn't change.

## Scoring lags as at level 157, so the game can crash in level 0

**Codes:** `OZUPOGSX` + `SPUPXGGK` (use both)
**Raw form:** `9C31:A9:A5` + `9C32:9D:44`
(address:new-byte:compare-byte, one 32KB bank)
**Recorded:** 2026-11-01 22:32:31

Tags: `interesting` `curated`

**Mechanism:** This brings the crash that ended the first "beaten" games
of Tetris (level 157 and nearby) into the first levels. After every
piece locks, `addLineClearPoints` adds the points for the lines cleared
once for each level, plus one, in BCD:

```
addLineClearPoints:
        lda     #$00
        sta     holdDownPoints
        lda     levelNumber
        sta     generalCounter
        inc     generalCounter
L9C37:  lda     completedLines
        asl     a
        tax
        lda     pointsTable,x
...
L9C94:  dec     generalCounter
        bne     L9C37
```

At very high levels this loop runs long enough that the frame's work is
not done when the NMI comes. The NMI calls
`generateNextPseudorandomNumber`, which uses `tmp1` (`$00`) as scratch.
If the NMI lands inside `switch_s_plus_2a`, after it has put a jump
address in `tmp1`/`tmp2` and before `jmp (tmp1)`, the jump goes to a
wrong address and the game crashes.

The codes change `lda levelNumber` (`A5 44`) to `lda #$9D` (157), so the
loop always runs 158 times, as at level 157, whatever the real level is.

**Evidence:** `tetris_original.nes` has
`A9 00 85 4F A5 44 85 A8 E6 A8` at `$9C2D` (`holdDownPoints` is `$4F`,
`levelNumber` is `$44`, `generalCounter` is `$A8`) and
`B5 00 29 02 85 00` at `$AB47` (the RNG's `sta tmp1`). The loop runs
after every lock, also when no lines are cleared (`pointsTable` gives 0
points then). All 32KB of PRG is always mapped, so no other bank can
match.

**Revisions:** US only. Other revisions were not checked.

**Caveats:**

- Not tested.
- Whether a lock crashes the game depends on where the NMI lands, as at
  level 157. Not every lock or line clear crashes, and a game may go on
  for a while first.
- The game crashes at any level, not only level 0. Every level now
  scores 158 times the normal points: a single is worth 6,320, and a
  Tetris is worth 189,600. The score stops at 999,999.
- Soft-drop points are not multiplied.
- Each lock now lags the game for a frame or so, which may be visible.

## I pieces have only 3 squares

**Codes:** `NYVEEUVN` + `NYNAVUVN` (use both)
**Raw form:** `8B68:FF:FE` + `8B76:FF:FE`
(address:new-byte:compare-byte, one 32KB bank)
**Recorded:** 2026-11-01 22:35:41

Tags: `demo-ready` `play-start` `video-ready`

**Mechanism:** `orientationTable` (`$8A9C`) holds 3 bytes for each square
(y offset, tile, x offset), 4 squares for each orientation. The I piece
has 2 orientations:

```
        .byte   -2, tile1, 0,-1, tile1, 0, 0, tile1, 0, 1, tile1, 0 ; $11 i vertical
        .byte    0, tile1,-2, 0, tile1,-1, 0, tile1, 0, 0, tile1, 1 ; $12 i horizontal (spawn)
```

The codes change the first square of each from offset -2 to -1, so it
sits on the second square. The piece is then 3 squares long (offsets -1,
0 and 1) and turns around its middle square. The table is read in the
same way to draw the piece (`stageSpriteForCurrentPiece`), to test moves
(`isPositionValid`) and to lock it into the playfield, so all three see
the same 3-square piece. The doubled square is written twice to the
same place, which does no harm.

**Evidence:** `tetris_original.nes` has
`FE 7B 00 FF 7B 00 00 7B 00 01 7B 00 00 7B FE 00 7B FF 00 7B 00 00 7B 01`
at `$8B68` (orientations `$11` and `$12`, `tile1` is `$7B`). All 32KB of
PRG is always mapped, so no other bank can match.

**Revisions:** US only. Other revisions were not checked.

**Caveats:**

- Not tested.
- The next-piece box and the statistics on the left still show the I
  piece with 4 squares. The next-piece box uses its own sprite data, and
  the statistics are background tiles.
- No piece is 4 squares tall any more, so a Tetris (4 lines at once) is
  not possible. 3 lines is the most.

## All pieces have only 3 squares in strange arrangements

**Codes:** `LAKEEZGA` + `LEXPYGGA` + `LESOXPGA` (use all 3)
**Raw form:** `8A48:03:04` + `94A7:03:04` + `99DA:03:04`
(address:new-byte:compare-byte, one 32KB bank)
**Recorded:** 2026-11-01 22:36:38

Tags: `demo-ready` `curated` `play-start` `video-ready`

**Mechanism:** Three loops read the 4 squares of the falling piece from
`orientationTable`: one draws it (`stageSpriteForCurrentPiece`), one tests
a move (`isPositionValid`) and one locks it into the playfield
(`playState_lockTetrimino`). The draw loop sets its count like this, and
the other two do the same with `generalCounter3`:

```
        lda     #$04
        sta     generalCounter2 ; iterate through all four minos
```

The codes change all three counts to 3, so the last square of every
orientation is skipped. The pieces become these shapes (y, x offsets of
the 3 squares that are left):

| Piece | Orientation | Squares | Shape |
|---|---|---|---|
| T | $00 up, $02 down (spawn) | (0,-1) (0,0) (0,1) | line |
| T | $01 right | (-1,0) (0,0) (0,1) | corner |
| T | $03 left | (-1,0) (0,-1) (0,0) | corner |
| J | $04 left | (-1,0) (0,0) (1,-1) | not joined |
| J | $05 up | (-1,-1) (0,-1) (0,0) | corner |
| J | $06 right | (-1,0) (-1,1) (0,0) | corner |
| J | $07 down (spawn) | (0,-1) (0,0) (0,1) | line |
| Z | $08 (spawn) | (0,-1) (0,0) (1,0) | corner |
| Z | $09 | (-1,1) (0,0) (0,1) | corner |
| O | $0A (spawn) | (0,-1) (0,0) (1,-1) | corner |
| S | $0B (spawn) | (0,0) (0,1) (1,-1) | not joined |
| S | $0C | (-1,0) (0,0) (0,1) | corner |
| L | $0D right | (-1,0) (0,0) (1,0) | line |
| L | $0E down (spawn) | (0,-1) (0,0) (0,1) | line |
| L | $0F left | (-1,-1) (-1,0) (0,0) | corner |
| L | $10 up | (-1,1) (0,-1) (0,0) | not joined |
| I | $11, $12 (spawn) | -2, -1, 0 | line, off center |

"Not joined" means one square touches the others only at a corner.

**Evidence:** `tetris_original.nes` has `A9 04 85 A9 BD 9C 8A` at `$8A47`
(`generalCounter2` is `$A9`, `orientationTable` is `$8A9C`) and
`A0 00 A9 04 85 AA` at `$94A4` and `$99D7` (`generalCounter3` is `$AA`).
All 32KB of PRG is always mapped, so no other bank can match.

**Revisions:** US only. Other revisions were not checked.

**Caveats:**

- Not tested.
- The shapes change as a piece turns. T, J and L turn between a line and
  a corner, and some orientations of J, S and L are not joined.
- The next-piece box and the statistics on the left still show 4
  squares. The next-piece box uses its own sprite data, and the
  statistics are background tiles.
- No piece is more than 3 squares tall, so a Tetris (4 lines at once) is
  not possible.
- With "I pieces have only 3 squares" also on, the I piece has only 2
  squares.

## B-type height 5 starts with 15 rows of garbage

**Codes:** `YEVAGNGE` + `ZLNEUAAI` (use both)
**Raw form:** `87E4:0F:0C` + `887B:32:50`
(address:new-byte:compare-byte, one 32KB bank)
**Optional, 16 rows instead:** `AOVAGNGA` + `AZNEUAAS` (`87E4:10:0C` + `887B:28:50`)
**Optional, 17 rows instead:** `POVAGNGA` + `TPNEUAAS` (`87E4:11:0C` + `887B:1E:50`)
**Recorded:** 2026-11-01 22:38:25

**Mechanism:** At the start of a B-type game, `initPlayfieldForTypeB`
always fills the bottom 12 rows with random garbage:

```
initPlayfieldForTypeB:
        lda     #$0C
        sta     generalCounter  ; decrements
```

Then it blanks the playfield from the top down to the cell given by the
height in `typeBBlankInitCountByHeightTable`:

```
typeBBlankInitCountByHeightTable:
        .byte   $C8,$AA,$96,$78,$64,$50
```

Height 5 is `$50` (cell 80, the start of row 8), so it keeps all 12 rows.
The first code makes 15 rows of garbage (rows 5 to 19). The second
changes the height 5 entry to `$32` (cell 50, the start of row 5), so
all 15 rows are kept. The top 5 rows are left empty. Pieces appear in
rows 0 and 1, so there are 3 rows below them to move a piece sideways
before it reaches the garbage.

The optional codes make 16 rows (`$10` and `$28`) and leave 4 empty
rows. That leaves only 2 rows below the spawn rows to move a piece
sideways.

The 17-row codes use `$11` and `$1E` (cell 30, the start of row 3) and
leave 3 empty rows. Pieces appear in rows 0 and 1, so there is only 1
row below them to move a piece sideways.

**Evidence:** `tetris_original.nes` has `A9 0C 85 A8 A5 A8 F0` at `$87E3`
(`generalCounter` is `$A8`) and `C8 AA 96 78 64 50` at `$8876`. All 32KB of
PRG is always mapped, so no other bank can match.

**Revisions:** US only. Other revisions were not checked.

**Caveats:**

- Not tested.
- Only height 5 changes. Heights 0 to 4 blank the extra rows, as before.
- As in the normal game, the blanking stops one cell into the first
  garbage row, so the left cell of the top garbage row is always empty.
- Each garbage row is about half full, so the stack is not solid. Pieces
  can still fall into open cells.

## Most pieces have 5 squares in strange arrangements

**Codes:** `IAKEEZGA` + `IEXPYGGA` + `IESOXPGA` (use all 3)
**Raw form:** `8A48:05:04` + `94A7:05:04` + `99DA:05:04`
(address:new-byte:compare-byte, one 32KB bank)
**Recorded:** 2026-11-01 22:48:01

Tags: `demo-ready` `play-start` `video-ready` `challenge`

**Mechanism:** These are the same three loop counts as in "All pieces
have only 3 squares": the loops that draw the falling piece
(`stageSpriteForCurrentPiece`), test a move (`isPositionValid`) and lock
it (`playState_lockTetrimino`). Here they change from 4 to 5. Each
orientation in `orientationTable` is 12 bytes (4 squares of 3 bytes),
so the 5th square read is the first square of the next orientation in
the table, with that orientation's tile.

The 5th square for each orientation (y, x offset):

| Piece | Orientation | 5th square | Shape |
|---|---|---|---|
| T | $00 up | (-1,0) | T, the square is already there |
| T | $01 right | (0,-1) | plus |
| T | $02 down (spawn) | (-1,0) | plus |
| T | $03 left | (-1,0) | T, the square is already there |
| J | $04 left | (-1,-1) | U |
| J | $05 up | (-1,0) | P |
| J | $06 right | (0,-1) | F |
| J | $07 down (spawn) | (0,-1) | J, the square is already there |
| Z | $08 (spawn) | (-1,1) | not joined |
| Z | $09 | (0,-1) | F |
| O | $0A (spawn) | (0,0) | O, the square is already there |
| S | $0B (spawn) | (-1,0) | F |
| S | $0C | (-1,0) | S, the square is already there |
| L | $0D right | (0,-1) | F |
| L | $0E down (spawn) | (-1,-1) | long T |
| L | $0F left | (-1,1) | long T |
| L | $10 up | (-2,0) | not joined |
| I | $11 vertical | (0,-2) | not joined |
| I | $12 horizontal (spawn) | (0,0) | I, the square is already there |

"Not joined" means the 5th square touches the piece only at a corner.

**Evidence:** `tetris_original.nes` has `A9 04 85 A9 BD 9C 8A` at `$8A47`
(`generalCounter2` is `$A9`, `orientationTable` is `$8A9C`) and
`A0 00 A9 04 85 AA` at `$94A4` and `$99D7` (`generalCounter3` is `$AA`).
The 4 bytes after the table, at `$8B80`, are `00 FF 00 00`. All 32KB of
PRG is always mapped, so no other bank can match.

**Revisions:** US only. Other revisions were not checked.

**Caveats:**

- Not tested.
- Not every orientation gets a new square, so the O piece always has 4
  and the other pieces change between 4 and 5 squares as they turn.
- Where the 5th square sits on a square that is already there, it is
  locked last, so that square can change color when the piece locks
  (T left, J down, O and S vertical).
- The horizontal I piece reads its 5th square from the code after the
  table, with tile `$FF`. `$FF` is a blank tile, so when the piece locks,
  its third square disappears. The game treats tiles from `tileEmpty`
  (`$EF`) up as empty when it tests a move, so other pieces can pass
  through that cell. When it checks for full rows it only looks for
  `$EF`, so the cell still counts as filled.
- The T, Z, S and L pieces now appear with a square one row above the
  playfield. It is drawn over the top border. If a piece locks with a
  square above the top row, the square is kept in unused RAM after the
  playfield (`$04C8`-`$04FF`) and is not shown, but later pieces can
  still hit it. This can end the game early.
- The next-piece box and the statistics on the left still show 4
  squares.
- With "All pieces have only 3 squares in strange arrangements" or "I
  pieces have only 3 squares" also on, the codes conflict.

## Type B averages 1 block per garbage row

**Codes:** `AAEAVAYA` + `LYUENENV` (use both)
**Raw form:** `8806:00:07` + `883F:7B:EF`
(address:new-byte:compare-byte, one 32KB bank)
**Recorded:** 2026-11-01 22:57:41

Tags: `demo-ready` `play-start` `curated`

**Mechanism:** `typeBGarbageInRow` fills each of the 10 cells of a garbage
row from `rngTable`, using the low 3 bits of `rng_seed` as the index:

```
        lda     rng_seed
        and     #$07
        tay
        lda     rngTable,y
```

```
rngTable:
        .byte   tileEmpty,tile1,tileEmpty,tile2
        .byte   tile3,tile3,tileEmpty,tileEmpty
```

Half of the entries are blocks, so each cell has a 50% chance. Then
`typeBGuaranteeBlank` writes `tileEmpty` into one random column, so no row
starts full. That makes 4.5 blocks per row on average.

The first code changes `and #$07` to `and #$00`, so every cell reads
entry 0 (`tileEmpty`) and the row starts empty. The second changes the
`lda #tileEmpty` in `typeBGuaranteeBlank` to `lda #tile1`, so the random
column gets a block instead of a hole. Every garbage row has exactly 1
block, in a random column.

**Evidence:** `tetris_original.nes` has `A5 17 29 07 A8 B9 7C 88` at
`$8803` (`rng_seed` is `$17`, `rngTable` is `$887C`),
`A9 EF 99 00 04` at `$883E` (`tileEmpty` is `$EF`, `playfield` is
`$0400`) and `EF 7B EF 7C 7D 7D EF EF` at `$887C`. All 32KB of PRG is
always mapped, so no other bank can match.

**Revisions:** US only. Other revisions were not checked.

**Caveats:**

- Not tested.
- All the blocks are `tile1`, so they are all one color.
- The level and the height do not change how full the rows are. The
  height only sets how many of the 12 rows are kept.
- As in the normal game, the height blanking stops one cell into the top
  garbage row, so the left cell of that row is always empty. If its one
  block is there, the top row has no blocks.

## Type B averages 9 blocks per garbage row

**Codes:** `PAEAVAYA` + `YYEEOEGN` (use both)
**Raw form:** `8806:01:07` + `8809:7F:7C`
(address:new-byte:compare-byte, one 32KB bank)
**Recorded:** 2026-11-01 22:57:41

Tags: `demo-ready` `play-start`

**Mechanism:** See "Type B averages 1 block per garbage row" for how a
garbage row is filled. The first code changes `and #$07` to `and #$01`,
so the index is 0 or 1. The second moves the read from `rngTable` to
`rngTable+3`:

```
        lda     rngTable,y      ; now rngTable+3,y
```

Entries 3 and 4 are `tile2` and `tile3`, so every cell gets a block, in
one of two colors. `typeBGuaranteeBlank` still empties one random column,
so every garbage row has exactly 9 blocks and one hole.

**Evidence:** `tetris_original.nes` has `A5 17 29 07 A8 B9 7C 88` at
`$8803` (`rng_seed` is `$17`, `rngTable` is `$887C`) and
`EF 7B EF 7C 7D 7D EF EF` at `$887C`. All 32KB of PRG is always mapped,
so no other bank can match.

**Revisions:** US only. Other revisions were not checked.

**Caveats:**

- Not tested.
- As in the normal game, the left cell of the top garbage row is always
  empty, so that row can have 8 blocks and two holes.
- At height 5 there are 12 rows, each with one hole in a random column.
  You clear a row by filling its one hole.

## Type B garbage rows are full and usually can't be cleared

**Codes:** `PAEAVAYA` + `YYEEOEGN` + `OLKAEEOO` (use all 3)
**Raw form:** `8806:01:07` + `8809:7F:7C` + `8840:B9:99`
(address:new-byte:compare-byte, one 32KB bank)
**Recorded:** 2026-11-01 22:57:41

Tags: `demo-ready` `play-start`

**Mechanism:** The first two codes are "Type B averages 9 blocks per
garbage row": every cell gets `tile2` or `tile3`. The third turns off
the write in `typeBGuaranteeBlank`:

```
typeBGuaranteeBlank:
...
        lda     #tileEmpty
        sta     playfield,y     ; now lda playfield,y
```

`sta` (`$99`) becomes `lda` (`$B9`) with the same address, so the random
column is read and not changed. The rest of the routine (picking the
column, waiting for the next frame) still runs.

**Evidence:** `tetris_original.nes` has `A5 17 29 07 A8 B9 7C 88` at
`$8803` and `A9 EF 99 00 04` at `$883E` (`tileEmpty` is `$EF`, `playfield`
is `$0400`). All 32KB of PRG is always mapped, so no other bank can
match.

**Revisions:** US only. Other revisions were not checked.

**Caveats:**

- Tested in play by the user. A full row cleared from a nearby block
  only once, so the rows usually can't be cleared.
- As in the normal game, the height blanking stops one cell into the top
  garbage row, so the left cell of that row is still empty. All the rows
  below it are full.
- The game only looks for full rows near a piece when it locks (from 2
  rows above the piece's center down to 1 row below it). The full rows
  stay until a piece locks near them. They are then cleared and count
  toward the 25 lines, but a piece can rarely lock that close.

## Auto-repeat rate is 1 column per 3 frames

**Codes:** `IEKESOZE`
**Raw form:** `89CD:0D:0A`
(address:new-byte:compare-byte, one 32KB bank)
**Recorded:** 2026-11-01 23:23:21

**Mechanism:** This was the optional code of "Shorter auto-shift delay".
When a held LEFT or RIGHT reaches `DAS_RESET` (`$10`), `shift_tetrimino`
moves the piece and sets `autorepeatX` to `DAS_DELAY` (`$0A`). Six more
frames then pass before the piece moves again:

```
        inc     autorepeatX
        lda     autorepeatX
        cmp     #DAS_RESET
        bmi     @ret
        lda     #DAS_DELAY
        sta     autorepeatX
        jmp     @buttonHeldDown
```

The code changes the `lda #DAS_DELAY` to `lda #$0D`. After each repeat,
`autorepeatX` is 3 steps short of `DAS_RESET`, so the piece moves again 3
frames later. A held button moves the piece one column every 3 frames
(20 columns a second, instead of 10). The rate in frames is `$10` minus
the new byte. "Auto-repeat rate is 1 column per frame" uses `$0F`.

**Evidence:** `tetris_original.nes` has
`E6 46 A5 46 C9 10 30 3D A9 0A 85 46 4C D7 89` at `$89C4` (`autorepeatX`
is `$46`). `autorepeatX` is only used in `shift_tetrimino`. All 32KB of
PRG is always mapped, so no other bank can match.

**Revisions:** US only. Other revisions were not checked.

**Caveats:**

- Not tested.
- It changes a different byte from "Shorter auto-shift delay" and works
  with it or without it. Without it, the first repeat still waits 16
  frames.
- Works in 1-player, 2-player and the demo. The demo was recorded with
  the normal timing, so pieces in the demo may end up in other columns.
- Holding DOWN still stops sideways movement, as before.
- "Auto-repeat left/right disabled" does nothing
  with this code on, since it turns off the auto-repeat this code speeds
  up.
- It uses the same byte as the other auto-repeat rate entries, so use
  only one of them.

## Auto-repeat rate is 1 column per frame

**Codes:** `YEKESOZE`
**Raw form:** `89CD:0F:0A`
(address:new-byte:compare-byte, one 32KB bank)
**Recorded:** 2026-11-01 23:23:21

**Mechanism:** This was the optional code of "Shorter auto-shift delay".
When a held LEFT or RIGHT reaches `DAS_RESET` (`$10`), `shift_tetrimino`
moves the piece and sets `autorepeatX` to `DAS_DELAY` (`$0A`). Six more
frames then pass before the piece moves again:

```
        inc     autorepeatX
        lda     autorepeatX
        cmp     #DAS_RESET
        bmi     @ret
        lda     #DAS_DELAY
        sta     autorepeatX
        jmp     @buttonHeldDown
```

The code changes the `lda #DAS_DELAY` to `lda #$0F`. After each repeat,
`autorepeatX` is one step short of `DAS_RESET`, so the piece moves again
on the next frame. A held button moves the piece one column every frame
(60 columns a second, instead of 10). No tapping technique can match
this. For slower rates, see "Auto-repeat rate is 1 column per 3 frames".

**Evidence:** `tetris_original.nes` has
`E6 46 A5 46 C9 10 30 3D A9 0A 85 46 4C D7 89` at `$89C4` (`autorepeatX`
is `$46`). `autorepeatX` is only used in `shift_tetrimino`. All 32KB of
PRG is always mapped, so no other bank can match.

**Revisions:** US only. Other revisions were not checked.

**Caveats:**

- Not tested.
- It changes a different byte from "Shorter auto-shift delay" and works
  with it or without it. Without it, the first repeat still waits 16
  frames.
- Works in 1-player, 2-player and the demo. The demo was recorded with
  the normal timing, so pieces in the demo may end up in other columns.
- Holding DOWN still stops sideways movement, as before.
- "Auto-repeat left/right disabled" does nothing
  with this code on, since it turns off the auto-repeat this code speeds
  up.
- It uses the same byte as the other auto-repeat rate entries, so use
  only one of them.

## Half of Level 0 speed on every level

**Codes:** `ATNAKPPA` + `AANEEOIP` (use both)
**Raw form:** `8974:60:01` + `8978:00:1D`
(address:new-byte:compare-byte, one 32KB bank)
**Recorded:** 2026-11-01 23:41:11

Tags: `demo-ready` `play-start`

**Mechanism:** `drop_tetrimino` looks up how many frames a piece waits
before it falls one row. Levels 0-28 read `framesPerDropTable` (48 frames
at level 0, down to 2 frames from level 19). Level 29 and up skip the
table and use 1 frame:

```
@lookupDropSpeed:
        lda     #$01
        ldx     levelNumber
        cpx     #$1D
        bcs     @noTableLookup
        lda     framesPerDropTable,x
@noTableLookup:
        sta     dropSpeed
        lda     fallTimer
        cmp     dropSpeed
        bpl     @drop
```

The first code changes `lda #$01` to `lda #$60` (96). The second changes
`cpx #$1D` to `cpx #$00`, so `bcs @noTableLookup` is always taken and the
table is never read. Every level now uses 96 frames per row, half the level 0
speed.

"Level 0 speed on every level" uses `lda #$30` (48 frames per row) in
place of `#$60`. `fallTimer` is compared with `bpl`, so values up to
`$7F` work.

**Evidence:** `tetris_original.nes` has
`A9 01 A6 44 E0 1D B0 03 BD 8E 89 85 AF A5 45 C5 AF 10 D2` at `$8973`
(`framesPerDropTable` is `$898E`, `dropSpeed` is `$AF`, `fallTimer` is
`$45`). `dropSpeed` is only set here and cleared at the start of a game.
All 32KB of PRG is always mapped, so no other bank can match.

**Revisions:** US only. Other revisions were not checked.

**Caveats:**

- Not tested.
- The level number, the level colors and the score
  still change as normal. Only the fall speed stays the same.
- Holding DOWN still drops the piece fast, and still scores the
  soft-drop points.
- Works in 2-player and the demo. The demo pieces fall more slowly, so
  the demo plays out differently.
- It changes the same bytes as "Level 0 speed on every level", so use
  only one of the two.

## Each instrument plays as the next one

**Codes:** `ZANVGYAA`
**Raw form:** `E77C:02:00`
(address:new-byte:compare-byte, one 32KB bank)
**Recorded:** 2026-11-02 00:08:05

Tags: `interesting` `play-start`

**Mechanism:** This was the optional code of "All music plays with one
instrument". The `$9F` music command sets `musicChanControl`, and its low
5 bits pick an instrument: a volume envelope from
`musicChanVolControlTable`. Each frame, `updateMusicFrame_setChanVol`
turns the instrument number into a table offset in Y and reads the
envelope's address:

```
        ldy     #$00
@controlMinus1Times2_storeToY:
        dec     AUDIOTMP2
        beq     @loadFromTable
        iny
        iny
        bne     @controlMinus1Times2_storeToY
@loadFromTable:
        lda     musicChanVolControlTable,y
        sta     AUDIOTMP3
        lda     musicChanVolControlTable+1,y
        sta     AUDIOTMP4
```

The code changes `ldy #$00` to `ldy #$02`, so each instrument number reads
the next entry. The songs use instruments up to `$15`, and the table has
22 entries, so the last one used still reads a real entry.

**Evidence:** `tetris_original.nes` has
`A0 00 C6 E1 F0 04 C8 C8 D0 F8 B9 4A EA 85 E2 B9 4B EA 85 E3` at `$E77B`
(`musicChanVolControlTable` is `$EA4A`, `AUDIOTMP2` is `$E1`). All 32KB
of PRG is always mapped, so no other bank can match.

**Revisions:** US only. Other revisions were not checked.

**Caveats:**

- Not tested.
- Only the two square channels use these envelopes. The triangle and
  noise (drum) channels sound as before.
- Notes set with instrument 0 (`$9F,$00`) skip the envelope and are not
  changed.
- The duty cycle (tone colour) still comes from each `$9F` command. Only
  the volume shape changes.
- Sound effects are not changed.
- "All music plays with one instrument" cancels this code, since its
  codes make the table reads ignore Y. Use only one of the two.

## The level goes up with every line

**Codes:** `AEKOKUTZ` + `AEVOSUIA` (use both)
**Raw form:** `9BCC:00:2E` + `9BED:00:0D`
(address:new-byte:compare-byte, one 32KB bank)
**Recorded:** 2026-11-02 00:46:34

Tags: `demo-ready` `play-start` `challenge`

**Mechanism:** In an A-type game, `incrementLines` runs once for each line
a piece clears. The game goes up a level only when the lines digit rolls
over to 0 and the level is lower than `lines / 10`:

```
incrementLines:
        ...
L9BC7:  lda     lines
        and     #$0F
        bne     L9BFB
        jmp     L9BD0

L9BD0:  ...
        lda     levelNumber
        cmp     generalCounter
        bpl     L9BFB
@incrementLevel:
        inc     levelNumber
        lda     #$06
        sta     soundEffectSlot1Init
        lda     outOfDateRenderFlags
        ora     #RENDER_LEVEL
        sta     outOfDateRenderFlags
L9BFB:  dex
        bne     incrementLines
```

The two codes change the offset of `bne L9BFB` and the offset of
`bpl L9BFB` to 0, so neither branch skips anything. Every line now falls
through to `@incrementLevel`, which adds 1 to the level, plays the
level-up sound and redraws the level. A 4-line clear adds 4 levels.

**Evidence:** `tetris_original.nes` has
`A5 50 29 0F D0 2E 4C D0 9B` at `$9BC7` and
`A5 44 C5 A8 10 0D E6 44 A9 06` at `$9BE8` (`lines` is `$50`,
`levelNumber` is `$44` while the player's data is copied to `$40`). Both
branches end at `L9BFB` (`$9BFB`). All 32KB of PRG is always mapped, so
no other bank can match.

**Revisions:** US only. The NWC version has different code here.

**Caveats:**

- Not tested.
- Only A-type changes. B-type counts the lines down in its own code and
  does not use `incrementLines`.
- Level 29 (1 frame per row) comes after 29 lines from level 0.
- Past level 29 the level display shows odd numbers and tiles, and past
  level 137 the colors glitch, as in the real game. The level goes back
  to 0 after 255.
- Both codes are needed. With only one of them, the level still goes up
  only when the lines digit is 0 and the level is behind.
- "The level goes up with every piece" also changes the level, so using
  both adds both.

## Skip the title screen

**Codes:** `GGVEAOYK`
**Raw form:** `8168:4C:4F`
(address:new-byte:compare-byte, one 32KB bank)
**Recorded:** 2026-11-03 00:03:43

**Mechanism:** This is the Tetris version of the "skip the title screen"
codes in the SNES files (ALTTP, Super Metroid, EarthBound, FF4, SMW). The
main loop runs the handler for `gameMode` from a jump table:

```
branchOnGameMode:
        lda     gameMode
        jsr     switch_s_plus_2a
        .addr   gameMode_legalScreen
        .addr   gameMode_titleScreen
        .addr   gameMode_gameTypeMenu
        ...
```

The code changes the low byte of the `gameMode_titleScreen` entry
(`$824F`) so it points to `$824C` instead. That is the end of
`gameMode_legalScreen`:

```
@continueToNextScreen:
        inc     gameMode
        rts
```

So mode 1 (title) now only adds 1 to `gameMode`, and the game type menu
(mode 2) comes up right after the legal screen. The title screen is never
drawn and its START sound doesn't play.

**Evidence:** `tetris_original.nes` has
`A5 C0 20 82 AC 00 82 4F 82 D1 82` at `$8161` (`gameMode` is `$C0`,
`switch_s_plus_2a` is `$AC82`; the table entries are `$8200`, `$824F`,
`$82D1`) and `E6 C0 60` at `$824C`. All 32KB of PRG is always mapped,
so no other bank can match.

**Revisions:** US only. The NWC version has different code here.

**Caveats:**

- Not tested.
- The demo never plays. Only the title screen starts it, after waiting
  for START for a while.
- Pressing START during the demo, or the end of the demo, now goes to the
  game type menu instead of the title screen.
- The legal screen still shows. Use "The legal screen goes by in a
  moment" too to get to the menu in about a second.
- The title screen also clears `displayNextPiece` and `$D0` (the demo
  recording flag). Both start at 0 after reset, so this only matters if
  the next piece was hidden with SELECT in an earlier game. It then stays
  hidden until SELECT is pressed again.

## The game runs at half speed

**Codes:** `VTKAAASA` + `VYUZUZAE` + `ALUXOZEY` + `LLSPXIEG` + `NYSPKIIE` + `LASPVIGA` (use all 6)
**Raw form:** `8040:E6:85` + `AA33:FE:00` + `AA39:30:F0` + `9D52:33:C0` + `9D54:FF:05` + `9D56:03:04`
(address:new-byte:compare-byte, one 32KB bank)
**Recorded:** 2026-11-03 00:08:06

**Mechanism:** This is the Tetris version of the half-speed codes
(Metroid, SMB1, Zelda). The main loop and the menus wait for the next
frame with `updateAudioWaitForNmiAndResetOamStaging`:

```
        lda     #$00
        sta     verticalBlankingInterval
        nop
@checkForNmi:
        lda     verticalBlankingInterval
        beq     @checkForNmi
```

The NMI sets the flag with `lda #$01` / `sta verticalBlankingInterval`.
The first code changes that `sta` to `inc`, so each NMI adds 1. The
next two codes make the wait start at `$FE` and loop with `bmi` instead
of `beq`. The flag goes `$FE`, `$FF`, `$00`, so the wait now lasts 2
frames. The other wait (`updateAudioAndWaitForNmi`) still starts at 0
and ends after 1 frame.

Just slowing the loop would lose half of the new button presses. The
NMI reads the controller every frame, and `newlyPressedButtons_player1`
only holds a press for one frame. So the last three codes change the
start of `pollControllerButtons`:

```
pollControllerButtons:
        lda     gameMode
        cmp     #$05
        beq     @demoGameMode
        jsr     pollController
        rts
```

to `lda verticalBlankingInterval` / `cmp #$FF` / `beq` to the `rts`. In
the first frame of the 2-frame wait the flag is `$FF`, so the
controller isn't read and the last press stays. In the second frame it
is read. So the game sees each new press exactly once.

**Evidence:** `tetris_original.nes` has `A9 01 85 33 20 51 9D` at `$803E`
(`verticalBlankingInterval` is `$33`, `pollControllerButtons` is
`$9D51`), `A9 00 85 33 EA A5 33 F0 FC` at `$AA32`, and
`A5 C0 C9 05 F0 04 20 9D AB 60` at `$9D51` (`gameMode` is `$C0`; the
`rts` is at `$9D5A`). All 32KB of PRG is always mapped, so no other bank
can match.

**Revisions:** US only. Other revisions were not checked.

**Caveats:**

- Not tested.
- 6 codes, more than a Game Genie holds.
- Play, the menus, the rocket and the music all run at half speed.
  Sound pitch doesn't change. Waits timed with `frameCounter` or
  `sleep_for_a_vblanks` stay at normal speed, so the title screen still
  starts the demo after the same time, and the legal screen's first wait
  doesn't change (the part that START can skip is twice as long).
- The demo no longer plays back its recorded input. The controller is
  read instead, so the demo pieces fall where the player puts them (or
  straight down), and the demo goes on until it tops out or START is
  pressed.
- The first 3 codes alone also give half speed, but about half of the
  presses of A, B, START and SELECT are missed, and a tap of left or
  right is sometimes missed.
