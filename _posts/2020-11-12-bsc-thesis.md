---
layout: post
title: Program analysis for Software Security via SMT solvers
date: 2020-11-12
permalink: /:title/
description: "On Program Analysis for Software Security"
tags: [compilers, log, sat, smt]
share: true
published: false
---


This page are a bunch of timestamped notes I'm taking while doing my Bachelors thesis in Computer
Engineering.
Most of what I'm currently learning is coming from whatever I could scrap from the Internet, ranging
from PhD thesis, books, tweets, conferences and so on...

##### Thu Nov 12 2020

As of now, the plan is to leverage LLVM to generate IR from C code and convert that to something
z3-compatible. There seem to some tools that already get the job done, most of which look far from
being actively maintained.
