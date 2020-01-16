---
layout: post
title: Heuristically measure reverse engineering efforts
date: 2020-01-11
permalink: /:title/
description: "Or how to tell if the soup needs more salt"
tags: [reverse engineering, thoughts]
share: true
---

When reversing, the resulting artifact of my efforts is usually an explaination in plain english of a system or a component.
I found myself asking more and more when is it that I've spent enough time on a given piece of software.
While day dreaming, a simple taxonomy came up to me:

To measure the degree of understanding, use the following taxonomy:

- **How**: lowest level of abstraction, explains how a given component works
- **What**: medium level of abstraction, explains what a component does without implementation details
- **Why**: highest level of abstraction, explains the role of the component in the system
