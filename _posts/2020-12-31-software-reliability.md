---
layout: post
title: On software, engineering and reliability
date: 2020-12-28
permalink: /:title/
description: "Reflections on engineering"
tags: [software engineering]
share: true
status: ongoing
---

This post aims to answer a recurring question I've had in the last 9 months as 
I've had the chance to find myself being tasked to build a system rather than
break it. Many mistakes of my own have driven me multiple times to asking
myself, how could an higher degree of reliability be achieved for the given
system I was working with.

_Why does software seemingly fail more often than other engineering artifacts ?_

There are many factors that could be taken into account when diagnosing this
problem, but everytime I come back to it, it feels as if the entire question
boils down to a few things:

1. Complexity
2. Economic incentives
3. Experience

**On complexity**

One of the first things I thought about is the very immaterial nature of the
subject at hand:
computer systems are far less limited in the growth of their complexity 
compared to the physical ones that have characterized engineering as 
a whole for centuries. Such over-linear growth has systematically 
made our systems grow in their complexity at a much faster rate at 
which we comprehend them. Meltdown, Rowhammer, et al. are prime examples.
Moreover, [@halvarflake](twitter.com/halvarflake)
in his [keynote]() went over to explain an even more fundamental
way in which complexity is not bounded: it is much more expensive to
build a special purpose machine than it is to use a general purpose one to simulate
whats needed. This effectively makes every special purpose machine
(i.e. program) inherit the entire jungle in which the general one lives as opposed to
only taking a banana one is looking for. Security is the epitome of such
phenomena as security vulnerabilities are all about [[Needless to say, tools that are hard to reason about do play their role. But it goes much much further than that. Producing secure software is _really_ hard.
::lsn]] leveraging
broken assumptions and deeper understanding of the underlying system.

{:.small .oblique}
> Infosec is all about the mismatch between our intuition and the actual behavior of the systems we build. That makes it harmful to study the field as an abstract, isolated domain. To truly master it, dive into how computers work, then make a habit of asking yourself "okay, but what if assumption X does not hold true?" every step along the way.
> <cite>- lcamtuf</cite>

**Economic incentives**

Silicon Valley and the startup culture have been one of the driving forces that power
<!-- Due to the lack of regulation of the field, the potential financial upside and, -->
<!-- for the most part, lack of rigid processes to adhere to software could be -->
<!-- developed at a much higher speed than a bridge would.  -->


Now, building a bridge
takes a much longer time for many more reasons than the ones listed, but bear
with me for the sake of the argument. 

**On experience**

Despite achieving unmatched [[Even though such results are indeed obtained by standing on the shoulders of giants they are still very impressive nonetheless::rsn]] results when it comes to changing society,
computer science has 100 years at most. It does not take too much to think of
people who were at the frontier of the field and designed very common things
such as Unix, C, UTF8, PageRank, ...  
You name it, we have it.

<!-- - Fundamental limits of computation (Limits of Turing machines, Incompleteness theorems) -->

