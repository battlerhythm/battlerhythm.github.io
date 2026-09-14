---
title: "The Twenty You Actually Use"
order: 13
module: "M5"
module_title: "Refactoring"
session: "10-11"
intent: "The online catalogue lists 72 refactorings. Three months of labelled commits in one codebase used six of them -- and the most common one was deleting code."
status: "complete"
excerpt_separator: "<!--more-->"
categories:
  - Code You Can Change
tags:
  - "The Twenty You Actually Use"
source: "Fowler"
---

Fowler's catalogue is a reference work that gets read as a curriculum, which is the wrong use of it. The interesting question is not what is in it, but which entries survive contact with a real repository.

<!--more-->

## The catalogue problem

The [online catalogue](https://refactoring.com/catalog/) accompanying *Refactoring* lists **72** named refactorings, each with motivation, mechanics, and an example. It is a genuinely good book and a genuinely intimidating artifact, and it produces a predictable reaction: people skim it, retain *Extract Function*, and conclude the rest is for someone else.

That reaction is half right. The catalogue is **not a curriculum**. It is a reference for naming a move you are already making, and the value of a name is not breadth of vocabulary -- it is that a named move comes with **mechanics**: a numbered sequence of steps, each small enough to leave the code working.

So the honest question for a working engineer is narrower than the book: *which ones actually come up, and how do the mechanics change in a language with better tooling than 1999?*

## The measurement

One way to answer that empirically is to look at a repository where refactorings were labelled as they happened. Here is one -- roughly 50,000 lines of Kotlin, **1,184 commits over three months**, conventional-commit prefixes, one author:

| Prefix | Commits |
|---|---|
| `feat` | 428 |
| `fix` | 321 |
| `docs` | 201 |
| **`refactor`** | **65** |
| everything else | 169 |

Sixty-five labelled refactorings. Classifying them by subject line (approximately -- a keyword pass, first match wins):

| Category | Count | Share |
|---|---|---|
| **Remove dead code** | 18 | 28% |
| **Move / split file** | 11 | 17% |
| **Consolidate duplicate** | 10 | 15% |
| **Rename** | 6 | 9% |
| **Extract function / class** | 6 | 9% |
| **Change data structure** | 2 | 3% |
| *Fits no refactoring name* | 12 | 18% |

Six categories cover four fifths of the work. That last row is the subject of the next post and we will come back to it.

And one aggregate that is worth more than the breakdown:

> **65 refactoring commits, +12,295 lines and −12,840 lines. Net: −545.**

Three months of refactoring made the codebase *smaller*. Twenty-three of the sixty-five commits were net-negative on their own. This is worth sitting with, because the mental image most people have of refactoring -- extracting things, adding structure, more files -- predicts the opposite sign.

## The most common refactoring is deletion

Twenty-eight percent. More than move, extract, and rename combined would be if you took any one of them.

*Remove Dead Code* is a minor entry in the catalogue, half a page, no interesting mechanics. In practice it is the dominant activity, and the reason is structural: **features are added at a much higher rate than they are removed, and every removed feature leaves residue.** A screen deleted from the navigation graph leaves its composables. A flag removed leaves both branches. An abandoned approach leaves the abstraction that was built to accommodate it.

The commits bear this out -- a deleted inbox screen, a demo-gating path, a dual in-memory implementation retired once Postgres became the only backend, a heuristic excluded from production once the model replaced it.

**Two practical consequences.**

First, deletion is the refactoring with the best cost-to-benefit ratio available, and it is systematically under-scheduled because it does not feel like progress. Nothing gets faster, no new capability appears; the codebase just stops containing a thing that was lying to readers about what the system does.

Second, **dead code is the only category where the tooling genuinely helps you find the work.** "Unused symbol" inspections, coverage reports on integration runs, and a grep for a removed feature flag will all produce candidate lists. None of the other five categories has an equivalent detector.

## The six, with Kotlin mechanics

**1. Remove Dead Code.** Mechanics: delete, compile, run tests, commit alone. The one discipline that matters is *alone* -- a deletion mixed into a feature commit is unreviewable, because reviewers cannot tell whether the removed lines were dead or load-bearing.

**2. Move Function / Move Field / split a file.** Kotlin makes the file-level version cheaper than most languages, because **a file is not a unit of meaning**: no one-public-class-per-file rule, and top-level declarations move between files without touching any call site as long as the package is unchanged.

This is the safest large change available. Evidence from the same repository: a 2,654-line screen file split by screen mode, a 1,274-line data-access file split per store, a 1,139-line component file split by family. All three commit messages say "pure move," and the diffs confirm it.

**3. Consolidate Duplicate.** Six hand-rolled avatar glyph implementations to one; five copy-pasted local-wipe lambdas to one function; every screen header to one shared composable. The mechanics that matter are *order*: unify the call sites to a single shape **first**, in its own commit, and only then delete the duplicates. Doing it in one step means the diff contains both the unification and the deletion, and a mistake in either is invisible.

**4. Rename.** The IDE does this correctly and it is the refactoring people most often skip out of politeness toward existing code. Do not. From [M2]({{ "/changing_code/m2-naming/" | relative_url }}): a name is the only part of the code nothing verifies.

**5. Extract Function / Extract Class.** The famous one, and only ninth percent of real usage. The high-value instance is the one from [M4]({{ "/changing_code/m4-breaking-dependencies/" | relative_url }}): a **pure function buried inside an impure method** -- a fold, a classification, a derivation. Extracting that is worth more than three cosmetic extractions, because it converts untestable logic into testable logic in one move.

**6. Change Data Structure.** The rarest and the most valuable per instance, because it is the one that removes whole categories of bug rather than moving code around -- a nullable single value becoming a list, two sources of truth collapsing to one derived value, a name field deleted because a code was already the identifier. These are [M1's illegal-states]({{ "/changing_code/m1-srp/" | relative_url }}) work arriving as refactorings.

## Balance, not size, predicts risk

The useful heuristic to come out of the measurement, and it takes one line of shell to compute:

> **`|insertions − deletions| ÷ (insertions + deletions)`**

Near **0** means the change was mechanical -- lines moved, names changed, content preserved. Near **1** means lines were added or removed outright, which means content changed.

The distribution in that repository is stark. The package rebrand touched **144 files** with +580/−560: balance 0.018, and one of the safest commits in the history despite being the largest. A dead-method deletion was +0/−8: balance 1.0, and a commit that must be reviewed as a behavior question, not a structural one.

**Size is not the risk axis.** A 144-file mechanical rename is safer than a four-file commit that removes thirty-six lines and adds seven, and the balance number says so immediately. It is a good thing to glance at before reviewing a commit labelled "refactor."

## The counterargument

**The mechanics assume no tooling.** The book's step sequences were written for a world where *Extract Function* was a manual, error-prone operation. For the refactorings an IDE performs -- rename, extract, inline, move, change signature -- following the printed mechanics by hand is strictly worse than pressing the key, because the tool updates every reference and you will not.

The division that actually matters today:

| IDE does it correctly | You must do it by hand |
|---|---|
| Rename, Extract/Inline Function, Extract Variable, Move, Change Signature | Consolidate duplicate, change a data structure, remove dead code, split a file by concept |

Everything in the right column requires a judgment about *meaning*, which is why no tool does it. That column is the one worth studying; the left column is worth a keyboard shortcut.

**Extract Function is over-applied.** This is the [M2 collision]({{ "/changing_code/m2-comments/" | relative_url }}) again: extracting aggressively produces many small functions, each shallow, and the reader now needs twenty interfaces to understand sixty lines. Fowler is more enthusiastic about small functions than Ousterhout, and the catalogue's prominence makes *Extract Function* the default move when it should be one of six.

**Parts of the catalogue are language-shaped.** Several entries address problems Kotlin does not have:

| Fowler entry | In Kotlin |
|---|---|
| Encapsulate Field | Properties are already accessors -- no-op |
| Replace Temp with Query | Often just `val x get() = ...` |
| Replace Constructor with Factory Function | A `companion object` function, or default arguments |
| Introduce Parameter Object | `data class` -- cheap enough that the refactoring barely has mechanics |
| Replace Conditional with Polymorphism | Frequently the *wrong* direction; `sealed` + `when` is the [Kotlin answer]({{ "/design_patterns/m4-visitor/" | relative_url }}) |
| Separate Query from Modifier | Still good advice, no language help |

Reading the catalogue as a to-do list in Kotlin produces Java-shaped code, which is the same failure mode as [applying Feathers' techniques mechanically]({{ "/changing_code/m4-kotlin-final/" | relative_url }}).

**And the catalogue is not a canon.** The second edition dropped entries, added others, and switched the examples to JavaScript. It is one person's carefully organised experience, revised once. Treating any specific entry as authoritative is a misreading of what the book is.

## When to stop

**Stop when the change you came for is easier.** That is the entire purpose. "Make the change easy, then make the easy change" -- the refactoring is the first half, and it ends when the second half is ready.

**Do not refactor code you are not about to change.** Refactoring has a cost paid immediately (review, merge conflicts, re-learning) and a benefit paid only when someone next touches that code. If nobody is about to, the benefit may never arrive.

**Do not chain refactorings in one commit.** Each of the six above is a separate commit. A commit that moves a file *and* consolidates duplicates inside it cannot be reviewed, and cannot be reverted in half.

**Stop before the sixth cosmetic extraction.** If the last three refactorings made the code prettier and none of them made a pending change easier, the activity has drifted into decoration.

## Exercise

Run this against your own repository:

```bash
git log --pretty=format:'%s' | grep -c '^refactor'
git log --pretty=format:'%H %s' | grep '^.\{40\} refactor' | while read h rest; do
  git show --shortstat --oneline "$h" | tail -1
done
```

Two questions from the output:

1. **What is your most common refactoring?** If the answer surprises you, your mental model of where the work goes is wrong in a way worth knowing.
2. **What is the net line count?** Positive means you are adding structure faster than you are removing accumulation. That is not automatically bad, but if it has been positive for a year, look for the deletions nobody scheduled.

And if your commits are not labelled, that is the finding. You cannot measure what you did not distinguish -- which is exactly the subject of the next post.
