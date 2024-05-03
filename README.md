####  ASM502
 
ASM502 is a 6502 assembler written in C99-compliant C code. It is small and light, and yet implements a useful subset of CA65 functionality.

I can hear everyone shout "we dont need another 6502 assembler", and they are right! In defence ASM502 has a few interesting points - it is (fairly) modern, compatible with a useful subset of CA65, and it is small, it can even compile/run (with cc65) on a Commodore 64 or Plus 4.

ASM502 uses standard 6502 assembler syntax. Internally ASM502 works with 32 bit numbers (ie the default C integer) depending on the C compiler options used. Numbers can be decimal (default), hex (precede with $) or binary (precede with %) ie:

```
   .byte 160, $A0, %10100000      ; Number 160 in decimal, hex, binary..
```

Unary operators can preceed an expression:
```
-  <(expression)    ; Low byte of expression
-  >(expression)    ; High byte of expression
-  -(expression)	   ; Negate expression
```
 
ASM502 supports variables and can evaluate complex expressions. It uses C-style operators "<<,>>,=,!=,<,>,&,|,^,+,-,*,/,%".

```
expr1 << expr2	; Shift expr1 left by expr2
expr1 >> expr2	; Shift expr1 right by expr2
expr1 = expr2		; True if equal
expr1 != expr2	; True if not equal
expr1 < expr2		; True if less
expr1 < expr2		; True if more
expr1 & expr2		; Bitwise AND
expr1 | expr2		; Bitwise OR
expr1 ^ expr2		; Bitwise EOR (EXCLUSIVE  OR)
expr1 + expr2		; Add
expr1 - expr2		; Subtract
expr1 * expr2		; Multiply
expr1 / expr2		; Divide
expr1 % expr2		; Modulus (remainder of division)
```

A variable must start with upper/lower case letter, then include numbers or underlines.  ie:

```
   My_var1 = (($123 + %1010 + 1234) * 2) & $ffff		; Assign const val to (My_var1)
   .word (My_var1 >> 1) + (33 * $22), $abc1, 0 		; Use that in .word code definition..
```

Here are some example lines:

```
						; Comments follow a semicolon.
label:					; lable can be a destination for a branch or jump or data. 
  lda #65				; 6502 load accumulator with decimal 65.
  .byte 1,2,3,4			; Define 4 bytes.
  .word $1234, $abcd	; Define 2 words (same as: .byte $34,$12,$cd,$ab)
  .byte <label, >label	; Low and high byte of label
```

ASM502 supports a subset of assembler macro/conditional directives.

```
*=(expression)						; Set start-address of assembly code..
.org  (expression)					; Set start-address of assembly code.. (alternative to *=.. )
.byte (expression),"strings"..		; Define some bytes in-line 
.word (expression),(expression)..	; Define some 16 bit words inline (in lo/hi byte order)
.end								; Terminate assembler.

.if (expression)		; Conditional compile following code if (expression) non-zero..
  asm code..
.else					; Compile following code if (expression) is zero..
  asm code..
.endif

.define var expression	; This is the same as "var=expression". more complex macro .define not (yet) implemented.
```

Most more complex directives (ie: macros) are not (yet) implemented.

Here is a more complete simple example program to print some text. 

```
;---------------------------------------------------------------------------------------
; Simple program to print some text.

* = 828				; Put code in C64 tape buffer, call with SYS 828.. (change for non-commodore computers..)
printchar = $ffd2	; Routine to print accumulator as char on screen   (change for non-commodore computers..)
IS_C64=1			; Set variable IS_C64 to 1

printtext:			; jsr printtext starts this program.
  ldx # 0  
printloop:				; loop point. 
  ldy textdata,x		; get character to print
  beq printdone			; Zero terminates print
  txa
  pha				; Save X on stack
  tya
  jsr printchar		; Print character in Accumulator
  pla
  tax				; Get X
  inx
  bne printloop
printdone:
  rts
  
textdata:
  .if IS_C64	; Only compile following if IS_C64 is non zero
    .byte "HELLO C64! ",0		; Define some text bytes, zero at end. 
  .else
    .byte "THIS IS NOT A C64! ",0	
  .endif
;---------------------------------------------------------------------------------------
```

to compile this program, cut and paste the above to a new text file called "testpr,s", then compile with:

```
  asm502 testpr.s testpr.prg
     (Specify correct name/path at start, ie:   ./asm502 testpr.s testpr.prg )
```
     
Load "testpr.prg" on your emulator (or real comp) and start with SYS 828.
There are more extensive code examples included with ASM502.


####   ASM502 notes
  
As things stand this program is a "work in progress". It started as an adaption of an earlier assembler of mine when I wanted something simpler and smaller than the CA65 (the CC65 assembler). Now CA65 is a very good assembler, but by neccessity it has to deal with far more complexity due to it being part of a C compiler - segmentation, object files, library formats etc.

Compiling the ASM502 single-file source just needs a C99 compatible C compiler, and is as simple as:

```
  # in Linux..
  cc asm502.c -o asm502 -Os
```

ASM502 is a 2 pass assembler. It opens and reads the source file twice, on the second pass (if there is no error) it will write the binary program  (with 2 byte start address header, ie name.prg) directly to disk. It does compile and run on a Commodore 64 (or other 6502 platforms with sufficient memory) using a C compiler like cc65. For these platforms the source file should be converted to upper case. 

In asm502.c there are macro defines that can be adjusted. Set USE_FULL = 0  to build a simple 1 pass direct assembler with no labels/variables. DEFINT is set as unsigned short, the default variable size for all label/var arithmetic. VAR_MAX is the max no of var/labels,  VAR_HEAP memory for all var names, VAR_MAXNAME max label name length.

In the examples folder are portable assembler sources (+binaries) for FREAX (nice 2k game for c64/vice20/c16), LIFET (game of life in 126 bytes) and a few other demos and examples.

ASM502 has some omissions, and probably some bugs too, but it is useable, so I thought I should put it out anyway. I havent done anything to it for a long while now. In the end I have found ASM502 interesting and useful, so maybe you will too!

The source code is released as free software under the GNU GPL3 licence, see GNU website:  <www.gnu.org/licenses/gpl-3.0.html>

