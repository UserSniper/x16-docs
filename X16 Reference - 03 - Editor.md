
# Chapter 3: Editor

The X16 has a built-in screen editor that is backwards-compatible with the C64, but has many new features.

## Modes

The editor's default mode is 80x60 text mode. The following text mode resolutions are supported:

| Mode | Description |
|------|-------------|
| $00  | 80x60 text  |
| $01  | 80x30 text  |
| $02  | 40x60 text  |
| $03  | 40x30 text  |
| $04  | 40x15 text  |
| $05  | 20x30 text  |
| $06  | 20x15 text  |
| $07  | 22x23 text  |
| $08  | 64x50 text  |
| $09  | 64x25 text  |
| $0A  | 32x50 text  |
| $0B  | 32x25 text  |
| $80  | 320x240@256c<br/>40x30 text |

Mode $80 contains two layers: a text layer on top of a graphics screen. In this mode, text color 0 is translucent instead of black.

To switch modes, use the BASIC statement `SCREEN` or the KERNAL API `screen_mode`. In the BASIC editor, the F4 key toggles between modes 0 (80x60) and 3 (40x30).

<!-- For PDF formatting -->
<div class="page-break"></div>

## ISO Mode

In addition to PETSCII, the X16 also supports the ISO-8859-15 character encoding. In ISO-8859-15 mode ("ISO mode"):

* The character set is switched from Commodore-style (with PETSCII drawing characters) to a new ASCII/ISO-8859-15 compatible set, which covers most Western European writing systems.
* The encoding (`CHR$()` in BASIC and `BSOUT` in machine language) now complies with ASCII and ISO-8859-15.
* The keyboard driver will return ASCII/ISO-8859-15 codes.

This is the encoding:

