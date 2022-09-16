---
layout: post
title: Google Null Poison attack
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

This attack originated from Google Project Zero's team, as an updated version
of The House of Einherjar, a `NUL` byte overflow technique originally published
in Malloc Maleficarium[1].

## Goals

As its predecessor, it aims to obtain two malloc chunks overlapping the same
memory region, the former considered free, the latter allocated and
attacker controlled.

## Constraints

One of the core differences in this version of the attack is the lack of
control of the `prev_size` field.
`NUL` termination errors are typical of string related tasks. An attacker
would not be able to provide a `prev_size` containing `NUL` bytes as they
would terminate the string supplied prematurely, being restricted to
impractically large values.

For instance, `0x0000000000000090` would terminate the string
provided before overflowing into the `size` field.

Even worse, they might not be able to control the `prev_size` quadword at all
, which might have program-defined value (such as a string suffix).

Lastly, if the program is compiled with a GLIBC version > 2.26 they have to
deal with additional mitigations that make targeting an allocated chunk even
harder.

## Execution

In order to achieve its goal, which is creating two overlapping chunks, this
technique makes use of an additional chunk that is used to set a `prev_size`
field. 
Friendly tip: drawing blocks while reading the following steps will make
everything easier. Please, save yourself an headache and do it.

Given `3` consecutive allocated chunks `A`, `B`, and `C` of sizes
`0x20`, `0x110`, `0x20`, the following steps will grant as the
exploitation primitive

![](/assets/img/posts/glibc-research/null-poison-starting-heap-1.png)

- Free `B`, the runtime sets `C.prev_size = 0x210` and `C.pre_inuse = 1`
    - `B` is added to the unsortedbin

![](/assets/img/posts/glibc-research/null-poison-starting-heap-2.png)

- Overflow into `B` and decrease its size from `0x210` to `0x200`
    - Keep in mind that `C.prev_size` is still `0x210`

![](/assets/img/posts/glibc-research/null-poison-starting-heap-3.png)

- Request a chunk `B1` of size `0x100` (remember to subtract `0x8` when using `malloc`)
    - triggers remaindering, linking the remaining `0x100` bytes into the unsortedbin

![](/assets/img/posts/glibc-research/null-poison-starting-heap-4.png)

- Request a chunk `B2` of size `0x100`
    - `malloc` tries to access the next chunk at `&B2+size(B2)` and tries to set its `prev_size` to
    `0x100` and its `prev_inuse` bit to 1. Fortunately, this happens to be exactly `0x10` before `C`,
    therefore leaving it untouched.
        - any requested size is fine, requesting a `0x100` chunk requests the
        last remainder leaving the heap in a cleaner state

![](/assets/img/posts/glibc-research/null-poison-starting-heap-5.png)

- Deallocate `B1`
- Deallocate `C`
    - this triggers a backward consolidation that creates a chunk spanning
    from `B1` to `C` therefore overlapping `B2`, which is still allocated

Allocating a big enough chunk will provide a pointer a memory region that
overlaps the same one pointed to by the unsortedbin
