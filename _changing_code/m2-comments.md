---
title: "Comments"
order: 7
module: "M2"
module_title: "Names and Boundaries"
session: "4-5"
intent: "Clean Code says a comment is a failure. A Philosophy of Software Design says a comment is design. They cannot both be right."
status: "complete"
excerpt_separator: "<!--more-->"
categories:
  - Code You Can Change
tags:
  - "Comments"
source: "Martin vs Ousterhout"
---

The one place in this series where two respected sources contradict each other flatly. Staging the argument is more useful than picking a side, because the disagreement turns out to be about *which* comments.

<!--more-->

## Position A: a comment is a failure

Martin, *Clean Code*:

> The proper use of comments is to compensate for our failure to express ourselves in code.

The argument: a comment is a second copy of the truth, and copies drift. Code is verified continuously -- by the compiler, by tests, by production. A comment is verified by nobody. So every comment is a future lie with a delay fuse, and the correct response to "this needs explaining" is to **change the code until it does not**: rename the variable, extract the block into a function whose name is the explanation.

He is not against all comments -- legal headers, intent, warnings of consequence, TODOs, and public API docs are all listed as legitimate. But the default is that a comment is a smell.

**What is right about this.** Every comment that restates the code is pure cost. Every comment that has drifted is worse than nothing, because it is believed. Anyone who has debugged against a stale comment knows the failure is real and expensive.

## Position B: a comment is design

Ousterhout, *A Philosophy of Software Design*, is blunt in the other direction. His claim is that **some information cannot be expressed in code at all**, and that pretending otherwise is how abstractions fail.

His central case is the *interface comment*: what a caller must know in order to use a method without reading its body. Preconditions. Side effects. What happens on failure. Thread safety. What it deliberately does **not** do.

> If you have to read the code of a method to use it, there is no abstraction.

And the sharper half of his position: comments are not documentation written afterwards -- they are a **design tool used first**. Write the interface comment before the implementation. If the comment comes out long and full of exceptions, the interface is wrong, and you have found that out before writing any code.

**What is right about this.** Code says *what* it does. It does not say why this order, why this constant, what breaks if you call it twice, or which of three plausible behaviours on failure was chosen deliberately. That information exists, it is needed, and there is nowhere else to put it.

## Where they actually collide

Three specific disagreements, and they are all downstream of one choice.

**Extract a function, or write a comment?** Martin: extract, and let the name be the explanation. Ousterhout: that produces shallow methods, and **each extraction adds an entry to an interface** -- the [depth]({{ "/changing_code/m2-deep-modules/" | relative_url }}) argument from the previous post. A comment costs a reader nothing if they do not need it; a method costs everyone who reads the class.

**Is self-documenting code possible?** Martin: mostly yes. Ousterhout: no, and treating it as possible is how the *why* gets lost. A function name is one phrase. A comment can be a paragraph, and some constraints need a paragraph.

**How long should a function be?** Martin: very short. Ousterhout: long is fine if it is coherent, and chopping a cohesive routine into fragments scatters something that was one idea.

They are the same disagreement seen three times: **Martin optimizes for code that reads without help; Ousterhout optimizes for readers who should not have to read the code at all.**

## The resolution

The dispute dissolves once you separate two kinds of comment, and this is the practical output of the session.

**Implementation comments** -- inside a function body. Here **Martin is mostly right.** Nearly all of these are replaceable by a better name, a clearer structure, or an extracted expression. `// increment the counter` is indefensible. A comment marking sections inside a 200-line function is a message about the function, not about the code.

**Interface comments** -- above a function, class or module. Here **Ousterhout is right**, and the point is not stylistic. There is genuinely no other container for:

- preconditions and postconditions -- the [contract]({{ "/changing_code/m1-lsp/" | relative_url }}) that LSP requires and no codebase writes down
- **why**, when the obvious alternative was rejected for a reason
- what the caller must **not** do
- thread-safety and performance characteristics
- **exception behaviour** -- and in Kotlin this is acute, because there are no checked exceptions, so *nothing but a comment or a test states what a function throws*

