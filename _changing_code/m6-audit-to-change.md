---
title: "From Audit to a Shipped Change"
order: 15
module: "M6"
module_title: "Applying It"
session: "12-13"
intent: "The whole track pointed at one 1,047-line file -- including the part where the audit's top finding does not survive contact with the code."
status: "complete"
excerpt_separator: "<!--more-->"
categories:
  - Code You Can Change
tags:
  - "From Audit to a Shipped Change"
source: "Feathers"
---

Five modules of technique, applied end to end to one file. The useful part is not that it works -- it is the two places where the plan was wrong and the procedure caught it.

<!--more-->

## The target

One file selected the boring way: three independent methods pointed at it.

- A **pattern audit**, reading the code, flagged it as a coordinator that had absorbed domain logic
- A **co-change analysis** of the git history put it at the top of the "changes with everything" list
- A **[coverage measurement]({{ "/changing_code/m3-legacy-definition/" | relative_url }})** found it was the largest file in the thinnest-covered module

1,047 lines. 24 observable state properties. 48 public functions. Two test files mention it.

**Convergence from three methods is stronger evidence than any one of them**, and it is worth saying why: each method has a different blind spot. Reading finds structure and misses frequency. Co-change finds frequency and misses whether the coupling is legitimate. Coverage finds risk and misses whether anyone touches the code at all. When all three land on the same file, the overlap is the part none of them could be wrong about.

## Step 0: name the change, not the file

The first step is the one most likely to be skipped, and skipping it is how a refactoring becomes a rewrite.

> **"Refactor the sync store" is not a change. It is a mood.**

Everything in this track is justified by a *specific pending change* -- the seam exists to enable a test, the test exists to protect a change, the refactoring exists to make that change easy. Without one, there is no stopping condition, and [without a stopping condition you are rewriting]({{ "/changing_code/m5-discipline/" | relative_url }}).

So: two candidate changes, both small, both from the audit's priority list.

**Change A** -- a single `actionFailed` Boolean is written by six different code paths (create, rename, leave, delete, and two more through a public setter) and consumed by three different screens. The audit said: split it, because one flag cannot represent which action failed.

**Change B** -- three request-filing methods stamp `Clock.System.now()` into a request body before posting it, and nothing asserts on what is posted.

## Step 1: verify the finding still holds

The audit is weeks old and was written by reading. Before acting on it, check it.

**Change A did not survive.** Reading the code carefully turns up a documented interface comment:

> *WardScreen renders an inline notice and clears it on the next success or attempt.*

A sticky flag cleared on the next attempt is **correct** for a persistent inline notice. The audit's framing -- "this should be a one-shot event, because two consecutive failures lose the second" -- applies to transient notifications like a toast, and this is not one. The finding as stated was wrong.

This is the step people skip, and it is the whole reason the procedure has a step here. **An audit finding is a hypothesis with a timestamp.** Acting on one without re-checking is how a codebase acquires changes that were justified by a misreading nobody revisited.

**But the re-check turned up something better.** One call site reads:

```kotlin
onClick = { wardSync.clearActionFailed(); showRename = true }
```

A **defensive clear** before opening an unrelated dialog. Somebody, at some point, saw a stale notice appear where it did not belong and added a clear to suppress it. The other consumers do not have that clear.

That is a much more interesting finding than the original, and notice where it came from: not from reading the store, but from reading its **callers**. A shared mutable flag's bugs live at the consumers, and a defensive workaround at one consumer is the most reliable evidence that the flag leaks.

**Change B held up.** Nine direct clock calls, three of them in a request body. No further verification needed -- non-determinism is a property of the code, not a judgement about it.

## Step 2: the dependency table

For each change, list what stands between you and a test. Not the class's dependencies in general -- only the ones on the path of this change.

**Change A:**

| Dependency | Enabling point |
|---|---|
| Server transport | Constructor parameter |
| Analytics | Constructor parameter, defaulted |

Both present. **No seam work required.**

This is worth pausing on, because the instinct after four modules of technique is to apply technique. Another test file in the same package already constructs this store directly with a fake transport and asserts on decoded payloads. The seam was built months ago, for a different feature, and nobody noticed it generalised.

> **Check whether you need a seam before building one.** The cheapest rung on the ladder is the one already installed.

**Change B:**

| Dependency | Enabling point |
|---|---|
| Server transport | Constructor parameter |
| `Clock.System.now()` | **none** |

One blank row, exactly as [the seams post predicted]({{ "/changing_code/m3-seams/" | relative_url }}) -- *usually one or two, and usually a clock.*

## Step 3: the smallest seam that works

Rung 3, one default parameter:

```kotlin
class WardSyncStore(
    private val transport: ServerTransport,
    private val analytics: Analytics = NoopAnalytics,
    private val cache: KvCacheRepository? = null,
    private val now: () -> Long = { Clock.System.now().toEpochMilliseconds() },   // added
)
```

and inside the three methods, `Clock.System.now().toEpochMilliseconds()` becomes `now()`.

**Commit this alone.** It is a structure change with zero behavior change and zero call-site changes -- the [balance ratio]({{ "/changing_code/m5-refactoring-catalogue/" | relative_url }}) is near zero and the label `refactor` is honest. If anything later goes wrong, this commit reverts cleanly.

