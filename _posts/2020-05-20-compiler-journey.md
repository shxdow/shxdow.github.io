---
layout: post
title: On compilers
date: 2020-05-08
permalink: /:title/
description: "Logging my journey in the magical world of compilers"
tags: [compilers, log]
share: true
published: false
---

This page are a bunch of timestamped notes I'm taking while trying to learn compilers.
Most of what I'm currently learning while embarking this magical journey is taken from "Crafting
Interpreters - Bob Nystrom". I'm by no mean trying to become a compiler god by the end of the week, I
just consider myself more of a curious undergrad that likes to poke at seemingly obscure areas of
engineering.

###### Wed 20 May 2020 

An interesting thing that I have already learned is how much I don't understand about what goes
underneath the code I write. When StackOverflow wizards come out with all the performance craziness
they do so because of a fundamental understanding of how languages internally work.

---

###### Sun 24 May 2020

I've just started the chapter on lexical analysis. It is called so because historically computers where so
memory constrained that reading sources files `(scanning)` and performing lexical analysis `(lexing)`
were two separate stages.

---

###### Wed 27 May 2020

The scanner/lexer divides lines of code into blobs called `lexemes`. A type is associated to these.
Once such chunk is recognized a token is emitted.
he rules that determine how characters are grouped into lexemes for some language are called its lexical grammar.

_Regular language_
> In theoretical computer science and formal language theory, a regular language
(also called a rational language) is a formal language that can be expressed using a regular expression,
in the strict sense of the latter notion used in theoretical computer science (as opposed to many 
regular expressions engines provided by modern programming languages, which are augmented with
features that allow recognition of languages that cannot be expressed by a classic regular expression) ~ Wikipedia


---

###### Wed Oct 21 2020

Lookaheads occur very frequently, thats why most compilers only look a couple 
characters at most and why lookaheads are so frowned upon in regular expression.

---

###### Fri Oct 23 2020

Challenges:

> The lexical grammars of Python and Haskell are not regular. What does that mean, and why aren’t they?

Python and Haskell's lexical grammars are said to be non-regular because they
can't be expressed with regular expressions (or at least, not in their entirety).
This implies that scanning strategies can vary.

> Aside from separating tokens—distinguishing print foo from printfoo—spaces aren’t used for much in most languages. However, in a couple of dark corners, a space does affect how code is parsed in CoffeeScript, Ruby, and the C preprocessor. Where and what effect does it have in each of those languages?

> Our scanner here, like most, discards comments and whitespace since those aren’t needed by the parser. Why might you want to write a scanner that does not discard those? What would it be useful for?

> Add support to Lox’s scanner for C-style /* ... */ block comments. Make sure to handle newlines in them. Consider allowing them to nest. Is adding support for nesting more work than you expected? Why?

###### Mon Oct 26 2020

This chapter is a bit more heavy on the theory size, I figured it would be appropriate to go through the basics of formal languages as I very ignorant about them.

> In formal language theory, a grammar (when the context is not given, often called a formal grammar for clarity) describes how to form strings from a language's alphabet that are valid according to the language's syntax. A grammar does not describe the meaning of the strings or what can be done with them in whatever contextÑonly their form. A formal grammar is defined as a set of production rules for strings in a formal language.
> Formal language theory, the discipline that studies formal grammars and languages, is a branch of applied mathematics. Its applications are found in theoretical computer science, theoretical linguistics, formal semantics, mathematical logic, and other areas.
> A formal grammar is a set of rules for rewriting strings, along with a "start symbol" from which rewriting starts. Therefore, a grammar is usually thought of as a language generator. However, it can also sometimes be used as the basis for a "recognizer"Ña function in computing that determines whether a given string belongs to the language or is grammatically incorrect. To describe such recognizers, formal language theory uses separate formalisms, known as automata theory. One of the interesting results of automata theory is that it is not possible to design a recognizer for certain formal languages.[1] Parsing is the process of recognizing an utterance (a string in natural languages) by breaking it down to a set of symbols and analyzing each one against the grammar of the language. Most languages have the meanings of their utterances structured according to their syntaxÑa practice known as compositional semantics. As a result, the first step to describing the meaning of an utterance in language is to break it down part by part and look at its analyzed form (known as its parse tree in computer science, and as its deep structure in generative grammar). 
