---
title: notes
layout: post
permalink: /notes/
---

This page is heavily inspired from [gwern.net/notes](https://www.gwern.net/Notes), it is a collection of things I found interesting enough to store somewhere

> Half the challenge of fighting procrastination is the pain of starting—I find when
I actually get into the swing of working on even dull tasks, it’s not so bad. 
So this suggests a solution: never start. Merely have perpetual drafts, 
which one tweaks from time to time. And the rest takes care of itself.

# Statistics

It is an understatement to say that I was left unsatisfied by the stats course I took in university.
I since made it my goal to properly go over statistics making an effort to properly learn the discipline.
The following notes are mostly taken from [Professor Knudson' s youtube videos](https://www.youtube.com/playlist?list=PLdxWrq0zBgPW0554eqyaR_jYMJ1ux5MgI)

### Probability Groundwork

#### Experiment:

Any procedure that:

1. can be repeated (theoretically) $\infty$ many times
2. has a well-defined set of possible outcomes  

_eg_: 
Given $4$ red and $6$ brown M&Ms, choose $1$, record color, and put it back

#### Sample space:

Set of all possible outcomes, denoted $S$

#### Sample outcome

One possible outcome, an element
$$
s \in S
$$

_eg_: flip a coin two times:

$$
S = \{ HH, HT, TH, TT \}
$$  
or  
$$S = \{HH, HT, TT\}$$ if order does not matter  
one sample otucome is 2 tails $(TT)$

# Technology

Computer science and engineering, Information science, etc

### Software Engineering

_Why is software engineering such a mess compared to other engineering disciplines ?_

One of the first things I thought about is its very immaterial nature:
computer systems are far less limited in the growth of their complexity 
compared to the physical ones that have characterized engineering as 
a whole for centuries. Such over-linear growth has systematically 
made our systems grow in their complexity at a much faster rate at 
which we comprehend them. Computer security is the epitome of such
phenomena as security vulnerabilities are all about [^1] leveraging
broken assumptions and deeper understanding of the underlying system.

> Infosec is all about the mismatch between our intuition and the actual behavior of the systems we build. That makes it harmful to study the field as an abstract, isolated domain. To truly master it, dive into how computers work, then make a habit of asking yourself "okay, but what if assumption X does not hold true?" every step along the way. _~ lcamtuf_

Unlike fields such as medicine and lawyers, it is not unheard of practitioners
having backgrounds that are all over the place
<!-- More problems: -->
<!--  -->
<!-- - Fundamental limits of computation (Limits of Turing machines, Incompleteness theorems) -->
<!-- - Very heterogeneous backgrounds among practitioners (This is both a pro and a con) -->

# Quotes

> "In any field, find the strangest thing and then explore it." ~ John Archibald Wheeler


This quotes is a really interesting answer to a question I don't know yet

--- 

> "A very useful heuristic, a potent filter:  When someone criticizes you, train to immediately ask yourself:  "Would I rather be him/her, or I'd rather be me?" before taking the remark at face value." - Nassim Nicholas Taleb

[Source](https://twitter.com/TalebWisdom/status/1199392662304034816)

[^1]: Needless to say, primitive and hard to reason about tools do play their role
