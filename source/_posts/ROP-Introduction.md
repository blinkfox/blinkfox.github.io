---
layout:   post
title:      "Introduction to ROP"
date:       2019-07-24
author:     "D4mianwayne"
tag:      rop, pwn, bof
category: Return Oriented Programming
---


This blog post will let you know about the ROP or Return Oriented Programming. 

# What is ROP?

Return Oriented Programming is a modern method of exploiting a binary that will allow us to take control of the stack and the abuse program's control flow by the help of gadgets.
Often times, this technique is used to exploit a binary which takes input without bound checking that will result in overflow of the memory in which the input is being stored resulting in segmentation fault.
This method is only used when we have handful of gadgets i.e. instruction sequences ending with **"ret"** or byte **"c3"**.

# Prerequisities 

Since, this method of exploitation is based on analyzation of functions and memory address which requires some basic reverse engineering and understanding of assembly language.

So, for reverse engineering you can refer to following resources in order to learn ROP.

* [Reverse Engineering with radare2](https://medium.com/@jacob16682/reverse-engineering-using-radare2-588775ea38d5)
* [Reverse Engineering with gdb](https://medium.com/@rickharris_dev/reverse-engineering-using-linux-gdb-a99611ab2d32)
  
As of now, I've only included radare2 and gdb which is going to be used for this series.

For Assembly, you can refer the follwoings:-

* [Assembly Language Guide 1](http://www.cs.princeton.edu/courses/archive/spr17/cos217/lectures/13_Assembly1.pdf)
* [Assembly Language Guide 2](https://www.cs.virginia.edu/~evans/cs216/guides/x86.html)

# Tools

This can be a little long because **the more the merrier**.

First off, we need something to analyze the binary.

##### Radare2

If you've ever tried binary analysis and reverse enginnering you must have come across radare2, which is a great binary analysis CLI tool and it has a built-in **gadget** finder.

##### GDB-PEDA

This is Python Exploit Development Assitance plugin for GDB which can be found [here](https://github.com/longld/peda).

##### Pwntools

This is absolutely a great python library which will help you with execution of your exploit by providing helpful functions, which can be get from [here](https://github.com/arthaud/python3-pwntools).

##### Ropper

This is also a great tool for finding gadgets within a binary, which can be obtained from [here](https://github.com/sashs/Ropper).

# Finding Gadgets

From my experience as of now, I've used **ropper** and radare2's built-in function **/R < instruction >**. With the use of these two tools you'll have the gadgets which will help you in bypassing DEP(Data Execution Prevention) hence, executing your payload.

### Use of gadgets

So, as of now you know that in order to build a ROP chain we have to get the binary's corresponding gadgets. Now, I'll tell you what is the **exact** use of gadget is.

##### Loading Constants to Register

With the help of ropper or radare2 you can find the `pop` instruction with a `ret` which can be used to store a constant into stack for further use.
Let a gadget be `pop edi, ret`, this will pop the `edi` register value from the stack and return the address to top of the stack.

###### System Call

System call i.e. `int 0x80` followed by `ret` instruction can be used to interrupt a kernel call that we have setup using previous gadget.
Following are the system call gadgets:-

* `int 0x80; ret`
* `call gs:[0x10]; ret`

### Gadget to lookout

There are some gadget which are better left alone i.e. we need to avoid these gadgets in order to avoid corruption of the stack frames.
* Gadgets with `pop ebp; ret` will mess our stack frames.
* Gadgets ending in pop ebp followed by ret or have the instruction pop ebp. Will also mess up our stack frame.

Sometimes these gadgets dont affect the overall execution of ROP shell. It depends on the execution
flow and will it be interrupted by changing the frame pointer.

# Continuing the series

This blogpost will help you in understanding the what and why of Revserse Oriented Programming. All of the resources will help you in understanding the Assembly and a little of reverse engineering.

Next I'll be posting how to get build a ROP chain from binary. Until then, read out all the resoures.



