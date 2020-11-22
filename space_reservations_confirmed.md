# Space Reservations Confirmed 

## This useful utility lets the more serious Timex-Sinclair user make use of space in upper memory.

#### By John Jainschigg

Every Sinclair ZX-81 (and TS-1000) programmer must be familiar with the technique of resetting the system variable RAMTOP to procure space in upper memory. RAMTOP reserve space has several intriguing qualities: it is immobile and entirely immune to functions of Basic ROM (such as NEW and LOAD), making it ideal not only for the storage of machine code rou- tines, but also a tempting resource for use in binary data storage schemes and program-to-program communi- cations. Unfortunately for those who wish to experiment with these more exotic applications, the simple Basic procedure most commonly used for RAMTOP reset is inappropriate for several reasons. 

## The Normal Approach 

Listing 1 demonstrates the usual approach. The Basic statements are entered from command mode on power-up. The number of bytes of storage space required is subtracted from the normal value of RAMTOP (corresponding to the current configuration of the TS-1000), and the result is POKEd to the system variable in high/low format. A NEW command then should be executed to zero system RAM and to rearrange it beneath the new RAMTOP address.

```
POKE 16388, (RAMTOP - #bytes) - INT ((((RAMTOP - #bytes)/256) * 256)) 
POKE 16389, INT ((RAMTOP - #bytes)/256) 
NEW
```
_Listing 1. Usual approach to RAMTOP reset._

NEW is the only Basic command that incorporates reformatting. Unfortunately, it does so in a manner destructive to the contents of memory, for which reason a Basic program cannot use a runtime variant of the above procedure to create space in high memory for its own use. Instead, the user must anticipate the need for RAMTOP reserve and create it prior to loading application programs. This two-step obligation is cumbersome in itself, and the result of it has been to limit the use of RAMTOP re- serve space by Basic programmers to schemes requiring predictable amounts of offset — offset that can be calculated and prepared ahead of runtime. 

## Another Way 

The subroutine Spacemaker (see Listing 2) constitutes an alternative approach to RAMTOP reset. Spacemaker creates RAMTOP reserve space in a nondestructive way — by reformatting the upper end of system memory. It allows a Basic program that incorporates it to conjure any degree of RAMTOP offset during execution and to put that space to immediate use. 

The mechanics of the subroutine are simple. When power is first applied to the TS-1000, the bootstrap procedure of the ROM formats available memory in the pattern shown in the memory map (Fig.l). The various partitioned blocks are herded into two broad sectors above and below a central reservoir of free space. The upper sector of system memory, comprising the gosub and machine stacks, is based at RAMTOP and builds downward, starting at the address immediately beneath the RAMTOP boundary. 

|Purpose| ZX-80 decimal memory location or system mnemonic |
| :-- | :-- |
|system variables|16384|
|program|16509| 
|display file|D-FILE|
|variables|VARS|
|keyboard buffer|E-LINE|
|calculator stack|STKBOT|
|FREE|STKEND|
|machine stack|SP| 
|GOSUB stack|ERR-SP| 
|RAMTOP reserve|RAMTOP|

_Figure 1. TS-1000 /ZX-81 memory map_

Besides RAMTOP, two additional pointers define the upper sector: the system variable ERR-SP, which marks the top (read "bottom") of the gosub stack, and the stack pointer register which marks the top (as above) of the machine stack. Creating RAMTOP reserve space is a matter of shifting the upper sector of system memory downward by the desired offset, and altering RAMTOP, ERR-SP and the stack pointer by the same value so that they point once more to appropriate addresses.

Basic is not a practical tool for this operation, however. One reason is that the stack pointer, an internal register of the Z-80 processor, cannot be changed directly by Basic commands. Another is that the block of bytes we wish to move is used intensively by the Basic system to manage program execution. The solipsistic conflict that would result from trying to use Basic to relocate bytes whose values are simultaneously required to manage the relocation would likely result in a system crash (how's that again?). In machine code, though, the procedure is extremely straightforward, as the subroutine's comments will show.

```
ADDRESS /MNEMONIC / CODE 
16515 SCF 55 ; 
16516 CCF 63 ; clear carry 
16517 LD(16507)SP 237,115,123,64 ; store SP 
16521 LD BC(16507) 237,75,123,64 ; SP into BC 
16525 LD HL(16388) 42,4,64 ; RAMTOP into HL 
16528 SBC HL,BC 237,66 ; RAMTOP-SP=sector length 
16530 LD DE,HL 84,93 ; result in DE 
16532 LD BC,OFFLO, OFFHI 1,0,0 ; offset into BC 
16535 LD HL(16388) 42,4,64 ; RAMTOP into HL 
16538 SBC HL,BC 237,66 ; RAMTOP offset=new value 
16540 LD(16388)HL 237,99,4,64 ; into RAMTOP 
16544 LD HL(16386) 42,2,64 ; ERR-SP into HL 
16547 SBC HL,BC 237,66 ; ERR-SP-o-f f set =new value 
16549 LD(16386)HL 237,99,2,64 ; into ERR-SP 
16553 LD HL(16507) 42,123,64 ; SP into HL 
16556 SBC HL,BC 237,66 ; SP offset=new value 
16558 LD SP,HL 249 ; into SP 
16559 LD BC,DE 66,75 ; length of block into BC 
16561 LD DE,HL 84,93 ; dest. of move into DE 
16563 LD HL( 16507) 42,123,64 ; start address into HL 
16566 LDIR 237,176 ; block copy loop 
16568 RET 201 ; return to BASIC 
16569 AND A,151 ; clear carry 
16570 LD(16507)SP 237,115,123,64 ; store SP 
16574 LD HL(16507) 42,123,64 ; SP into HL 
16577 LD BC(16412) 237,75,28,64 ; STKND into BC 
16581 SBC HL,BC 237,66 ; det. free space 
16583 LD BC,HL 68,77 ; result into BC 
16585 RET 201 ; and out 
```
_Listing 2. Spacemaker subroutine: an alternative approach to RAMTOP reset_ 

The Basic loader in Listing 3 serves to place the decimal opcodes of Spacemaker in a REM statement #1, 71 bytes in length, starting at address 16515 at the beginning of your Basic program listing. To use the subroutine, incorporate steps into the body of the program to poke the number of bytes of space required, in high/low format, to subroutine variables OFFLO and OFFHI at addresses 16533 and 16534. Spacemaker then may be called via a statement RAND USR 16515. 

```
1 REM "(71 spaces) " 
5000 FOR X = 16515 TO 16585 
5010 INPUT A 
5020 PRINT A
5030 POKE X,A 
5040 NEXT X 
```
_Listing 3. Basic loader._ 

_Note: When using Spacemaker, be careful not to set off more free space than is currently available to Basic. If you do, the utility will copy the upper end of the system RAM over the calculator stack, and a dazzling system crash will result. To help forestall this catastrophe, Spacemaker incorporates a sub-utility, Free (in bytes 16569-16585), which calculates how much space you have available. Free should be called prior to Spacemaker by incorporating the statement:_ 

```
LET SPACE = USR 16569 
```
_... into your Basic program. As long as the value returned for Space is somewhat greater than the number of bytes you wish to reserve, you should be okay. 
The subroutine uses the free double byte at 16507 and 16508 as a storage register during execution. The routine is relocatable as long as the new locations of OFFLO and OFFHI are taken into account, and the relevant poke statements are altered accordingly._
