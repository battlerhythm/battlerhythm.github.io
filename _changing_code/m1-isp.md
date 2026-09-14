---
title: "Interface Segregation"
order: 4
module: "M1"
module_title: "SOLID, Minus the Two You Know"
session: "2-3"
intent: "A client should not be forced to depend on methods it does not use. The cost shows up first in the tests."
status: "complete"
excerpt_separator: "<!--more-->"
categories:
  - Code You Can Change
tags:
  - "Interface Segregation"
source: "Martin"
---

The smallest of the five, and the only one you can settle with a spreadsheet.

<!--more-->

## The problem

Here is a shape that turns up in almost every codebase that has a persistence layer. Names changed; the numbers are from a real project.

```kotlin
interface WardStore {
    suspend fun ward(id: WardId): Ward?
    suspend fun createWard(...): Ward
    suspend fun members(id: WardId): List<Member>
    suspend fun addMember(...)
    // ... 39 methods in total
}
```

Now count what each caller actually touches:

| Client | Methods used |
|---|---|
| `WardInvite` | 7 / 39 |
| `RequestInbox` | 5 / 39 |
| `Share`, `Profiles`, `Notifiers`, `Calendar`, `Billing` | 3 / 39 each |
| `Auth` | 2 / 39 |
| `RosterImport` | 1 / 39 |

Nine clients. Not one of them needs more than seven methods, and every one of them compiles against thirty-nine.

**The interface was not designed for its clients. It was transcribed from its implementation.**

## The claim

> Clients should not be forced to depend on methods they do not use.

Martin's original case was Xerox printer software: one enormous `Job` type covering print, staple and fax, so that a change for any of them forced a recompile and redeploy of all of them.

Fowler has a sharper pair of names for the two shapes:

- a **header interface** is the public surface of a class, copied. It exists because someone needed a seam and extracted every method.
- a **role interface** is what one client needs, named after the role. `Iterable` is a role. `Comparable` is a role.

**ISP restated: prefer role interfaces.** And the way `WardStore` got to thirty-nine methods is the standard route -- a class was extracted to an interface for testability, mechanically, method for method.

## The objection

**The original motivation is largely gone.** Recompilation and redeployment were the stated cost in the 1990s, under static linking and overnight builds. Incremental compilation and dynamic linking removed most of it. If you sell ISP on build times you will lose the argument, correctly.

What survived is different and better: **cognitive load and the cost of substitution.** A client that depends on thirty-nine methods is a client whose contract nobody can hold in their head, and whose test double is a chore.

**Segregation has its own failure mode.** Forty single-method interfaces are worse than one fat one -- that is Ousterhout's shallow-module problem, and the classitis warning from [SRP]({{ "/changing_code/m1-srp/" | relative_url }}) applies unchanged. Interfaces are things to learn; more of them is not free.

**"Unused by whom?"** Segregation is relative to clients, and clients differ. Five kinds of client could justify five interfaces -- and then every implementation implements all five, and you have added types without removing dependencies. **ISP creates pressure to multiply interfaces with clients, and that pressure needs a limit.**

The limit is the one this series keeps returning to: does the result get easier to follow? Splitting thirty-nine methods into nine role interfaces is not obviously better than splitting them into three.

## The technique

**Build the usage matrix.** This is the whole method, and it is mechanical enough to script:

```bash
# for each method of the interface, which files call it?
for m in $(grep -oE '^\s+(suspend )?fun [A-Za-z0-9_]+' Store.kt | awk '{print $NF}'); do
  n=$(grep -rl "\.$m(" src/ | wc -l)
  printf "%3d  %s\n" "$n" "$m"
done | sort -n
```

Two readings come out of it:

- **Methods with exactly one caller** are candidates to move to a role interface with that caller -- or out of the interface entirely.
- **Clients that cluster** -- three clients using the same four methods -- name a role. That cluster is the interface you should have written.

**Then let the test doubles arbitrate.** Writing a fake is the most honest measure of an interface's size, because a fake must implement every method whether the test needs it or not. If a fake requires twenty methods of `TODO()`, you have not written a test helper; you have written a report on the interface.

