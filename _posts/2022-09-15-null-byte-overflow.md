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

---
* toc
{:toc}
---

This attack originated[^1] from Google Project Zero's team, as an updated version
of [The House of Einherjar](https://github.com/umbum/pwn-archive/blob/master/how2heap/house_of_einherjar.c),
a `NUL` byte overflow technique originally published.

# Goal

As its predecessor, it aims to obtain two malloc chunks overlapping the same
memory region, the former considered free, the latter allocated and
attacker controlled.

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

<!-- Lastly, if the program is compiled with a GLIBC version > 2.26 they have to -->
<!-- deal with additional mitigations that make targeting an allocated chunk even -->
<!-- harder. -->

# Heap manipulation

In order to achieve its goal, which is creating two overlapping chunks, this
technique makes use of an additional chunk that is used to set a `prev_size`
field. 

## Initial memory state

Start by allocating 4 consecutive chunks `A`, `B`, `C` and `D`:will of sizes
`0x20`, `0x110`, `0x20`, `0x20` respectively. Note that chunk `D` is only used
to prevent consolidation with the top chunk and as such is not strictly necessary.

![](/assets/img/posts/glibc-research/null-poison-starting-heap-1.png)

Free chunk `B`, the runtime will set `C.prev_size = 0x210` and `C.prev_inuse =
1`. Chunk `B` is added to the unsortedbin as its size does not fit any of the
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
`C` and goes all the way back to `C1`.  Free chunk `C` to trigger the backward
consolidation and create a free chunk spanning from `B1` to `C` therefore
overlapping `B2`, which is still allocated.

Allocating a big enough chunk will provide a pointer a memory region that
overlaps the same one pointed to by the unsortedbin.

## Leakage of memory addresses

If this attack is used in the first stage of an exploit, usually an attacker
leaks libc and heap addresses via this memory setup:  
Leaking a libc address is as simple as getting an chunk into the unsortedbin
(therefore setting its forward and backward pointers) and reading one of these
values by reading memory from a chunk overlapping the one just free'd.

# References

[^1]: [Google Project Zero Null Poison attack, pkexec exploit](https://googleprojectzero.blogspot.com/2014/08/the-poisoned-nul-byte-2014-edition.html).
