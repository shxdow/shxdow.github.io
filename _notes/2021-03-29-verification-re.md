---
layout: post
title: Reverse Engineering the implementation of LLBM - Part I
author: Toure Pape Alpha
date: 2021-03-29
permalink: /:title/
description: "Introduction to logic"
tags: [reverse engineering, logic, math]
share: true
published: true
status: Ongoing
---

Reversing notes:
each segment starts with an hypothesis and builds from there

## Where does this binary start its main() ?


The function at address `0x00402b20` is main. It it is possible to infere this by the fact that the
function `entry` (found by Ghidra because every executable file format must provide the address of
the entry point in its header) calls HLT after a function call whose parameters are other functions
This is a very common pattern in ELF files (is it common for PE and MacO too ?)


## What does the main() do ?

