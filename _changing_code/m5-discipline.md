---
title: "Refactoring Is Not Rewriting"
order: 14
module: "M5"
module_title: "Refactoring"
session: "10-11"
intent: "One rule has appeared in every module of this track. This is why it is the only one that does not bend, and what you lose the moment it does."
status: "complete"
excerpt_separator: "<!--more-->"
categories:
  - Code You Can Change
tags:
  - "Refactoring Is Not Rewriting"
source: "Fowler"
---

Never change behavior and structure in the same commit. It has turned up in four modules so far, always as an aside. It deserves its own session, because the reason it matters is not tidiness and the failure it prevents is not hypothetical.

<!--more-->

## The pain

A bug ships. You find the commit, run `git revert`, and discover that the commit also moved four hundred lines into three new files. Reverting the bug means reverting the reorganisation -- and everything merged on top of it now conflicts.

Or: `git bisect` narrows a regression to one commit, you open it, and it is eight hundred lines of "refactor and fix." Bisect did its job perfectly and told you nothing, because the unit it isolated is not a unit.

Both are the same failure, and it happened at commit time, weeks earlier, in about four seconds of judgement.

## The claim

Fowler's definition is doing work that people read past: refactoring is **by definition** behavior-preserving. Code that changes what the program does is not refactoring done badly -- it is not refactoring.

He makes the practical version vivid with the **two hats**. You are either adding function or refactoring, never both, and you switch deliberately and often. The hats are not a metaphor about focus. They are about what a commit **claims**.

> A commit labelled `refactor` is a claim: *nothing observable changed here, so you may read this diff structurally rather than semantically.*

That claim is the entire value of the label. It licenses a reviewer to skim, a revert to be safe, a bisect to be decisive. Make the claim falsely and you have not just mislabelled a commit -- you have poisoned the mechanism that the label exists to power.

## Four things you lose, precisely

**1. Revert granularity.** A mixed commit can only be reverted whole. You wanted to undo one behavior; you also undid structure that later work depends on. In practice this means the revert is abandoned and someone writes a forward fix under time pressure, which is the worst moment to be writing anything.

**2. Bisect resolution.** `git bisect` is only as precise as your smallest commit. Mixed commits are exactly the ones large enough to matter and ambiguous enough to be useless when bisect lands on them.

**3. Review attention -- and this is the strongest one.** A reviewer opening a 400-line diff of moved code reads carefully for about forty lines, confirms the pattern is "same code, new file," and starts scrolling. That is not laziness; it is a correct response to a diff that has announced itself as mechanical. **The behavior change on line 312 is invisible, and it is invisible *because* the rest of the diff was honest.** Mixing does not make the change harder to see than average -- it makes it harder to see than if you had buried it in a random feature commit.

**4. The characterization-test guarantee.** From [M3]({{ "/changing_code/m3-characterization-tests/" | relative_url }}): pins that pass before and after prove that structure moved and behavior did not. That proof holds only if the commit did not *also* change behavior. A mixed commit does not weaken the guarantee; it voids it, and it does so silently, because the pins were updated in the same commit to match the new behavior and are all green.

Losses 1--3 are inconveniences. Loss 4 is the one that turns the whole of M3 and M4 into theatre.

## How often it actually happens

Measured on the same repository as the [previous post]({{ "/changing_code/m5-refactoring-catalogue/" | relative_url }}) -- 65 commits labelled `refactor` over three months, one author, no reviewer:

- **12 of 65 (18%)** have subject lines that fit no refactoring name at all.
- **14 of 65 (22%)** modified files under a test source set.

The second number needs care, because touching tests in a structural commit is not automatically a violation. A package rename that updates symbols in 33 test files is mechanical and correct. Moving a test alongside the source it covers is correct.

The violation is narrower and mechanically detectable: **a commit labelled `refactor` in which an existing test's expected value changed.** If the structure moved and the behavior did not, no assertion needed to move.

Three examples from that set, verified by reading the diffs:

| Commit subject (paraphrased) | What the test diff shows |
|---|---|
| "apply the rest-period rule by default" | A test named *…GatedBySpecialException* **deleted**, replaced by *…ListedForDisplay*. A regulatory constraint's default flipped from opt-in to always-on |
| "anonymous = local-only" | The server fake rewritten from stateless to stateful; new tests for a new auth model; an architecture decision record revised |
| "simplify the calendar feed" | A whole privacy-scope feature removed, including two server routes and their tests |

Every one of these is a real, deliberate, well-reasoned product decision. **The commit bodies are honest and detailed** -- they describe exactly what changed and why, with dates and decision references.

Only the prefix is wrong.

That is the interesting part. This is not a discipline failure in the sense of carelessness; the author knew precisely what they were doing and wrote it down. The failure is treating the prefix as **description** when it is a **contract**.

## Why it happens, and the fix that is not willpower

The mechanism is always the same, and it is sympathetic.

You start a refactoring. Halfway through, you see that the behavior is wrong -- and now you are holding the file, you understand it completely, and the fix is one line. Stopping to commit the half-finished refactoring, fixing the bug in a second commit, and then finishing feels like bureaucracy performed for an audience of nobody.

