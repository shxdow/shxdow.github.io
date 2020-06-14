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

<!-- # Statistics -->
<!--  -->
<!-- It is an understatement to say that I was left unsatisfied by the stats course I took in university. -->
<!-- I since made it my goal to properly go over statistics making an effort to properly learn the discipline. -->
<!-- The following notes are mostly taken from [Professor Knudson' s youtube videos](https://www.youtube.com/playlist?list=PLdxWrq0zBgPW0554eqyaR_jYMJ1ux5MgI) -->
<!--  -->
<!-- ### Probability Groundwork -->
<!--  -->
<!-- #### Experiment: -->
<!--  -->
<!-- Any procedure that: -->
<!--  -->
<!-- 1. can be repeated (theoretically) $\infty$ many times -->
<!-- 2. has a well-defined set of possible outcomes   -->
<!--  -->
<!-- _eg_:  -->
<!-- Given $4$ red and $6$ brown M&Ms, choose $1$, record color, and put it back -->
<!--  -->
<!-- #### Sample space: -->
<!--  -->
<!-- Set of all possible outcomes, denoted $S$ -->
<!--  -->
<!-- #### Sample outcome -->
<!--  -->
<!-- One possible outcome, an element -->
<!-- $$ -->
<!-- s \in S -->
<!-- $$ -->
<!--  -->
<!-- _eg_: flip a coin two times: -->
<!--  -->
<!-- $$ -->
<!-- S = \{ HH, HT, TH, TT \} -->
<!-- $$   -->
<!-- or   -->
<!-- $$S = \{HH, HT, TT\}$$ if order does not matter   -->
<!-- one sample otucome is 2 tails $(TT)$ -->

### On Software Engineering

_Why does software seemingly fail more often than other engineering artifacts ?_

It really boils down to two things:

1. Complexity
2. Experience

**On complexity**

One of the first things I thought about is its very immaterial nature:
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
only taking a banana. Security is the epitome of such
phenomena as security vulnerabilities are all about [^1] leveraging
broken assumptions and deeper understanding of the underlying system.

> Infosec is all about the mismatch between our intuition and the actual behavior of the systems we build. That makes it harmful to study the field as an abstract, isolated domain. To truly master it, dive into how computers work, then make a habit of asking yourself "okay, but what if assumption X does not hold true?" every step along the way. _~ lcamtuf_

**On experience**

Despite achieving unmatched [^2] results when it comes to changing society,
computer science has 100 years at most. It does not take too much to think of
people who were at the frontier of the field and designed very common things
such as Unix, C, UTF8, PageRank, ...  
You name it, we have it.

<!-- - Fundamental limits of computation (Limits of Turing machines, Incompleteness theorems) -->

# Quotes

> "In any field, find the strangest thing and then explore it." ~ John Archibald Wheeler

If I reacall correctly, this is close to what Paul Graham calls a good heuristic to identify interesting problems

--- 

> "When you are overloaded, don't think of it as not having enough time; think of it as having too much to do. You can't give yourself more time, but you can give yourself less to do, at least for the moment." ~ [@KentBeck](https://twitter.com/kentbeck) and [@martinfowler](https://twitter.com/martinfowler)

--- 

> "A very useful heuristic, a potent filter:  When someone criticizes you, train to immediately ask yourself:  "Would I rather be him/her, or I'd rather be me?" before taking the remark at face value." - Nassim Nicholas Taleb

[Source](https://twitter.com/TalebWisdom/status/1199392662304034816)

[^1]: Needless to say, tools that are hard to reason about do play their role. But it goes much much further than that. Producing secure software is _really_ hard.
[^2]: Humm I forgot what I wanted to say
