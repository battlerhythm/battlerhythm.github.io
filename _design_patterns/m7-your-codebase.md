---
title: "Auditing Your Own Code"
order: 26
module: "M7"
module_title: "Integration"
session: "14"
intent: "Smells that map to patterns, patterns that should be removed, and a one-page guide for deciding."
status: "complete"
excerpt_separator: "<!--more-->"
categories:
  - Design Patterns
tags:
  - "Integration"
  - "Refactoring"
---

The last session, and the one that runs in both directions. Finding where a pattern would help is half of it; finding where one is already costing you and buying nothing is the half nobody does.

<!--more-->

## Direction 1: smells that have a pattern answer

Each row is a query you can actually run, not a feeling.

| Smell | How to find it | Suspect |
|---|---|---|
| One type branched on in 3+ files | grep the variant names, count distinct files | [Strategy]({{ "/design_patterns/m1-strategy/" | relative_url }}) or [State]({{ "/design_patterns/m1-state/" | relative_url }}) -- ask who owns the transition |
| A class with N independent booleans | count `var .*Boolean` properties | [State]({{ "/design_patterns/m1-state/" | relative_url }}) -- N booleans is 2^N states, most illegal |
| A function with 2+ boolean parameters | grep `fun .*Boolean.*Boolean` | a sealed type; the flags are probably mutually exclusive |
| Nullable fields meaningful only in some statuses | a `status` field plus `T?` siblings | [State]({{ "/design_patterns/m1-state/" | relative_url }}) -- make illegal states unconstructible |
| A foreign type's vocabulary inside your domain | grep the third-party type name | [Adapter]({{ "/design_patterns/m3-adapter/" | relative_url }}) -- it should appear in one file |
| The same 4-step sequence in several callers | grep any distinctive middle step | [Facade]({{ "/design_patterns/m3-facade/" | relative_url }}) |
| A class with 6+ constructor parameters used together | count parameters, then read the body | [Facade]({{ "/design_patterns/m3-facade/" | relative_url }}), or the parameters are unrelated and it is doing two jobs |
| Recursion re-derived per operation | grep for branching on node type | [Composite]({{ "/design_patterns/m4-composite/" | relative_url }}) + one function per operation |
| Config branching that must outlive the build | rules in code that users ask to change | [Interpreter]({{ "/design_patterns/m4-interpreter/" | relative_url }}) |
| A class name made of two vocabularies | `PushShiftNotifier`, `JsonHttpClient` | [Bridge]({{ "/design_patterns/m6-bridge/" | relative_url }}) -- if both halves grow |
| A feature users would undo and cannot | ask support, not the code | [Command]({{ "/design_patterns/m5-command/" | relative_url }}) + [Memento]({{ "/design_patterns/m6-memento/" | relative_url }}) |

**Right-size the prescription.** The most common audit error is not missing a smell -- it is answering a small one with a large pattern. A `when` repeated in seven files might need one extension function, not a Strategy hierarchy. Gate 3 from [Foundations]({{ "/design_patterns/m0-fundamentals/" | relative_url }}) is the check: *does the code get shorter, or at least easier to follow?*

## Direction 2: patterns that should come out

This is the pass almost nobody runs, and it finds as much as the first.

**Interfaces with one implementation and no test double.** The rule from Foundations, made into a query: list every interface, count implementations, count test fakes. One and zero means you built a seam and never used it.

Two legitimate exceptions, and you must check both before deleting anything:

- **The interface crosses a module boundary in the inverting direction** -- declared in the module that *uses* it, implemented in the one that provides it. That is dependency inversion; the single implementation is the point.
- **It exists for a fake you are about to write.** Fine -- write the fake now, or drop the interface until you do.

**`object` declarations with mutable state or I/O.** The Singleton audit: list every `object`, and for each ask whether it holds mutable state or touches time, randomness, or the network. A stateless namespace of pure functions is fine and common. Anything else is a hidden dependency that no signature mentions.

**Decorator stacks nobody can order.** If a wrapper chain exists and no document says what each layer should observe, the order is accidental. Write down the intent per layer; where you cannot, the layer is probably in the wrong place.

**Abstract factories with one family.** A `Environment`/`Provider` interface with a single implementation is the first item again, wearing a creational hat.

**Patterns applied to closed sets.** A Strategy over an enum that has had the same three values for two years is a `when` that grew a hierarchy.

## The one-page decision guide

Keep this; it is the actual output of the whole series.

**Before adding any pattern, three gates -- all must pass.**

1. Has the change **actually arrived twice**? Once is a coincidence.
2. Is there **exactly one clear axis** of variation? Two means consider Bridge. Unclear means it is too early.
3. Does the code get **shorter, or at least easier to follow**, afterward?

**Then pick by the question you are answering:**

| The question | The answer |
|---|---|
| Something varies, and the caller knows which | Strategy -- a function type first |
| Something varies, and the object itself decides when | State -- sealed, with the data per state |
| The order is fixed, the steps vary | a higher-order function; a class only past 3 hooks |
| I need a different interface | Adapter |
| I need the same interface, plus behavior | Decorator -- `by` delegation |
| I need the same interface, with access controlled | Proxy |
| I need a simpler interface over many things | Facade -- or a module boundary, which is stronger |
| These objects must come from the same world | Abstract Factory -- or your DI component, which already is one |
| The structure is recursive | Composite, sealed, one function per operation |
| The rules must outlive the build | Interpreter |
| Several unrelated things react to one event | Observer -- `Flow` |
| Several related things have interlocking rules | Mediator -- and keep the domain rules out of it |
| Steps run in order and one may stop the rest | Chain of Responsibility |
| I need undo, queueing, or a replayable log | Command, with Memento for the irreversible parts |
| Two independent axes, both growing | Bridge |
| Thousands of near-identical objects, measured | Flyweight -- or a `value class` |

**And the Kotlin overlay.** Five patterns survive intact: **Abstract Factory, Facade, Mediator, Flyweight, Bridge**. Everything else has a language form that should be your first attempt, and the class form is what you fall back to when the language form stops fitting -- more than one operation, state to carry, a name that has to appear in a log.

## Doing the audit

An hour, in this order:

1. Run the queries from direction 1. Write down file and line, not impressions.
2. Run direction 2. Expect it to find more than you think.
3. Sort by the three gates. Most findings fail gate 1, and that is a result.
4. Take the top one. Only the top one.

Then stop. **An audit that produces a twelve-item backlog produces nothing**; one that produces a single change you make this week has paid for itself.

## What the series was actually for

Not the twenty-three names. Three things:

**A vocabulary for design conversations.** "That is a Decorator, and the order matters" is faster than three paragraphs, and it is precise.

**The ability to read libraries.** The [previous session]({{ "/design_patterns/m7-in-the-wild/" | relative_url }}) is the transferable skill: six passes, and the question of what each pattern bought.

**A calibrated sense of when not to.** Every page here had a *Don't use it when* section for a reason. The failure mode of learning patterns is applying them, and the gates exist because the most common outcome of a design-patterns course is worse code for about six months.

If the series worked, the test is not that you can name Bridge. It is that you notice `PushShiftNotifier` and pause.
