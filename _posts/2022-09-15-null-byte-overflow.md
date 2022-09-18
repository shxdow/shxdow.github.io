---
layout: post
title: Google NUL Poison attack
date: 2022-09-15
permalink: /:title/
description: "Notes on exploitation techniques in Gnu C Library"
tags: [linux]
share: true
comments: false
published: true
status: Ongoing
---

The hacking community has always been built upon intellectual curiosity,
knowledge sharing and the desire of pushing the boundaries of what is
currently possible. Despite these tenants, or perhaps becasue of them, it
often falls short in the systematization of knowledge, leading to treating
memory corruption exploitation as a folkloristic discipline, passed informally
from one another, rather than a body of knowledge existing on its own.


---
* toc
{:toc}
---


# Introduction

Memory corruption vulnerabilities can arise from very benign programming
errors and can lead to code execution if exploited by a skilled attacker.  

One byte overflows are no different, and can be leveraged to craft fully
fledged exploits against binaries linked to Linux's ptmalloc2 memory allocator.  

This technique was originally discovered by Google Project Zero's team, as an
updated version of [The House of Einherjar](https://github.com/umbum/pwn-archive/blob/master/how2heap/house_of_einherjar.c),
a `NUL` byte overflow exploitation technique dating several years prior to 2014.  

As many modern heap exploitation techniques, it can be used to leak memory
addresses by obtaining two malloc chunks overlapping the same memory region.
One considered free and the other one allocated and attacker controlled. This
post covers backward consolidation specifically, but the principles can be
applied to forward consolidation as well.

# Required reading

The reader should be familiar with malloc internals before reading the following chapters:

- [Gnu C Library Wiki, Overview of Malloc](https://sourceware.org/glibc/wiki/MallocInternals)

# Constraints and and `NUL` termination errors

One of the core differences in this version of the attack is the lack of
control of the `prev_size` field:
`NUL` termination errors are typical of string related tasks.  
An attacker would not be able to provide a `prev_size` containing `NUL` bytes
as they would terminate the string supplied prematurely, being restricted to
impractically large values.

For instance, `0x0000000000000090` would terminate the string
provided before overflowing into the `size` field.

Even worse, they might not be able to control the `prev_size` quadword at all
, which might have program-defined value (such as a string suffix).


# Heap manipulation

In order to achieve its goal, which is creating two overlapping chunks, an
attacker needs to be able to somewhat control memory allocations.  
Such degree of control over the memory layout may look unrealistic at first
sight, but its actually a very common instance of any program running a
scripting engine.

## Initial memory state

Note: _this assumes a GLIBC version < 2.26_

Start by allocating 4 consecutive chunks `A`, `B`, `C` and `D`:will of sizes
`0x20`, `0x110`, `0x90`, `0x20` respectively. Note that chunk `D` is only used
to prevent consolidation with the top chunk and as such is not strictly necessary.

![](/assets/img/posts/glibc-research/null-poison-starting-heap-1.png)

Free chunk `B`, the runtime will set `C.prev_size = 0x210` and `C.prev_inuse =
0`. Chunk `B` is added to the unsortedbin as its size does not fit any of the
fastbins (singly-linked lists holding sizes from `0x20` to `0xb0`)

![](/assets/img/posts/glibc-research/null-poison-starting-heap-2.png)

## Leveraging the overflow

Now it's the time to use the overflow bug at the attacker's disposal:
overflow into `B` and decrease its size from `0x210` to `0x200`.  
Keep in mind that `C.prev_size` is still `0x210`

![](/assets/img/posts/glibc-research/null-poison-starting-heap-3.png)

Request a chunk `B1` of size `0x100`. Malloc searches through its unsortedbin
for a block of size greater than or equal to `0x100`.
Finding chunk `B`, it triggers remaindering, linking the remaining `0x100`
bytes into the unsortedbin.

![](/assets/img/posts/glibc-research/null-poison-starting-heap-4.png)

Request a chunk `B2` of size `0x100`: malloc tries to access the next chunk at
address `&B2+chunk_size(B2)` and tries to set its `prev_size` to `0x100` and
its `prev_inuse` bit to 1. Fortunately, this happens to be exactly `0x10`
before `C`, therefore leaving its fields untouched.  
Any requested size is fine, requesting a `0x100` chunk requests the last
remainder leaving the heap in a cleaner state.

![](/assets/img/posts/glibc-research/null-poison-starting-heap-5.png)

Free chunk `B1`, setting up a backward consolidation that starts from chunk
`C` and goes all the way back to `C1`.  

![](/assets/img/posts/glibc-research/null-poison-starting-heap-6.png)

Free chunk `C` 

![](/assets/img/posts/glibc-research/null-poison-starting-heap-7.png)

this triggers the backward consolidation and create a free chunk spanning from
`B1` to `C` therefore overlapping `B2`, which is still allocated.

![](/assets/img/posts/glibc-research/null-poison-starting-heap-8.png)

Allocating a big enough chunk will trigger remaindering once again and links
the remainder into the unsortedbin, thus setting the first two quadwords of `C`
to the address of the unsortedbin.

![](/assets/img/posts/glibc-research/null-poison-starting-heap-9.png)

## Leakage of memory addresses

If this attack is used in the first stage of an exploit, usually an attacker
leaks libc and heap addresses via this memory setup:  
Leaking a libc address is as simple as getting an chunk into the unsortedbin
(therefore setting its forward and backward pointers) and reading one of these
values by reading memory from a chunk overlapping the one just free'd.


# References

- [Google Project Zero Null Poison attack, pkexec exploit](https://googleprojectzero.blogspot.com/2014/08/the-poisoned-nul-byte-2014-edition.html).
- [Chromium ticket](https://bugs.chromium.org/p/project-zero/issues/detail?id=96&redir=1)

