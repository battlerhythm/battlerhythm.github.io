---
title: "Names"
order: 5
module: "M2"
module_title: "Names and Boundaries"
session: "4-5"
intent: "The one part of the code the compiler never checks, and the cheapest thing to get right."
status: "complete"
excerpt_separator: "<!--more-->"
categories:
  - Code You Can Change
tags:
  - "Names"
source: "Martin"
---

The compiler checks types, arity, nullability and exhaustiveness. It does not check whether `cachePainted` means what you think it means.

<!--more-->

## The problem

A wrong name is worse than no name, and worse than a wrong comment.

Nobody trusts a comment. Everybody trusts a name -- it is the thing you read when you are scanning, and scanning is most of reading. **A name that lies gets believed**, and the belief propagates: the next person writes code against what the name implied, and now two places are wrong.

Consider a boolean called `cachePainted`. What is true when it is true? That something was painted? That a cache exists? That a first render completed from cached data? Only the third is right, and the only way to find out is to read the code -- **which is the failure, because the name existed to save you that.**

## The claim

This is the one place the two traditions agree, so take both.

**Martin:** names should reveal intent, be searchable, and avoid mental mapping. Do not make the reader translate.

**Ousterhout, and this is the sharper formulation:** *a name is an abstraction.* It creates a picture in the reader's head of what the thing is, and a good name creates a picture that is both **precise** (rules out what it is not) and **complete** (covers everything it is).

And his corollary, which is the most useful sentence in the subject:

> **If a variable or method is hard to name, that is a red flag about the design.**

Naming difficulty is not a vocabulary problem. It usually means the thing has no single coherent identity -- which is the [SRP]({{ "/changing_code/m1-srp/" | relative_url }}) finding arriving by another route.

He adds a second criterion most treatments skip: **consistency**. The same concept must always get the same word. An inconsistent vocabulary forces the reader to ask whether `fetch` and `load` differ, and to read both to find out.

## The objection

**Short names are not bad names.** `i`, `n`, `it` are correct in a three-line scope, and the older rule is better than "descriptive names": **name length should scale with scope size** (Kernighan and Pike). A loop counter called `currentIterationIndex` is worse, not better.

**The banned-suffix lists are overapplied.** `Manager`, `Helper`, `Util` get called smells, and then `WorkManager` and `ConnectivityManager` are perfectly good Android APIs. The suffix is not the problem. The problem is when the suffix is *the only information in the name* -- `DataManager` says nothing; `WorkManager` says it manages deferred work.

**The bilingual case is real and mostly unaddressed.** A product built for a Korean domain has terms with no English equivalent: 프리셉터 is not "mentor", 듀티 and 나이트킵 have specific meanings inside a ward's scheduling culture. Three options, all with costs:

- **Translate** -- readable to any engineer, loses precision, and diverges from what the customer says on a support call
- **Transliterate** (`preceptor`, `nightKeep`) -- keeps the mapping to the domain, reads oddly, and is stable
- **Keep the original** -- exact, and makes the identifier unsearchable from a non-Korean keyboard

There is no right answer, but there is a wrong practice: **choosing differently in different files.** Whatever the rule, it has to be one rule, and it belongs in a glossary that exists even for a team of one.

## The technique

Four checks, all cheap, in the order that finds the most.

**1. Predict, then read.** Take a function name, say aloud what it does, then read the body. Every mismatch is a naming defect, and you find them faster in someone else's code -- so do this on your own code from six months ago, which is the same thing.

**2. Grep for synonyms.** This is the one that produces numbers.

```bash
for w in fetch load get retrieve read; do
  printf "%s=%s " "$w" "$(grep -rhoE "fun $w[A-Z][A-Za-z0-9_]*" src/ | wc -l)"
done
```

A real result from a mid-sized Kotlin codebase:

```
fetch=10  load=38  retrieve=1  read=3
save=14   persist=4  upsert=4  write=1
delete=31 remove=4  drop=1
```

Four words for reading, four for writing, three for deleting. **Nobody decided this.** It accumulated, and every one of those minority spellings makes a reader stop and ask whether the difference is meaningful. The fix is not a mass rename -- it is picking the majority word, writing it down, and converging when you touch a file.

**3. Booleans should be questions.** `isLoaded`, `hasMembers`, `canEdit`. An adjective alone (`loaded`, `enabled`) is ambiguous about tense and subject; a participle with an unclear actor (`cachePainted`, `classGuessed`) is worse. If the question form reads badly, the flag is probably [encoding a state machine]({{ "/design_patterns/m1-state/" | relative_url }}) rather than a fact.

**4. Keep a glossary.** One file, domain terms, the chosen spelling for each. For a bilingual domain it is not optional.

## In Kotlin

**Promote names to types.** This is the language's strongest naming feature and it is underused.

```kotlin
fun assign(nurse: String, shift: String)   // swap the arguments: still compiles
```

```kotlin
@JvmInline value class NurseId(val raw: String)
@JvmInline value class ShiftId(val raw: String)

fun assign(nurse: NurseId, shift: ShiftId)  // swap the arguments: compile error
```

A `typealias` gives the name without the safety -- it is an alias, not a type, and the swap still compiles. **`value class` gives the name *and* makes the compiler check it**, at no runtime cost in the common case. Where a name matters enough to get wrong, make it a type.

**Named arguments put the name at the call site.** `RosterCell(changed = true, pending = false)` reads; `RosterCell(true, false)` does not. If a function takes two parameters of the same type, requiring names at the call site is a real defence.

**Backtick names make test names sentences.**

```kotlin
@Test fun `rejects a swap that would break the 11 hour rest rule`() { }
```

The test name is the specification. This is the one place where a very long name is right, because the scope is one function and the audience is a failure report.

**Extension functions create places to put names.** `nurse.displayName()` names an operation that would otherwise be an anonymous expression at a call site.

## In the wild

The standard library encodes contracts in name suffixes, consistently enough to rely on:

- **`toX` copies, `asX` wraps** -- the distinction the [Adapter]({{ "/design_patterns/m3-adapter/" | relative_url }}) post is built on
- **`first` throws, `firstOrNull` does not** -- the suffix is the error contract
- **`mapNotNull`, `filterIsInstance`** -- the name states the filter, so no comment is needed

That consistency is why you can guess a standard-library function name and be right. It is the payoff of Ousterhout's second criterion at library scale.

## When to stop

**Renaming is cheap but not free.** It touches every call site, which means review time and merge conflicts for other people's in-flight branches.

**Converge opportunistically.** Pick the winning word, record it, and rename only files you are already editing. A pull request whose whole content is `load` → `fetch` costs more attention than it returns.

**A name you cannot fix is a design you have not understood.** If nothing fits, stop naming and go back to what the thing is -- that is the red flag doing its job.

## Exercise

Run the synonym grep on your own codebase for read, write and delete verbs.

Then answer the question that matters: **for each minority spelling, is the difference meaningful?** If `upsert` means something `save` does not, it earns its place and belongs in the glossary. If it does not, you have found a name that costs every reader a moment and buys nothing.
