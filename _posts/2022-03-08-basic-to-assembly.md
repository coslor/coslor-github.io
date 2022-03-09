# Converting BASIC to Assembly

|Type           |BASIC structure                |Assembly Version                             |Notes                                |
|---------------|-------------------------------|---------------------------------------------|-------------------------------------|
| Remarks       | `990 REM THE START OF THE PROGRAM` | `; The start of the program`           |                                     |
|               | `GOSUB 1000:REM MAKE PEW PEW NOISE` | `jsr MakePewPewNoise ; makes pew pew noise, duh` | Anything after the semicolon on a line is ignored, unless it's inside quotes |
| Variables & Memory | `AU=42` (the first definition of *AU*) | `AnswerToUniverse: .byte 42`       | Defines the variable[^1], and loads it with the numeric value 42. Labels can be any length. AnswerToUniverse is a single byte, range -128 to 127 or 0-255 |
|               | `AU=17` (after AU has been defined) | `lda #17`<br>`sta AnswerToUniverse`   |                                     |
|               | `AU=PEEK(53281)`              | `lda 53281` <br>`sta AnswerToUniverse`<BR>  | Translation: "Load the current value of memory location 53280 (could also be written as $d020) into the accumulator, and store it in the location represented by `AnswerToUniverse`."<br>Assumes that `AnswerToUniverse` has already been defined and is a single byte. Note also that this will overwrite anything previously in the accumulator. |
|               | `POKE 53280,6`                | `lda #6` <br>`sta 53280`                    | Actually, it would be more common to use something like:<br>`lda #COLOR_BLUE`<br>`sta BORDER_COLOR`<br>...where COLOR_BLUE and BORDER_COLOR are constants that have been defined earlier in the program.<br>Another little note: constants are traditionally all in upper case. This isn't required, but it helps distinguish between constants and variable labels. |
| Comparisons   | `IF D<=3 THEN GOSUB 1000:SC=SC+1:GOTO 1300` | `lda DistToRock` <br>`cmp #MIN_DISTANCE` <br>`bne no_boom` <br>`jsr MakeBoomNoise`<br>`inc Score` <br>`jmp NextRock` | You haven't seen some of these operands yet, but hopefully you get the idea. Notice that we've restructured the code a little in the assembly version so that, instead of executing the remainder of the line if the distance *is* less than or equal to the minimum distance, we jump to other code if it *isn't*, and then describe what to do otherwise. |

[^1]: But what address does *AnswerToUniverse* actually represent? No, it's not 42; 42 is the value that is put into the address when the program first runs. So what is the address? The short answer is: don't worry about it, the assembler will take care of it and you'll almost never need to know the exact value, any more than you needed to know exactly what address a variable was held in, in BASIC. <br><br>
If you still really want to know: it depends on *AnswerToUniverse*'s place in the overall program structure. For example, if the first few lines of the assembly file look like this: 
  <br><br>`*=$0801` 
  <br><br>`;-------------------` 
  <br>`; BASIC header`                            (see [here](https:google.com) for more info about the BASIC header) 
  <br>`;-------------------` 
  <br>`.byte $0d,$08`                             BASIC link to next line address (that is empty in this case)
  <br>`.byte $e6,$07`                             the line number, which can be any value you want but is often the year the program was written (2022 in this case)
  <br>`.byte $9e,$20,'2','0','7','0'`             SYS to start the program; points to the first JMP statement, at *fake_start*
  <br>`.byte $32,$00,$00,$00`                     end of BASIC program
  <br>`;-------------------` 
  <br> `fake_start: jmp real_start`               GOTO our actual program code
  <br><br>`; ---constants---` 
  <br>`VIC2_start=$d800`						  here *VIC2_start* is a *constant*, a label that replaces a number, in this case the starting address for the VIC2 chip 
  <br><br>`;---variables---` 
  <br>`var_8: .byte $00`                          a single-byte variable
  <br>`var_16: .word $0000`                       a 2-byte variable
  <br>`AnswerToUniverse: .byte 42`
  <br><br> `;--- the actual program ---`
  <br>`real_start:	lda #0`
  <br> (rest of program)
  <br><br>...then what's the value of *AnswerToUniverse*? Well, in a real program the assembler will figure it for you, but just for fun, let's try it ourselves: start with the  beginning address, which is $801 or 2049. Then count the bytes: the BASIC header is 14 bytes; the JMP is 3 bytes; the constants don't take up *any* space, since they're just labels; `var_8` takes 1 byte; `var_16` takes 2 bytes; and then there's **AnswerToTheUniverse**, which should be 2049+14+3+1+2=2069, or $815. So the *address* of *AnswerToTheUniverse* is **$815**, and the initial *value* in it is **42**.
