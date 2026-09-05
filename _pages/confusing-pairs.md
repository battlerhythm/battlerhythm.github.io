---
title: "Confusing Pairs"
permalink: /design_patterns/confusing-pairs/
layout: single
author_profile: false
toc: true
toc_sticky: true
sidebar:
  nav: "design_patterns"
redirect_from:
  - /design_patterns/structural_patterns_comparison/
  - /design_patterns/decorator_vs_composite_pattern/
---

When two patterns draw the same diagram, the thing that separates them is always the intent. This page collects the one-sentence tests. It is a living page -- new pairs get added as the series goes on.

## The quick table

| Pair | The one sentence that separates them |
|---|---|
| **Strategy vs State** | Does the *client* know about the transition, or does the *object itself*? |
| **Strategy vs Template Method** | Composition, swappable at runtime, versus inheritance, fixed at compile time. |
| **Strategy vs Bridge** | Swapping an algorithm versus separating **two independent axes** of variation. |
| **Bridge vs Adapter** | Prevention at design time versus repair after the fact. |
| **Adapter vs Decorator** | **Changes** the interface versus **keeps** it and adds to it. |
| **Decorator vs Proxy** | **Adds** behavior versus **controls access** to it. Structurally identical. |
| **Decorator vs Chain of Responsibility** | Always passes through versus **may stop here**. |
| **Decorator vs Composite** | **One** wrapped child plus new behavior versus **many** children. |
| **Facade vs Adapter** | Invents a **new, simpler** interface versus matching one **somebody else defined**. |
| **Factory Method vs Abstract Factory** | **One** product versus a **family** that must match. |
| **Builder vs Abstract Factory** | The **construction process** is complex versus **which kind** to construct. |
| **Observer vs Mediator** | The publisher **knows nothing** versus the hub **knows everyone**. |
| **Command vs Strategy** | Objectifies the **request** (undo, queueing) versus objectifies the **algorithm**. |
| **Visitor vs sealed + `when`** | New operations are cheap versus **new types are cheap**. Opposite ends of the Expression Problem. |

## The four wrappers

Adapter, Decorator, Proxy and Facade all wrap another object and forward calls to it. This is the group people mix up most, so it is worth stating as a single table.

| Pattern | The interface | The purpose |
|---|---|---|
| **Adapter** | **Changes** it | Join two things that were never meant to fit |
| **Decorator** | **Keeps** it | Add responsibility, stackable |
| **Proxy** | **Keeps** it | Control access: lazy, cached, permissioned, remote |
| **Facade** | **Invents** one | Simplify a complicated subsystem |

Decorator and Proxy are the hard one, because their class diagrams are the same. The test: **if you are adding behavior it is a decorator; if you are deciding whether, when, or for whom the original behavior runs, it is a proxy.**

## The three that share one diagram

Strategy, State and Bridge are near-identical on paper. One question separates all three.

- **Strategy** -- the client picks the algorithm and injects it. Nobody transitions anywhere.
- **State** -- the states hand control to one another. The object owns its own transitions.
- **Bridge** -- there are **two** hierarchies growing independently, and any member of one can pair with any member of the other.

If you find yourself asking "is this Strategy or State?", ask instead: **who knows about the transition?**
