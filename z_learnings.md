# Bare-metal basic (functionnaly useless) C program compilation using arm-none-eabi-gcc

command :
arm-none-eabi-gcc -ffreestanding -nostdlib main.c -o main.elf

with main.c that looks like this : 


int main(void) {

    while (1) {
        
    }
}

outputs : 
/usr/lib/gcc/arm-none-eabi/10.3.1/../../../arm-none-eabi/bin/ld: warning: cannot find entry symbol _start; defaulting to 0000000000008000


# ELF files content

It's not a regular text file, using vim isn't really relevant and won't provide any value of useful information.

.../theoOS/theoOS$ arm-none-eabi-size main.elf
   text    data     bss     dec     hex filename
     12       0       0      12       c main.elf


     
.../theoOS/theoOS$ arm-none-eabi-readelf -S main.elf
There are 9 section headers, starting at offset 0x8248:

Section Headers:
  [Nr] Name              Type            Addr     Off    Size   ES Flg Lk Inf Al
  [ 0]                   NULL            00000000 000000 000000 00      0   0  0
  [ 1] .text             PROGBITS        00008000 008000 00000c 00  AX  0   0  4
  [ 2] .persistent       PROGBITS        0001800c 00800c 000000 00  WA  0   0  1
  [ 3] .noinit           NOBITS          0001800c 000000 000000 00  WA  0   0  1
  [ 4] .comment          PROGBITS        00000000 00800c 000033 01  MS  0   0  1
  [ 5] .ARM.attributes   ARM_ATTRIBUTES  00000000 00803f 00002a 00      0   0  1
  [ 6] .symtab           SYMTAB          00000000 00806c 000130 10      7   8  4
  [ 7] .strtab           STRTAB          00000000 00819c 00005e 00      0   0  1
  [ 8] .shstrtab         STRTAB          00000000 0081fa 00004e 00      0   0  1
Key to Flags:
  W (write), A (alloc), X (execute), M (merge), S (strings), I (info),
  L (link order), O (extra OS processing required), G (group), T (TLS),
  C (compressed), x (unknown), o (OS specific), E (exclude),
  D (mbind), y (purecode), p (processor specific)



.../theoOS/theoOS$ arm-none-eabi-objdump -d main.elf

main.elf:     file format elf32-littlearm


Disassembly of section .text:

00008000 <main>:
    8000:       e52db004        push    {fp}            ; (str fp, [sp, #-4]!)
    8004:       e28db000        add     fp, sp, #0
    8008:       eafffffe        b       8008 <main+0x8>


.../theoOS/theoOS$ arm-none-eabi-nm main.elf
0001800c T __bss_end__
0001800c T __bss_start
0001800c T __bss_start__
0001800c T __data_start
0001800c T __end__
0001800c T _bss_end__
0001800c T _edata
0001800c T _end
00080000 B _stack
         U _start
00008000 T main