What was *not* done here matters as much. No interface was extracted. No clock abstraction was introduced across the codebase. No other call site was migrated. The seam is the minimum that unblocks this change, and the [stopping condition]({{ "/changing_code/m4-breaking-dependencies/" | relative_url }}) was reached before the work felt finished.

## Step 4: pin, before touching anything

Now the characterization tests, in their own commit, passing against the **unchanged** behavior.

For Change B, the pin is on what gets posted -- both directions of the boundary, as [M3 required]({{ "/changing_code/m3-characterization-tests/" | relative_url }}):

```kotlin
@Test fun fileApplicationPostsTheRequestWithTheCurrentClock() = runBlocking {
    val api = FakeServerApi().apply { onPost("/wards/w1/requests", "{}") }
    val store = WardSyncStore(connectedTransport(api), now = { 1_000L })
    store.refresh()

    assertTrue(store.fileApplication("w1", date, "D", "N"))
    val (path, body) = api.posts.last()
    assertEquals("/wards/w1/requests", path)
    val req = json.decodeFromString<Request>(body)
    assertEquals(1_000L, req.createdAt)      // pinned: the stamp comes from the injected clock
    assertEquals("D", req.currentCode)
    assertEquals("N", req.requestedCode)
}
```

For Change A, the pin is the question itself -- the one reading could not settle:

```kotlin
@Test fun actionFailedFromOneActionIsVisibleToAnother() = runBlocking {
    // Pins CURRENT behavior: one flag, six producers. If this passes, the notice crosses actions
    // and the defensive clear at the rename call site is load-bearing.
    ...
    assertTrue(store.actionFailed)   // after a leave failure
    // ...and nothing scopes it to the action that failed
}
```

**The pin is how you settle a question that reading cannot.** That is the reframe worth taking from this whole section: a characterization test is not only a safety net for a change you have already decided on -- it is the cheapest instrument for deciding.

Whatever it shows, it stays. If it shows cross-talk, it is the failing test the fix makes pass. If it does not, it is a pin on behavior that looked fragile and was not, which is worth keeping for the next person who suspects it.

## Step 5: make the change

Only now. And separately.

Change B is already done -- the seam *was* the change, because the goal was assertability. This is more common than it looks: a surprising share of "I want to change this" resolves to "I want to be able to see what this does."

Change A, if the pin shows cross-talk, is a data-structure change -- the [rarest and most valuable]({{ "/changing_code/m5-refactoring-catalogue/" | relative_url }}) category:

```kotlin
sealed interface ActionFailure {
    data object Create : ActionFailure
    data object Rename : ActionFailure
    data object Leave : ActionFailure
    data object Delete : ActionFailure
}
var lastFailure by mutableStateOf<ActionFailure?>(null)
    private set
```

Consumers filter on the case they care about, and the defensive clear at the rename call site deletes itself -- which is the signal that the change was the right one. **A fix that removes a workaround is a fix that addressed the cause.**

This commit is `fix`, not `refactor`, and it updates the pin in the same commit, because the pin's expected value is exactly what the change is changing. That is the [M5 rule]({{ "/changing_code/m5-discipline/" | relative_url }}) working as intended: the pin cannot be quietly updated during a structural commit, so the behavior change has to declare itself.

## The commit sequence

```
refactor(ward): parameterize the request clock -- no behavior change
test(ward): pin fileApplication's posted body and the shared failure flag
fix(ward): scope the action-failure notice to the action that failed
```

Three commits, three hats, in the only order that makes each one verifiable:

1. The **refactor** is provable because the change has not happened yet
2. The **pins** pass against unchanged code, which is what makes them a baseline
3. The **fix** changes behavior and its test in one commit, so the change declares itself

Reverting the third leaves the seam and the pins in place. Reverting all three is clean. Bisecting through them is decisive. None of that is true of the one commit that would have contained all three.

## What the exercise actually taught

**The audit's top finding was wrong, and the procedure caught it in step 1.** Not because the procedure is clever, but because it has a step whose only job is re-checking a claim before acting on it.

**The best finding came from reading the callers, not the class.** Every audit in this track pointed at the store. The evidence that something was actually wrong was a single line in a screen file -- a workaround somebody wrote and never mentioned. **Defensive code at a consumer is the highest-signal artifact in a codebase**, because it is a bug report written by someone who decided not to file one.

**The seam for one change was already there and unused.** Four modules of technique, and the answer to half the problem was "check what already exists." The ladder's real first rung is *look*.

**And the change that shipped was smaller than the audit proposed.** The audit suggested splitting a 48-function class. What shipped was one default parameter, two pins, and a sealed interface with four cases. The large restructuring is still available, still unjustified, and now -- with two pins in place -- meaningfully safer to attempt than it was.

> That is the actual output of this track: **not a cleaner file, but a file where the next change is cheaper than the last one.**

Next, and last: **the one-page version** of everything above -- the procedure without the worked example, short enough to run from memory.

## Exercise

Take your own audit -- the one from [M0]({{ "/changing_code/m0-measuring-change/" | relative_url }}), or a list of things you have been meaning to fix.

Pick the top item and run step 1 only: **verify the finding still holds.** Read the code and its callers, not just the code.

Then answer:

1. **Does the finding survive?** If not, what does the code actually do, and why did the audit read it wrong?
2. **Is there a defensive workaround at a caller?** A clear, a guard, a retry, a null check that should be impossible. That is where the real finding is.
3. **What is the smallest change that would settle the question?** If the answer is "a test," write that first -- you are not refactoring yet, you are measuring.