|        |x0 |x1 |x2 |x3 |x4 |x5 |x6 |x7 |x8 |x9 |xA |xB |xC |xD |xE |xF |
|--------|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| **0x** |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| **1x** |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| **2x** |   | ! | " | # | $ | % | & | ' | ( | ) | * | + | , | - | . | / |
| **3x** | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | : | ; | < | = | > | ? |
| **4x** | @ | A | B | C | D | E | F | G | H | I | J | K | L | M | N | O |
| **5x** | P | Q | R | S | T | U | V | W | X | Y | Z | [ | \ | ] | ^ | _ |
| **6x** | ` | a | b | c | d | e | f | g | h | i | j | k | l | m | n | o |
| **7x** | p | q | r | s | t | u | v | w | x | y | z | { | \| | } | ~ |   |
| **8x** |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| **9x** |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| **Ax** |   | ¡ | ¢ | £ | € | ¥ | Š | § | š | © | ª | « | ¬ | 🦋 | ® | ¯ |
| **Bx** | ° | ± | ² | ³ | Ž | µ | ¶ | · | ž | ¹ | º | » | Œ | œ | Ÿ | ¿ |
| **Cx** | À | Á | Â | Ã | Ä | Å | Æ | Ç | È | É | Ê | Ë | Ì | Í | Î | Ï |
| **Dx** | Ð | Ñ | Ò | Ó | Ô | Õ | Ö | × | Ø | Ù | Ú | Û | Ü | Ý | Þ | ß |
| **Ex** | à | á | â | ã | ä | å | æ | ç | è | é | ê | ë | ì | í | î | ï |
| **Fx** | ð | ñ | ò | ó | ô | õ | ö | ÷ | ø | ù | ú | û | ü | ý | þ | ÿ |

* The non-printable areas $00-$1F and $80-$9F in the character set are filled with inverted variants of the codes $40-$5F and $60-$7F, respectively.
* The code $AD is a non-printable soft hyphen in ISO-8859-15. The ROM character set contains the Commander X16 logo at this location.

ISO mode can be enabled and disabled using two new control codes:

* `CHR$($0F)`: enable ISO mode
* `CHR$($8F)`: enable PETSCII mode (default)

You can also enable ISO mode in direct mode by pressing Ctrl+`O`.

**Important:** In ISO mode, BASIC keywords need to be written in upper case, that is, they have to be entered with the Shift key down, and abbreviating keywords is no longer possible.

## Background Color

In regular BASIC text mode, the video controller supports 16 foreground colors and 16 background colors for each character on the screen.

The new "swap fg/bg color" code is useful to change the background color of the cursor, like this:

```basic
PRINT CHR$(1);   : REM SWAP FG/BG
PRINT CHR$($1C); : REM SET FG COLOR TO RED
PRINT CHR$(1);   : REM SWAP FG/BG
```

The new BASIC instruction `COLOR` makes this easier, but the trick above can also be used from machine code programs.

To set the background color of the complete screen, it just has to be cleared after setting the color:

```basic
PRINT CHR$(147);
```

## Scrolling

The C64 editor could only scroll the screen up (when overflowing the last line or printing or entering DOWN on the last line). The X16 editor scrolls both ways: When the cursor is on the first line and UP is printed or entered, the screen contents scroll down by a line.

## New Control Characters

This is the set of all supported PETSCII control characters. Entries in bold indicate new codes compared to the C64:

If there are two meanings listed, the first indicates input (a keypress) and the second indicates output.

| Code  |                            |                           | Code  |
|-------|----------------------------|---------------------------|-------|
| $00  | NULL                       | **VERBATIM MODE**         | $80  |
| $01  | **SWAP COLORS**            | COLOR: ORANGE             | $81  |
| $02  | **PAGE DOWN**              | **PAGE UP**               | $82  |
| $03  | STOP                       | RUN                       | $83  |
| $04  | **END**                    | **HELP**                  | $84  |
| $05  | COLOR: WHITE               | F1                        | $85  |
| $06  | **MENU**                   | F3                        | $86  |
| $07  | **BELL**                   | F5                        | $87  |
| $08  | DISALLOW CHARSET SW (SHIFT+ALT) | F7                        | $88  |
| $09  | **TAB** / ALLOW CHARSET SW | F2                        | $89  |
| $0A  | **LF**                     | F4                        | $8A  |
| $0B  | -                          | F6                        | $8B  |
| $0C  | -                          | F8                        | $8C  |
| $0D  | RETURN                     | SHIFTED RETURN            | $8D  |
| $0E  | CHARSET: LOWER/UPPER       | CHARSET: UPPER/PETSCII    | $8E  |
| $0F  | **CHARSET: ISO ON**        | **CHARSET: ISO OFF**      | $8F  |
| $10  | **F9**                     | COLOR: BLACK              | $90  |
| $11  | CURSOR: DOWN               | CURSOR: UP                | $91  |
| $12  | REVERSE ON                 | REVERSE OFF               | $92  |
| $13  | HOME                       | CLEAR                     | $93  |
| $14  | DEL (PS/2 BACKSPACE)       | INSERT                    | $94  |
| $15  | **F10**                    | COLOR: BROWN              | $95  |
| $16  | **F11**                    | COLOR: LIGHT RED          | $96  |
| $17  | **F12**                    | COLOR: DARK GRAY          | $97  |
| $18  | **SHIFT+TAB**              | COLOR: MIDDLE GRAY        | $98  |
| $19  | **FWD DEL (PS/2 DEL)**     | COLOR: LIGHT GREEN        | $99  |
| $1A  | -                          | COLOR: LIGHT BLUE         | $9A  |
| $1B  | ESC                        | COLOR: LIGHT GRAY         | $9B  |
| $1C  | COLOR: RED                 | COLOR: PURPLE             | $9C  |
| $1D  | CURSOR: RIGHT              | CURSOR: LEFT              | $9D  |
| $1E  | COLOR: GREEN               | COLOR: YELLOW             | $9E  |
| $1F  | COLOR: BLUE                | COLOR: CYAN               | $9F  |

**Notes:**

* $01: SWAP COLORS swaps the foreground and background colors in text mode
* \$07/\$09/\$0A/\$18/\$1B: have been added for ASCII compatibility. *[\$0A/\$18/\$1B do not have any effect on output. Outputs of \$08/\$09 have their traditional C64 effect]*
* \$80: VERBATIM MODE prints the next character (only!) as a glyph without interpretation. This is similar to quote mode, but also includes codes CR (\$0D) and DEL (\$14).
* F9-F12: these codes match the C65 additions
* $84: This code is generated when pressing SHIFT+END.
* Additionally, the codes \$04/\$06/\$0B/\$0C are interpreted when printing in graphics mode using `GRAPH_put_char`.

<!-- For PDF formatting -->
<div class="page-break"></div>

## Keyboard Layouts

The editor supports multiple keyboard layouts.

## Default Layout

On boot, the US layout (`ABC/X16`) is active:

* In PETSCII mode, it matches the US layout where possible, and can reach all PETSCII symbols.
* In ISO mode, it matches the Macintosh US keyboard and can reach all ISO-8859-15 characters. Some characters are reachable through key combinations:

| Key               | Result |
|-------------------|--------|
| Alt+`1`           | ¡      |
| Alt+`3`           | £      |
| Alt+`4`           | ¢      |
| Alt+`5`           | §      |
| Alt+`7`           | ¶      |
| Alt+`9`           | ª      |
| Alt+`0`           | º      |
| Alt+`q`           | œ      |
| Alt+`r`           | ®      |
| Alt+`t`           | Þ      |
| Alt+`y`           | ¥      |
| Alt+`o`           | ø      |
| Alt+`\`           | «      |
| Alt+`s`           | ß      |
| Alt+`d`           | ð      |
| Alt+`g`           | ©      |
| Alt+`l`           | ¬      |
| Alt+`'`           | æ      |
| Alt+`m`           | µ      |
| Alt+`/`           | ÷      |
| Shift+Alt+`2`     | €      |
| Shift+Alt+`8`     | °      |
| Shift+Alt+`9`     | ·      |
| Shift+Alt+`-`     | X16 logo |
| Shift+Alt+`=`     | ±      |
| Shift+Alt+`q`     | Œ      |
| Shift+Alt+`t`     | þ      |
| Shift+Alt+`\`     | »      |
| Shift+Alt+`a`     | ¹      |
| Shift+Alt+`d`     | Ð      |
| Shift+Alt+`k`     | X16 logo |
| Shift+Alt+`'`     | Æ      |
| Shift+Alt+`c`     | ³      |
| Shift+Alt+`b`     | ²      |
| Shift+Alt+`/`     | ¿      |