The test that decides any individual case:

> **Is this information already somewhere in the code?** If yes, the comment is a duplicate and will drift. If no, the comment is the only container it has.

And a second one, for level: **a comment should sit at a higher level of abstraction than the code beneath it.** At the same level it is a restatement; below it, it is noise.

## The technique

**Delete it and see.** Remove a comment and ask what a reader loses. Nothing lost means it should not have existed.

**Move what can be executed into code.** This is the strongest defence against drift, because executable statements are verified.

```kotlin
// Before: the constraint lives in prose
/** The list must be sorted by date before calling this. */
fun apply(cells: List<ApplyCell>) { ... }

// After: the constraint is checked
fun apply(cells: List<ApplyCell>) {
    require(cells.zipWithNext().all { (a, b) -> a.date <= b.date }) {
        "cells must be sorted by date"
    }
    ...
}
```

`require` and `check` turn a precondition from a claim into a fact. So does a [contract test]({{ "/changing_code/m1-lsp/" | relative_url }}) with a backtick name -- a test called ``rejects a swap that breaks the 11 hour rest rule`` is documentation that fails the build when it becomes false.

**The rule that follows: anything you could write as a comment and could also write as code, write as code.** What remains after that filter is the part that genuinely needed prose -- and it is a much smaller set than most codebases have, and a much larger set than zero.

**Write the interface comment first.** Ousterhout's method, and it costs nothing to try. If the comment needs three "except when" clauses, you have found a design problem while it is still free to fix.

## What this looks like when it is going well

Comment density on its own says nothing -- but it becomes informative alongside two other counts. From a real Kotlin codebase of about 32,000 lines:

| Measure | Value |
|---|---|
| Comment ratio | 17.6% |
| `TODO` / `FIXME` / `HACK` | 2 |
| Commented-out code | 1 |

Seventeen percent is high. Taken alone it would look like over-documentation. Taken with the other two rows it reads differently: **almost no rotting markers and almost no commented-out code**, which means the comments are being maintained rather than accumulated.

And a sample of what those comments contain:

```kotlin
// date1904="0" is the default; stated explicitly because the whole serial scheme depends on it.
// ...would otherwise STEAL key focus
// A two-syllable Hangul word is a person only when the sheet's people axis says so
```

Every one of those is a *why* or a non-obvious constraint. None restates the code. **That is the Ousterhout position applied well**, and it is worth saying out loud, because the default advice -- "you have too many comments" -- would be exactly wrong here.

The live risk for a codebase in this state is not absence. It is **drift**: at 17.6%, some of those statements will quietly stop being true. Which is why the move that matters is the one above -- converting every claim that *can* be executable into `require`, `check`, or a named test, and leaving prose only where prose is the only option.

## When to stop

**Do not run a comment cleanup.** Deleting comments in bulk deletes the good ones too, and the good ones are the irreplaceable ones.

**Fix comments where you are already working.** A stale comment found while changing a function is cheap to fix; a sweep is not.

**Never leave commented-out code.** Version control exists. This is the one absolute rule here, and it is the only one both sources agree on.

## Module M2: what boundaries are made of

Three posts, one idea. A boundary is only real if a caller can work on one side of it without reading the other, and this module is the three things that decide whether that holds:

- **[Names]({{ "/changing_code/m2-naming/" | relative_url }})** -- what the caller reads first, and the only part nothing verifies
- **[Depth]({{ "/changing_code/m2-deep-modules/" | relative_url }})** -- how much stays on the far side of the boundary
- **Comments** -- the part of the contract that no construct in the language can hold

They compose into one test for any module you own: **can someone use this correctly from the signature, the name, and the interface comment alone?** If they must open the body, the boundary is decorative -- whatever the diagram says.

## Exercise

Take the file you changed most recently and classify every comment as *implementation* or *interface*.

Then for each interface comment ask the conversion question: **could this be a `require`, a `check`, or a named test instead?** Convert one. That single change turns a claim nobody verifies into one the build verifies -- and it is the cheapest anti-drift move available.
