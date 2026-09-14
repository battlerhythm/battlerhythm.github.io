---
title: "Characterization Tests"
order: 10
module: "M3"
module_title: "Legacy Code I: Getting a Grip"
session: "6-7"
intent: "Tests that record what the code does, not what it should do. The first ones you write must not assert correctness -- that is the whole trick."
status: "complete"
excerpt_separator: "<!--more-->"
categories:
  - Code You Can Change
tags:
  - "Characterization Tests"
source: "Feathers"
---

The technique that resolves the legacy dilemma is a kind of test most people have never deliberately written, and its defining property sounds like a mistake: it does not check that the code is right.

<!--more-->

## The pain

You have a seam. You are ready to change the code. And you are stuck on a question nobody can answer:

**What is this supposed to do?**

There is no spec. The original ticket is three years old and describes a different feature. The person who wrote it is gone, or is you, eighteen months ago. Behavior has accreted: a special case for one customer, a workaround for a bug in a dependency that has since been fixed, an off-by-one that something downstream now depends on.

A normal unit test is unwritable here, because a normal unit test asserts *intended* behavior and you do not know the intent.

## The claim

Feathers' move is to stop asking.

> A characterization test ... documents the actual behavior of a piece of code.

Not the correct behavior. The **actual** behavior, including the parts that are wrong.

The algorithm is mechanical, and the first step is the one that feels illegal:

1. Write a test asserting something you are fairly sure is **false** -- `assertEquals(0, subject.compute(input))`
2. Run it. The failure message tells you the real value
3. **Change the test to expect that value**
4. Repeat until the behaviors you care about are pinned

You are not using the test runner as a judge. You are using it as an **instrument** -- the cheapest available probe into what the code actually does. The assertion is a question, and the failure message is the answer.

## Why "wrong on purpose" is the right default

Three reasons, and the third is the one that matters most in practice.

**You cannot assert intent you do not have.** Guessing at intent and asserting it produces a test that fails for the *only* reason that is uninformative: you guessed wrong. Now you must debug the test.

**A refactor needs a different question answered.** Refactoring means changing structure while preserving behavior. The safety net therefore has to answer *did behavior change?* -- not *is behavior correct?* A test that encodes what the code does today answers exactly that question and nothing else, which is why it is the right tool and a correctness test is not.

**Pinning a bug is how you make changing it deliberate.** This is the inversion worth internalizing. An unpinned bug can be fixed by accident, during a refactor, and nobody notices until something downstream that depended on it breaks in production. A **pinned** bug cannot be changed silently -- the pin fails, and someone has to decide, in that moment, whether the change is intended.

> The pin does not endorse the behavior. It makes changing the behavior a decision instead of a side effect.

## The counterargument

This technique is unusually well-liked, which means the objections get skipped.

**"You are codifying bugs as requirements."** The strong form of this is real. A pin written to unblock one refactor, left in place for two years and never revisited, becomes indistinguishable from a specification. New team members read it as intent. The bug is now a feature by accretion, and nobody ever decided that.

The defense is not "pins are fine." It is **labelling**: a pin on behavior you believe is wrong must say so in the test, with a date and a reason. An unlabelled pin is the thing the objection describes.

**"They are brittle."** Often true, and usually self-inflicted. A pin that asserts on call sequence, internal state, or the exact shape of a private structure fails on every refactor -- which makes it a tax on precisely the activity it was supposed to enable. Pins belong on **observable behavior at the boundary**: return values, posted payloads, persisted state. Not on how the code got there.

**"The golden-master variant does not scale."** Approval testing -- snapshot a large output, diff future runs against it -- is the same idea with worse ergonomics at size. A five-thousand-line approved blob means every change requires a human to eyeball a diff, and humans stop. The failure mode is universal and has a name: mass-accepting the new snapshot to get green. `jest -u` and its equivalents in every language.

**The scope limit.** Characterization tests are an **epistemic** tool, for code whose behavior you do not know. For code you fully understand, skip it and write the test you actually want. Reaching for characterization on code you wrote last week is ceremony.

## The technique, concretely

**Fake at the outermost boundary.** The instinct is to fake the nearest collaborator. The better move is to fake the *lowest* layer -- the raw HTTP interface, the raw file API -- and let everything above it run for real.

A harness I measured does exactly this: the fake implements the raw server interface, and the test then boots a **real** auth store and a **real** transport in front of it. The code under test therefore crosses the genuine production path, including the 401-retry funnel, and only the JSON per route is canned. Faking one layer higher would have made those tests pass while the real path stayed unexercised.

**Prefer fakes to mocks.** A mock pins *how* -- which methods were called, in what order. A characterization test wants *what* -- what came out, what got written. Mock-heavy pins are the brittleness objection in its most common form. A hand-written fake with a couple of counters gives you both, and the counters stay opt-in.

**Pin the boundary in both directions.** Not just the return value: also what the subject *sent*. A store that returns `true` on success and posts a malformed body is passing a weak pin.