(The X16 logo is code point \\xad, SHY, soft-hyphen.)

The following combinations are dead keys:

* Alt+<tt>&#96;</tt>
* Alt+`6`
* Alt+`e`
* Alt+`u`
* Alt+`p`
* Alt+`a`
* Alt+`k`
* Alt+`;`
* Alt+`x`
* Alt+`c`
* Alt+`v`
* Alt+`n`
* Alt+`,`
* Alt+`.`
* Shift+Alt+`S`

They generate additional characters when combined with a second keypress:

| First <br/>Key      | Second <br/>Key | Result |
|--------------------|-----|---|
| Alt+<tt>&#96;</tt> | `a` | à |
| Alt+<tt>&#96;</tt> | `e` | è |
| Alt+<tt>&#96;</tt> | `i` | ì |
| Alt+<tt>&#96;</tt> | `o` | ò |
| Alt+<tt>&#96;</tt> | `u` | ù |
| Alt+<tt>&#96;</tt> | `A` | À |
| Alt+<tt>&#96;</tt> | `E` | È |
| Alt+<tt>&#96;</tt> | `I` | Ì |
| Alt+<tt>&#96;</tt> | `O` | Ò |
| Alt+<tt>&#96;</tt> | `U` | Ù |
| Alt+<tt>&#96;</tt> | `␣` | &#96; |
| Alt+`6`            | `e` | ê |
| Alt+`6`            | `u` | û |
| Alt+`6`            | `i` | î |
| Alt+`6`            | `o` | ô |
| Alt+`6`            | `a` | â |
| Alt+`6`            | `E` | Ê |
| Alt+`6`            | `U` | Û |
| Alt+`6`            | `I` | Î |
| Alt+`6`            | `O` | Ô |
| Alt+`6`            | `A` | Â |
| Alt+`e`            | `e` | é |
| Alt+`e`            | `y` | ý |
| Alt+`e`            | `u` | ú |
| Alt+`e`            | `i` | í |
| Alt+`e`            | `o` | ó |
| Alt+`e`            | `a` | á |
| Alt+`e`            | `E` | É |
| Alt+`e`            | `Y` | Ý |
| Alt+`e`            | `U` | Ú |
| Alt+`e`            | `I` | Í |
| Alt+`e`            | `O` | Ó |
| Alt+`e`            | `A` | Á |
| Alt+`u`            | `e` | ë |
| Alt+`u`            | `y` | ÿ |
| Alt+`u`            | `u` | ü |
| Alt+`u`            | `i` | ï |
| Alt+`u`            | `o` | ö |
| Alt+`u`            | `a` | ä |
| Alt+`u`            | `E` | Ë |
| Alt+`u`            | `Y` | Ÿ |
| Alt+`u`            | `U` | Ü |
| Alt+`u`            | `I` | Ï |
| Alt+`u`            | `O` | Ö |
| Alt+`u`            | `A` | Ä |
| Alt+`p`            | `␣` | , |
| Alt+`a`            | `␣` | ¯ |
| Alt+`k`            | `a` | å |
| Alt+`k`            | `A` | Å |
| Alt+`x`            | `␣` | . |
| Alt+`c`            | `c` | ç |
| Alt+`c`            | `C` | Ç |
| Alt+`v`            | `s` | š |
| Alt+`v`            | `z` | ž |
| Alt+`v`            | `S` | Š |
| Alt+`v`            | `Z` | Ž |
| Alt+`n`            | `o` | õ |
| Alt+`n`            | `a` | ã |
| Alt+`n`            | `n` | ñ |
| Alt+`n`            | `O` | Õ |
| Alt+`n`            | `A` | Ã |
| Alt+`n`            | `N` | Ñ |
| Shift+Alt+`s`      | `␣` | \\xa0 |
| Shift+Alt+`;`      | `=` | × |