**Working solo removes the only force that reliably corrects this**, because the cost lands on a future person who is also you, months later, under different pressure.

Telling yourself to be more disciplined does not work. Two habits do:

**Commit the refactoring *before* the change, not after.** The Beck line "make the change easy, then make the easy change" is usually read as advice about order of work. Read it as advice about order of **commits**. The refactoring commit is the one whose behavior you can still prove is unchanged, because you have not made the change yet. Commit it there, while the proof is free.

**Use `git add -p` when you notice mid-flight.** You do not have to unwind the work. Stage the structural hunks, commit as `refactor`, stage the rest, commit as `fix` or `feat`. It takes a minute and the history is correct.

And a detector you can run, for the times neither habit fires:

```bash
# refactor commits whose diff modifies an existing assertion
git log --pretty='%H %s' | grep ' refactor' | cut -d' ' -f1 | while read h; do
  git show "$h" -- '*Test.kt' | grep -qE '^-\s+assert' && git log -1 --oneline "$h"
done
```

A deleted or modified `assert` line under a `refactor` label is a behavior change wearing the wrong hat. Not every hit is a violation -- but every violation is a hit.

## The larger version: rewriting

The same rule, scaled up, is the argument about rewrites.

Joel Spolsky's [*Things You Should Never Do, Part I*](https://www.joelonsoftware.com/2000/04/06/things-you-should-never-do-part-i/) (2000) is the famous statement of the case against, built on Netscape's rewrite. The essay is twenty-six years old, frequently overstated, and frequently strawmanned -- but its core observation has not aged:

> **Old code is not bad code. It is code with the bug fixes in it.**

Every strange branch in a mature function is, statistically, someone's field report. A rewrite discards the report and keeps only the summary, and the summary was written by people who did not know which branches mattered.

The connection to this session is exact. **A rewrite is a behavior change with no pins.** You cannot demonstrate equivalence, because there is nothing to run against both versions. So you are not preserving behavior; you are re-deriving it from your current understanding and hoping the delta is small. Users find the delta.

The incremental alternative is Fowler's [**Strangler Fig**](https://martinfowler.com/articles/2024-strangler-fig-rewrite.html): stand the new implementation beside the old, route one capability at a time across, and delete the old path only after the new one has served real traffic. What it buys, in this session's terms, is that **each migration step is small enough to pin** -- you can run both and compare. It converts an unverifiable rewrite into a sequence of verifiable changes.

**When a rewrite is actually the right call**, which is less often than people want and more often than the essay implies:

- **The requirements changed**, not the code quality. You are not reproducing behavior, so there is nothing to preserve and the argument does not apply.
- **The platform is gone.** A dead runtime, an unsupported framework with a security hole and no upgrade path.
- **It is small enough to re-specify completely.** The honest test is one question: *can you state what this does precisely enough to verify a replacement?* If yes, a rewrite is a well-defined task. If no, a rewrite will silently drop behavior nobody wrote down -- and "nobody wrote it down" is the definition of the [legacy code]({{ "/changing_code/m3-legacy-definition/" | relative_url }}) this track is about.

Note what is missing from that list: *the code is bad*. Bad code with known behavior is the cheapest thing in software to improve incrementally. It is only expensive when nobody is willing to do it in small steps.

## When to stop

**Do not split commits past the point of meaning.** The rule is one hat per commit, not one line per commit. A refactoring that genuinely requires eleven files to move together is one commit.

**Do not let the rule block a one-line fix.** If you find a bug mid-refactor, fix it -- in its own commit, before or after. The rule is about separation, never about deferral.

**Do not retro-label.** A commit that did both is not fixed by renaming it. If it has not been pushed, split it. If it has, leave it and note the behavior change in whatever place your project records decisions.

**Stop calling a rewrite a refactoring.** This is the cheapest honesty available and it changes how a plan is evaluated. "We will refactor the sync layer" invites approval. "We will rewrite the sync layer, and we cannot verify equivalence" invites the right questions.

## Module M5: refactoring

- **[The catalogue]({{ "/changing_code/m5-refactoring-catalogue/" | relative_url }})** -- 72 named refactorings, six that carry the work, and the most common one is deletion
- **Discipline** -- the label is a contract about reviewability, and a rewrite is the same violation at project scale

Together they are two halves of one idea: **refactoring is valuable exactly to the degree that its behavior-preservation can be relied on.** The catalogue supplies moves whose mechanics preserve behavior; the discipline is what makes that preservation legible to everyone downstream -- a reviewer, a bisect, a revert, and the pins from M3.

Remove the discipline and the moves still work, but nobody can tell, which in a codebase is the same as them not working.

Next is **M6**, where the whole track gets pointed at one real file.

## Exercise

Run the detector above on your repository. Then take the most recent hit and answer three questions:

1. **Could this commit have been reverted cleanly** if the behavior change in it turned out to be wrong?
2. **What did the `refactor` label tell a reader** -- and was it true?
3. **Where would the split have been?** Find the line. It is usually obvious in hindsight, which is the point: it was obvious at the time too, and the cost of acting on it was about one minute.

If you get zero hits, check that your commits are labelled at all. An unlabelled history is not disciplined; it is unmeasured -- and everything in this post is invisible in it.