**Label known-wrong behavior in the test.** Date, reason, and the fact that it is accepted rather than correct.

```kotlin
@Test fun refreshOn429KeepsGraphAndFlagsOffline() = runBlocking {
    // The transport collapses a 429 to null exactly like offline, so the store rides the
    // existing degradation path. The 'offline' mislabel is accepted for an abuse-only
    // event (2026-07-28 decision); pinned here so changing it has to be deliberate.
    ...
    assertTrue(store.offline)
    assertEquals(listOf("f1"), store.graph.friends)   // keep-last -- a throttle never wipes the graph
}
```

That comment is the difference between a pin and an accidental specification.

## The Kotlin translation

**The failure message is the instrument, so argument order matters.** `assertEquals(expected, actual)` -- reversed, the message reads backwards and the probe step becomes confusing exactly when you are relying on it.

**No mocking framework.** In Kotlin Multiplatform `commonTest` this is close to forced -- the usual JVM mocking libraries do not cover every target -- and the constraint pushes toward the better answer anyway. A fake is a class that implements the interface. It is more readable than the mock DSL it replaces.

**`data class` equality makes whole-object pinning cheap.** `assertEquals(expected, actual)` on a data class compares every field and prints a readable diff, so pinning a whole result is often better than pinning five fields. Fewer assertions, more coverage, and the failure names the field that moved.

**Determinism comes from the previous post.** A pin is worthless if the subject calls `Clock.System.now()` internally. This is where the [seam ladder]({{ "/changing_code/m3-seams/" | relative_url }}) pays off: extract the logic as a function that takes `now: Long`, and the pin becomes

```kotlin
private val now = 1_000_000L
```

one line at the top of the test class. Fourteen pins on one extracted derivation core, all deterministic, because the extraction put the clock in the signature.

**`runBlocking` for suspend subjects**, and keep the fake synchronous -- introducing concurrency into a harness you are using as a measuring instrument is a way to spend a day.

## The ordering that makes it work

The pins go in **before** the refactor, in their own commit, and they must pass against the unchanged code. That is the whole guarantee: if they passed before and pass after, structure moved and behavior did not.

This is visible in history when a team does it. In one repository the sequence over a single day reads:

```
test(shared): FakeServerApi characterization harness + store pins (plan 0.1)
refactor(shared): delete dead rotateInviteCode (plan 1.5)
refactor(shared): transport JSON decode helpers -- 54/62 sites migrated (plan 2.6a)
```

Phase 0.1 is the pins. Everything numbered after it is the work the pins made safe. A refactor commit that also *creates* its own safety net proves nothing, because the net was woven around whatever the code does after the change.

And the rule that keeps pins honest afterwards, from a comment in that same codebase: **the next intentional behavior change must update the pins in the same commit that declares the change.** Never in a refactor commit. That single rule is the M5 discipline -- separate behavior changes from structure changes -- stated from the test's side, and it is what stops pins from silently drifting into lies.

## When to stop

**Stop when the change you came for is covered.** Not when the class is covered. Pins are scaffolding for a specific change; scaffolding for a building you are not constructing is just clutter.

**Do not pin private behavior.** If you need to reach past the public surface to pin something, either it is not worth pinning or you have found the extraction the refactor should do first.

**Do not pin timing, ordering, or log output** unless the behavior under change is timing, ordering, or log output. These are the top three sources of flaky pins, and a flaky pin gets disabled, which leaves you with no net and the belief that you have one.

**Do not let a pin outlive its question.** After the refactor lands, a pin is either promoted to a real test -- renamed to state intent, with the known-wrong comment resolved -- or deleted. The ones that get neither are how the "codifying bugs" objection comes true.

## Module M3: getting a grip

Three ideas, one sequence:

- **[Definition]({{ "/changing_code/m3-legacy-definition/" | relative_url }})** -- legacy is a measurable property of a *region*, not an aesthetic judgment about a codebase
- **[Seams]({{ "/changing_code/m3-seams/" | relative_url }})** -- and a class is testable only at the level of its least substitutable dependency
- **Characterization tests** -- record what is, so that changing it becomes a decision

They compose into the escape from the dilemma: **find the dependency with no enabling point, give it one with the smallest possible change, pin the behavior that exists, and only then make the change you came for.**

What M3 does *not* cover is the case where the smallest possible change is still blocked -- where the dependency is a `final` class, a top-level `object`, or a constructor call buried in a method body. That is where Feathers' main technique set lives, and where Kotlin's defaults fight it. **M4.**

## Exercise

Take the class from the last exercise -- the one whose dependency table had blank rows.

Fill in the cheapest blank row, then write **one** characterization test, using the algorithm exactly: assert something false first, run it, read the failure, paste the real value in.

Then answer the question that decides whether the pin was worth writing: **could this value have changed during a refactor without anyone noticing?**

If yes, you just built a net. If no, delete it and pick a different behavior -- the ones worth pinning are the ones nothing else would catch.
