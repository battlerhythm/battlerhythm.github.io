---
title: "What Legacy Actually Means"
order: 8
module: "M3"
module_title: "Legacy Code I: Getting a Grip"
session: "6-7"
intent: "Feathers' definition has nothing to do with age: legacy code is code without tests -- which makes it a solvable problem."
status: "complete"
excerpt_separator: "<!--more-->"
categories:
  - Code You Can Change
tags:
  - "What Legacy Actually Means"
source: "Feathers"
---

Most definitions of "legacy code" are complaints wearing a lab coat. Feathers' definition is the only one that turns the phrase into something you can run a script against.

<!--more-->

## The pain

Every codebase has a file people route around. It works. It ships. Nobody wants to open it.

Ask why, and the answers are all aesthetic: *it's old, it's messy, it grew organically, it was written under deadline.* None of those is actionable. You cannot un-grow something organically. "Old" has no fix.

That is the real cost of the usual definition -- not that it is wrong, but that it **names no next move**.

## The claim

Feathers, *Working Effectively with Legacy Code*:

> To me, legacy code is simply code without tests.

The word does the work of an argument. He is not describing a mood; he is picking the single property that predicts whether you can change the code safely, and then refusing to include anything else.

Notice what the definition **deletes**:

- **Age.** Code written this morning with no test is legacy. Code from 2009 with a good test suite is not.
- **Style.** Ugly code with tests can be cleaned up incrementally, in safety. Beautiful code without tests cannot be touched without risk.
- **Authorship.** "The person who wrote this left" is irrelevant if the behavior is pinned.

And notice what it **adds**: falsifiability. "Is this legacy?" becomes a question with a mechanical answer. You can compute it.

## Why the definition earns its narrowness

The definition is doing something specific: it converts an **unfalsifiable aesthetic judgment** into a **measurable structural one**, and it does that by choosing a property that sits directly on the causal path to the thing you actually care about.

You care about: *can I change this without breaking something I don't know about?*

Tests are how you find out, immediately, that you broke something. Without them, the feedback loop runs through code review, staging, and production -- hours to weeks, with a human in every link. So "has no tests" is not a proxy for badness. It is the direct statement that **the feedback loop on change is long enough for fear to be rational**.

Once you say it that way, the fear stops being a character flaw and becomes information.

## The counterargument

The definition is popular enough that it rarely gets pushed on. It should be.

**It over-flags.** A forty-line pure function with no test is not a problem. Its inputs are values, its output is a value, you can read the whole thing in one screen, and if it breaks the failure is local and obvious. Calling it legacy is technically correct under Feathers' rule and useless in practice.

**It under-flags.** A module with a large test suite that is slow, flaky, or welded to implementation details is worse than untested code in one specific way: the tests are a **cost on every change** and a signal you have learned to ignore. A suite that fails for unrelated reasons trains people to force it green. Under the strict definition that module isn't legacy. It absolutely is.

So the honest refinement is one step more general:

> **Legacy code is code where you cannot get feedback fast enough to change it confidently.**

Tests are the dominant reason that feedback is missing, which is why Feathers' shorthand is right nearly all of the time. But keeping the general form in view stops two failure modes: chasing coverage on trivially safe code, and declaring victory because a coverage number is high.

## The measurement

The interesting consequence of a mechanical definition is that you can point it at a codebase and get an answer instead of an opinion.

Here is one, measured by module on a project a little over three months old -- Kotlin Multiplatform, one author, no inherited code at all:

| Module | Source | Tests | Test lines per source line |
|---|---|---|---|
| Domain core | 66 files / 7,614 lines | 42 files / 5,027 lines | **0.66** |
| Server | 43 files / 6,296 lines | 55 files / 7,985 lines | **1.27** |
| UI + stores | 85 files / 18,955 lines | 19 files / 2,901 lines | **0.15** |

Three modules, one repository, one person, written in the same quarter. The ratio spans **8x**.

This is the definition doing its job. Nothing here is old. Nothing was written by someone who left. Under any age- or authorship-based definition of legacy, this codebase has none. Under Feathers', the UI layer is legacy and the server is not, and that judgment is checkable rather than arguable.

It also survives the drill-down, which is where it gets uncomfortable. Inside that UI layer:

| File | Lines | Test files that reference it |
|---|---|---|
| Sync store | 1,047 | 2 |
| Auth store | 731 | 5 |
| Social store | 309 | 3 |
| Two sync engines | 47 + 36 | 0 |

**The biggest file has the second-thinnest coverage.** That is the normal shape, and it is the shape that makes the definition worth having: the file everyone routes around is measurably the file nobody pinned.

## The dilemma

Feathers states the trap plainly:

> When we change code, we should have tests in place. To put tests in place, we often have to change code.

That circularity is not a paradox to be admired; it is the actual subject of this module and the next. The whole technique set exists to get out of it -- **change as little as possible, as safely as possible, purely to make testing possible**, and only then make the change you came to make.

The two halves:

- **[Seams]({{ "/changing_code/m3-seams/" | relative_url }})** -- finding places where behavior can be substituted without editing the code you're afraid of
- **[Characterization tests]({{ "/changing_code/m3-characterization-tests/" | relative_url }})** -- pinning what the code does today, correctness not required

Both are deliberately unambitious. That is the point.

## Legacy is a region, not a codebase

The single most useful consequence of measuring rather than judging:

> **"Is this a legacy codebase" is the wrong question. Every codebase is a mix. The question is which regions, and whether the region you are about to change is one of them.**

The table above is not a verdict on a project. It is a map with one dark area, and the dark area happens to be where features get added. That is a work plan, not an indictment.

## When to stop

**Do not start a coverage campaign.** A ratio of 0.15 is not a task. Feathers' goal is never coverage; it is being able to make *today's* change without fear. Tests written in a sweep, away from any pending change, are written without knowing which behaviors matter -- and they are the ones that end up slow, coupled, and eventually ignored.

**Add tests where a change is arriving.** The pending change tells you which behaviors are load-bearing. That is information you do not have during a sweep.

**Do not test trivially safe code to improve the number.** The pure functions in a domain module are the cheapest thing to cover and the least valuable. Covering them moves the ratio and not the risk.

**Stop when the change you came for is safe.** Not when the file is clean.

## Exercise

Run the measurement on something you own. Two numbers, no tooling:

1. **Per module**: test lines / source lines. The spread between your highest and lowest module is more informative than any single figure.
2. **Per file, in your worst module**: for each of the ten largest files, count how many test files mention it by name. Sort ascending.

The top of that list is not a backlog. It is the answer to one question: **when a change lands in this file next week, what will catch it?**

If the answer is "review," you have found the region this module is about.
