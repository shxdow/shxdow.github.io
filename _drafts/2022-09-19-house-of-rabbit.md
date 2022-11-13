---
layout: post
title: House of Rabbit
date: 2022-09-19
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


# Constraints

- The controlled quadword is further in memory that the target address
- No heap leak required

# Core idea

- Increase the maximum memory malloc can request to the kernel
    - this step is necessary as otherwise malloc with treat unusually large
    requests as exceptions, thus using mmap instead of being allocated from an arean 
- Pressure memory enough to induce malloc to reduce fragmentation by
consolidating all fastbins and moving them in the unsortedbin
- linking the chunk into the unsortedbin requiress some finess as there are
multiple integrity checks at several points of the process

# Execution

- Link into the unsortedbin a chunk of a large size. This will guard against
malloc's abort when it fails to find chunks in the unsortedbin
- Use a double free fastbin to link a fake chunk
- Tamper with the size field and avoid both backward and forward consolidation
by
    - bk: set prev_inuse to true
    - fd: set size to 0x1. Malloc ignores flags, and reads size = 0x00 bytes
- Request a large chunk to pressure memory and trigger malloc_consolidate
- The fake fastbin chunk of size 0x1 is linked into the unsortedbin
- Change the size of the fake chunk to 0x80001, which is the minimum value for
the largest bin

# Heap Feng Shui



# References

- [Google Project Zero Null Poison attack, pkexec exploit](https://googleprojectzero.blogspot.com/2014/08/the-poisoned-nul-byte-2014-edition.html).
- [Project Zero Bug tracker](https://bugs.chromium.org/p/project-zero/issues/detail?id=96&redir=1)

