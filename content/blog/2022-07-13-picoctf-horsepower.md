---
title: "PicoCTF Horsepower - V8 exploitation"
slug: "picoctf-horsepower-v8-exploitation"
date: 2022-07-13T00:00:00Z
description: "Solution to a V8 exploitation challenge"
tags: [ctf]
comments: false
---


This is the solution to Horsepower, one of the simplest v8 challenges
available I solved one year ago or so.
Writeups can be found all over the internet so I'd rather not write
one of my own. Thanks to all the people that helped me!

![d8 terminal output from the exploit: leaked float_map, element and object-array addresses, a WASM RWX page at 0xd8966000, shellcode written, ending in flag{this_is_not_the_real_flag}.](/assets/img/d8-horsepower.png)

# Full exploit code

The code can be found on [GitHub](https://github.com/shxdow/exploits/blob/master/picoCTF_horsepower.js)
