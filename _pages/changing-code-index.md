---
title: "Code You Can Change"
permalink: /changing_code/
layout: single
author_profile: false
toc: false
sidebar:
  nav: "changing_code"
---

The subject is not clean code. It is **changeable** code -- and the difference matters, because "clean" is an aesthetic judgement and changeability is something you can measure.

This series works through SOLID, legacy code, and refactoring, in Kotlin, on a real codebase. It draws on four sources and **does not treat them as agreeing**, because they do not:

| Source | What it is good for | Where to be careful |
|---|---|---|
| **Feathers**, *Working Effectively with Legacy Code* | The strongest of the four. Technique-dense, not opinion-dense, and the problem it solves has not changed since 2004. | Its techniques lean hard on subclass-and-override, which Kotlin's `final`-by-default blocks. That translation gets its own module. |
| **Ousterhout**, *A Philosophy of Software Design* | Turns "good design" into measurable claims: change amplification, cognitive load, unknown unknowns, deep modules. | Short, opinionated, and light on worked examples. |
| **Martin**, *Clean Code* | Naming, boundaries, error handling, and the discipline of noticing. | Written for an earlier career stage, and the later chapters take function decomposition past the point where it helps. Read selectively. |
| **Fowler**, *Refactoring* (2nd ed.) | The mechanical catalogue, and the commit discipline that makes it safe. | It is a reference, not a read-through. |

**The most instructive thing in this area is where two of them contradict each other.** Clean Code says a comment is a failure of expression; A Philosophy of Software Design says a comment carries what code structurally cannot. They disagree about function length for the same reason. This series stages that argument rather than picking a winner, because knowing *why* each position is held is worth more than either conclusion.

Two of the five SOLID principles -- **OCP** and **DIP** -- were covered in depth in the [design patterns series]({{ "/design_patterns/" | relative_url }}) and are referenced rather than repeated here. The other three get a post each.

## The map

{% include series_map.html collection="changing_code" %}

## The rule this series is built on

From the [Foundations]({{ "/design_patterns/m0-fundamentals/" | relative_url }}) of the previous series, and it applies just as hard here:

> **Does the code get shorter, or at least easier to follow, afterward?** If a change leaves three more files and a flow that is harder to trace, it failed -- and "it's the correct design" is not a justification.

The failure mode of a refactoring course is a codebase churned for six months with nothing shipped. Every module ends with a stopping condition for that reason.