That is why the cost of a fat interface shows up **in the tests first**. Production code can ignore methods it does not call. A test double cannot.

## In Kotlin

**The smallest role interface has no interface at all.**

```kotlin
// instead of: class RosterImport(private val store: WardStore)   // 39 methods, uses 1
class RosterImport(private val membersOf: suspend (WardId) -> List<Member>)

RosterImport(store::members)
```

One dependency, one method, no new type, and a test passes a lambda. For the clients above that use one to three methods, this is usually the right answer -- and it explains why Kotlin codebases need ISP less often than Java ones: **the function type is the role interface, already named by its signature.**

**`fun interface` when the role deserves a name.**

```kotlin
fun interface MemberLookup {
    suspend fun members(ward: WardId): List<Member>
}
```

SAM conversion means callers still pass a lambda, and the role now appears in stack traces and dependency lists.

**Compose interfaces rather than splitting classes.**

```kotlin
interface WardReader { suspend fun ward(id: WardId): Ward? ; suspend fun members(id: WardId): List<Member> }
interface WardWriter { suspend fun createWard(...): Ward ; suspend fun addMember(...) }
interface WardStore : WardReader, WardWriter
```

Existing code keeps `WardStore`. New clients take the narrower one. **The migration is additive, which matters -- a segregation you can do without touching callers is one you will actually do.**

The standard library is the reference implementation of this idea: `Iterable` has one method, `Collection` adds a few, `MutableCollection` adds mutation, and a client asks for the smallest one that works.

## In the wild

- **`Iterable` / `Collection` / `List` / `MutableList`** -- role interfaces stacked, each client asking for the least it needs
- **`Comparable`, `Comparator`, `AutoCloseable`** -- single-role interfaces that have survived decades unchanged
- **`java.sql.Connection`** (over 50 methods) and `ServletRequest` -- the counterexamples, both extracted from implementations
- **Ktor's `ApplicationCall`** -- a large interface that is arguably fine, because there is one kind of client

## When to stop

**One client means no segregation.** The principle is about plural clients; with one, the interface is already a role interface by construction.

**If a function type does it, do not write an interface.**

**Segregate additively.** Extract the narrow interface, make the fat one extend it, migrate clients when you touch them. A rewrite of nine call sites to prove a principle is the failure mode this series warns about.

## Module M1: the five as one question

Two of the five were covered in the [patterns series]({{ "/design_patterns/m0-fundamentals/" | relative_url }}): **OCP** is what almost every design pattern is a recipe for, and **DIP** is "program to an interface" at architecture scale. With SRP, LSP and ISP added, the set reads as five angles on a single question:

> **When a change arrives, how far does it spread?**

| | What it separates | The question it answers |
|---|---|---|
| **SRP** | the *sources* of change | who can ask for this to change? |
| **ISP** | the *receivers* of change | who is forced to care that it did? |
| **LSP** | the *substitution points* | does swapping an implementation leak? |
| **OCP** | extension from modification | can I add without editing? |
| **DIP** | the *direction* of dependency | does the arrow point at the abstraction? |

SRP and ISP are the same cut made from opposite sides -- SRP says a module should not serve two actors, ISP says a client should not depend on what a second actor asked for. They meet in the middle at `WardStore`: thirty-nine methods because the ward domain has several actors, and nine clients depending on all of them because nobody wrote role interfaces.

**One honest caveat to close the module.** SOLID is a vocabulary, not a detector. Nothing here finds a problem -- the [measurement]({{ "/changing_code/m0-measuring-change/" | relative_url }}) from M0 finds it, SOLID gives it a name so it can be discussed, and the [legacy-code]({{ "/changing_code/m3-legacy-definition/" | relative_url }}) and refactoring modules are what actually change anything. Principles that are used as a detector produce codebases refactored on suspicion.

## Exercise

Pick the largest interface you own and build the usage matrix: methods down the side, clients across the top.

Then look for the clusters rather than the totals. **The interface you should have written is the group of methods that three clients use together** -- and if no such group exists, the interface is fine at thirty-nine methods and the problem is somewhere else.