"␣" denotes the space bar.

### ROM Keyboard Layouts

The following keyboard layouts are available from ROM. You can select one directly with the BASIC `KEYMAP` command, e.g. `KEYMAP"ABC/X16"`, or via the X16 Control Panel with the BASIC `MENU` command.

| Identifier  | Description                         | Code                                       |
|-------------|-------------------------------------|--------------------------------------------|
| `ABC/X16`   | ABC - Extended (X16)                | -                                          |
| `EN-US/INT` | United States - International       | [00020409](http://kbdlayout.info/00020409) |
| `EN-GB`     | United Kingdom                      | [00000809](http://kbdlayout.info/00000809) |
| `SV-SE`     | Swedish                             | [0000041D](http://kbdlayout.info/0000041D) |
| `DE-DE`     | German                              | [00000407](http://kbdlayout.info/00000407) |
| `DA-DK`     | Danish                              | [00000406](http://kbdlayout.info/00000406) |
| `IT-IT`     | Italian                             | [00000410](http://kbdlayout.info/00000410) |
| `PL-PL`     | Polish (Programmers)                | [00000415](http://kbdlayout.info/00000415) |
| `NB-NO`     | Norwegian                           | [00000414](http://kbdlayout.info/00000414) |
| `HU-HU`     | Hungarian                           | [0000040E](http://kbdlayout.info/0000040E) |
| `ES-ES`     | Spanish                             | [0000040A](http://kbdlayout.info/0000040A) |
| `FI-FI`     | Finnish                             | [0000040B](http://kbdlayout.info/0000040B) |
| `PT-BR`     | Portuguese (Brazil ABNT)            | [00000416](http://kbdlayout.info/00000416) |
| `CS-CZ`     | Czech                               | [00000405](http://kbdlayout.info/00000405) |
| `JA-JP`     | Japanese                            | [Custom IME](https://github.com/X16Community/x16-rom/pull/364) (since r49) |
| `FR-FR`     | French                              | [0000040C](http://kbdlayout.info/0000040C) |
| `DE-CH`     | Swiss German                        | [00000807](http://kbdlayout.info/00000807) |
| `EN-US/DVO` | United States - Dvorak              | [00010409](http://kbdlayout.info/00010409) |
| `ET-EE`     | Estonian                            | [00000425](http://kbdlayout.info/00000425) |
| `FR-BE`     | Belgian French                      | [0000080C](http://kbdlayout.info/0000080C) |
| `EN-CA`     | Canadian French                     | [00001009](http://kbdlayout.info/00001009) |
| `IS-IS`     | Icelandic                           | [0000040F](http://kbdlayout.info/0000040F) |
| `PT-PT`     | Portuguese                          | [00000816](http://kbdlayout.info/00000816) |
| `HR-HR`     | Croatian                            | [0000041A](http://kbdlayout.info/0000041A) |
| `SK-SK`     | Slovak                              | [0000041B](http://kbdlayout.info/0000041B) |
| `SL-SI`     | Slovenian                           | [00000424](http://kbdlayout.info/00000424) |
| `LV-LV`     | Latvian                             | [00000426](http://kbdlayout.info/00000426) |
| `LT-LT`     | Lithuanian IBM                      | [00000427](http://kbdlayout.info/00000427) |

All remaining keyboards are based on the respective Windows layouts. `EN-US/INT` differs from `EN-US` only in Alt/AltGr combinations and some dead keys.

The BASIC command `KEYMAP` allows activating a specific keyboard layout. It can be added to the auto-boot file, e.g.:

```basic
10 KEYMAP"NB-NO"
SAVE"AUTOBOOT.X16
```

### Loadable Keyboard Layouts

The tables for the active keyboard layout reside in banked RAM, at $A000 on bank 0:

| Addresses     | Description |
|---------------|-------------|
| \$A000-\$A07F | Table 0     |
| \$A080-\$A0FF | Table 1     |
| \$A100-\$A17F | Table 2     |
| \$A180-\$A1FF | Table 3     |
| \$A200-\$A27F | Table 4     |
| \$A280-\$A07F | Table 5     |
| \$A300-\$A37F | Table 6     |
| \$A380-\$A3FF | Table 7     |
| \$A400-\$A47F | Table 8     |
| \$A480-\$A4FF | Table 9     |
| \$A500-\$A57F | Table 10    |
| \$A580-\$A58F | big-endian bitfield:<br/>keynum codes for which Caps means Shift |
| \$A590-\$A66F | dead key table |
| \$A670-\$A67E | ASCIIZ identifier (e.g. "ABC/X16") |

The first byte of each of the 11 tables is the table ID which contains the encoding and the combination of modifiers that this table is for.

| Bit | Description        |
|-----|--------------------|
| 7   | 0: PETSCII, 1: ISO |
| 6-3 | always 0           |
| 2   | Ctrl               |
| 1   | Alt                |
| 0   | Shift              |

* AltGr is represented by Ctrl+Alt.
* ID $C6 represents Alt _or_ AltGr (ISO only)
* ID $C7 represents Shift+Alt _or_ Shift+AltGr (ISO only)
* Empty tables have an ID of $FF.

The identifier is followed by 127 output codes for the keynum inputs 1-127.

* Dead keys (i.e. keys that don't generate anything by themselves but modify the next key) have a code of 0 and are further described in the dead key table (ISO only)
* Keys that produce nothing have an entry of 0. (They can be distinguished from dead keys as they don't have an entry in the dead key table.)

The dead key table has one section for every dead key with the following layout:

| Byte | Description                                  |
|------|----------------------------------------------|
| 0    | dead key ID (PETSCII/ISO and Shift/Alt/Ctrl) |
| 1    | dead key scancode                            |
| 2    | full length of this table in bytes           |
| 3    | first additional key ISO code                |
| 4    | first effective key ISO code                 |
| 5    | second additional key ISO code               |
| 6    | second effective key ISO code                |
| ...  | ...                                          |
| n-1  | terminator 0xFF                              |

Custom layouts can be loaded from disk like this:

```BASIC
BLOAD"KEYMAP",8,0,$A000
```

Here is an example that activates a layout derived from "ABC/X16", with unshifted Y and Z swapped in PETSCII mode:

```BASIC
100 KEYMAP"ABC/X16"                               :REM START WITH DEFAULT LAYOUT
110 BANK 0                                        :REM ACTIVATE RAM BANK 0
120 FORI=0TO11:B=$A000+128*I:IFPEEK(B)<>0THENNEXT :REM SEARCH FOR TABLE $00
130 POKEB+$2E,ASC("Y")                            :REM SET KEYNUM $2E ('Z') to 'Y'
140 POKEB+$16,ASC("Z")                            :REM SET KEYNUM $16 ('Y') to 'Z'
170 REM
180 REM *** DOING THE SAME FOR SHIFTED CHARACTERS
190 REM *** IS LEFT AS AN EXERCISE TO THE READER
```

### Custom BASIN PETSCII code override handler

**Note**: This behavior only exists in R44 or later

Some use cases of the BASIN (CHRIN) API call may benefit from being able to modify its behavior, such as intercepting or redirecting certain PETSCII codes.  The Machine Language Monitor uses this mechanism to implement custom behavior for F-keys and for loading additional disassembly or memory display when scrolling the screen.

To set up a custom handler, one must configure it before each call to BASIN.

The key handler vector is in RAM bank 0 at addresses $ac03-$ac05.  The first two bytes are the call address, and the next byte is the RAM or ROM bank.  If your callback routine is in low ram, specifying the bank in $ac05 is not necessary.

The editor will call your callback for every keystroke received and pass the PETSCII code in the A register with carry set.  If your handler does not want to override, simply return with carry set.

If you do wish to override, return with carry clear.  The editor will then unblink the cursor and call your callback a second time with carry clear _for the same PETSCII code_.  This is your opportunity to override. Before returning, you are free to update the screen or perform other KERNAL API calls (with the exception of BASIN). At the end of your routine, set `A` to the PETSCII code you wish the editor to process. If you wish to suppress the input keystroke, set `A` to `0`.

```ASM
ram_bank = $00
edkeyvec = $ac03
edkeybk  = $ac05

BASIN    = $ffcf
BSOUT    = $ffd2

.segment "ONCE"
.segment "STARTUP"
    jmp start
.segment "CODE"

keyhandler:
    bcs @check
    cmp #$54 ; 'T'
    bne :+
    lda #$57 ; 'W'
    rts
:   lda #$54 ; 'T'
    rts
@check:
    cmp #$54 ; 'T'
    beq @will_override
    cmp #$57 ; 'W'
    beq @will_override
    sec
    rts
@will_override:
    clc
    rts

enable_basin_callback:
    lda ram_bank
    pha
    stz ram_bank ; RAM bank 0 contains the handler vector
    php
    sei
    lda #<keyhandler
    sta edkeyvec
    lda #>keyhandler
    sta edkeyvec+1
    ; setting the bank is optional and unnecessary
    ; if the handler is in low RAM.
    ; lda #0
    ; sta edkeybk
    plp
    pla
    sta ram_bank
    rts

start:
    jsr enable_basin_callback
    ; T and W are swapped
@1: jsr BASIN
    cmp #13
    bne @1
    jsr BSOUT
    ; normal BASIN
@2: jsr BASIN
    cmp #13
    bne @2

    rts

```

### Custom Keyboard Keynum Code Handler

**Note**: This behavior is for R43 or later, differing from previous releases.

If you need more control over the translation of keynum codes into PETSCII/ISO codes, or if you need to intercept any key down or up event, you can hook the custom scancode handler vector at $032E/$032F.

On all key down and key up events, the keyboard driver calls this vector with

* .A: keycode, where bit 7 (most-significant) is clear on key down, and set on key up.

The keynum codes are enumerated [here](https://github.com/X16Community/x16-rom/blob/master/inc/keycode.inc), and their names, similar to that of PS/2 codes, are based on their function in the US layout.

The handler needs to return a key event the same way in .A

* To remove a keypress so that it is not added to the keyboard queue, return .A = 0.
* To manually add a key to the keyboard queue, use the `kbdbuf_put` KERNAL API.

You can even write a completely custom keyboard translation layer:

* Place the code at $A000-$A58F in RAM bank 0. This is safe, since the tables won't be used in this case, and the active RAM bank will be set to 0 before entry to the handler.
* Fill the locale at $A590.
* For every keynum that should produce a PETSCII/ISO code, use `kbdbuf_put` to store it in the keyboard buffer.
* Always set .A = 0 before return from the custom handler.

```ASM
;EXAMPLE: A custom handler that prints "A" on Alt key down

setup:
    sei
    lda #<keyhandler
    sta $032e
    lda #>keyhandler
    sta $032f
    cli
    rts

keyhandler:
    pha

    and #$ff    ;ensure A sets flags
    bmi exit    ;A & 0x80 is key up

    cmp #$3c    ;Left Alt keynum
    bne exit

    lda #'a'
    jsr $ffd2

exit:
    pla
    rts
```

### Function Key Shortcuts

The following Function key macros are pre-defined for your convenience. These shortcuts only work in the screen editor. When a program is running, the F-keys generate the corresponding PETSCII character code.

| Key | Function   | Comment   |
|-----|------------|-----------|
| F1  | `LIST:`    | Lists the current program |
| F2  | `SAVE"@:`  | Press F2 and then type a filename to save your program. The `@:` instructs DOS to allow overwrite. |
| F3  | `LOAD "`   | Load a file directly, or cursor up over a file listing and press F3 to load a program. |
| F4  | 40/80      | Toggles between 40 and 80 column screen modes, clearing the screen. Pressing return is required to prevent accidental mode switches. |
| F5  | `RUN:`     | Run the current program. |
| F6  | `MONITOR`  | Opens the Supermon machine language monitor. |
| F7  | `DOS"$<cr>`| Displays a directory listing. |
| F8  | `DOS"`     | Issue DOS commands. |
| F9  | -          | Not defined.  Formerly cycled through keyboard layouts. Instead, use the `MENU` command to enter the X16 Control Panel, select one, and optionally save the layout as a boot preference. |
| F10 | -          | Not defined |
| F11 | -          | Not defined |
| F12 | debug      | debug features in emulators |

<!-- For PDF formatting -->
<div class="page-break"></div>
