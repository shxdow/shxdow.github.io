---
title: "Kernel notes"
slug: "nt-kernel-notes"
date: 2016-04-20T00:00:00Z
description: "Notes on OS internals, kernels and ..."
tags: ["notes"]
toc: true
---


This is a collections of notes on various kernel related topics, featuring
Windows NT. A good chunk of these snippets were gathered as a precocius high
schooler around somewhere between 2016 and 2017, therefore, do not expect the
quality to be on par of a peer reviewd research paper, far from it. The notes
don't lie in any particular order as most of this material was gathered through
books during self study.

---

## What are PCR, PRCB and why do we have them?

_PCR_ (Process Control Region) is a per-processor data structure used in the
kernel that contains critical information about it. Inside of it
there's _PRCB_ (Process Region Control Block) which contains other stuff
about it (both structures are undocumented). These are used in order to access the PCR _FS/GS_ are used
(_x86/x64_ respectively)


## How does an interrupt work ?

The operating system uses data structures (_IDT_) to know which interrupts 
are handled and how.


## What are SYSENTER / SYSEXIT and why do we need them

These two instructions are used to implement syscalls. (The reason we have
syscalls in the first place is beacuse user space *can't* directly access
hardware / physical mem addresses.)


## How are syscalls information data stored/retrieved

Windows distinguishes GUI syscalls from non-GUI ones: `W32pServiceTable` and
`KiServiceTable`. They have a base and a limit.

	
## Explain in which way a system call can be implemented (NT)

Syscalls can be implemented by usign either having a dedicated int code
(`0x2e`) or a trap instructions (`SYSENTER/SYSEXIT`).

## Explain what IRQL is

Interrupt Request Level are numbers associated with interrupts and describe
the "priority" of the int. IRQL is an unsigned char and is per-processor:
interrupts are handled only if they number is equal or higher than the
current IRQL. 

<!-- doubt 1: How does the CPU know when to increase/decrease IRQL ? Does it
have a queue or something ?


## What are MDLs,why do we need them and what is their relationship w/ the MMU ?

-->

## Explain execution contests

Unlike Linux, Windows distinguishes threads from processess (every process
has at least 1 thread). A contest is a collection of all the information
relevant to the process.


## Explain what a work item is 

Worker threads are kernel mode threads executed on behalf of the
caller. 

## Kernel routines decompilation

See [GitHub](https://github.com/shxdow/low-level-exercises/tree/master/practical-reverse-engineering).

# References

- [Practical Reverse Engineering: x86, x64, ARM, Windows Kernel, Reversing Tools, and Obfuscation](https://www.goodreads.com/en/book/show/18227589)
- [Windows Internals Series](https://learn.microsoft.com/en-us/sysinternals/resources/windows-internals)
- [Geoff Chappell, Software Analyst](https://geoffchappell.com)
