---
title: "Breaking Dependencies"
order: 11
module: "M4"
module_title: "Legacy Code II: Breaking Dependencies in Kotlin"
session: "8-9"
intent: "Feathers' catalogue, reordered by how much of the frightening code each technique makes you touch -- and the ones Kotlin has made obsolete."
status: "complete"
excerpt_separator: "<!--more-->"
categories:
  - Code You Can Change
tags:
  - "Breaking Dependencies"
source: "Feathers"
---

Two dozen named techniques, and the book presents them alphabetically. Sorted by cost instead, they turn into a ladder with an obvious rule for climbing it.

<!--more-->

## The situation

You have done the work of the previous module. You know the class you need to change, you have listed its dependencies, and one of them has no enabling point. Now you need to give it one.

The constraint that shapes everything: **you must do this without a test**, because the test is what you are trying to make possible. So every technique here is judged on one axis before any other -- **how much of the frightening code does it make you touch?**

That is the ordering the book does not give you, and it is the only ordering that matters when you are working without a net.

## The ladder

| # | Technique | What you touch | Risk |
|---|---|---|---|
| 1 | **Sprout method / class** | One added line | Near zero |
| 2 | **Wrap method / class** | One rename, one new method | Low |
| 3 | **Parameterize method / constructor** | One signature, zero call sites | Low |
| 4 | **Extract interface** | A type, and every declaration site | Medium |
| 5 | **Break out method object** | The whole method | High |

Take the lowest-numbered one that solves the problem. Not the one that produces the nicest design -- the nicest design is what you do *after* the tests exist.

## 1. Sprout

You need to add behavior to a 400-line method you cannot test. Do not add it inline.

Write the new behavior as a **new function or class, fully tested on its own**, and add exactly one line to the untested method: the call.

```kotlin
// the 400-line method, untested. One line added:
val conflicts = RosterConflictFinder.find(cells, rules)   // new, tested in isolation
```

The trade is explicit and it is worth stating plainly, because this is where people balk: **the big method stays big.** You have not improved it. What you have done is guarantee that *the new logic* is tested, and that the risk of today's change is one line rather than forty.

Sprout a class rather than a method when the new behavior needs its own state, or when the existing class is so tangled that adding a private method means dealing with its constructor.

## 2. Wrap

Sprout works when the new behavior happens *inside* the flow. When it has to happen **around** it -- before, after, or conditionally instead -- wrap.

```kotlin
// before
fun publish(roster: Roster) { /* 80 untested lines */ }

// after
fun publish(roster: Roster) {          // same name, same signature: no call site changes
    publishRoster(roster)
    notifyMembers(roster)              // new, tested
}
private fun publishRoster(roster: Roster) { /* the original 80 lines, unmoved */ }
```

The original body is **renamed, not edited** -- and "unmoved" is the whole safety argument. An IDE rename is one of the few transformations you can trust without tests.

Kotlin adds a second form the book does not have. When the thing being wrapped is an *interface*, `by` gives you a decorator with no boilerplate:

```kotlin
class LoggingWardStore(private val inner: WardStore) : WardStore by inner {
    override suspend fun createWard(name: String): Ward =
        inner.createWard(name).also { log("created ${it.id}") }
}
```

Every other member is forwarded automatically. In Java this technique costs you a file full of delegating methods, which is the real reason it was used less than it should have been.

## 3. Parameterize

The most useful rung, and the one that Kotlin has changed the most.

The dependency is created or reached *inside* the method. Move that creation to a parameter, with the current behavior as the default:

```kotlin
// before -- hidden dependency
suspend fun run(file: PickedFile, wardId: String) {
    val pseudonymiser = Pseudonymiser()
    val today = CalendarMath.today()
    ...
}

// after -- two enabling points, zero call sites changed
suspend fun run(
    file: PickedFile,
    wardId: String,
    pseudonymiser: Pseudonymiser = Pseudonymiser(),
    today: LoomDate = CalendarMath.today(),
) { ... }
```

**Default arguments are the reason this rung is nearly free in Kotlin.** In Java, parameterizing a constructor means keeping the old constructor as an overload that delegates -- Feathers spends pages on the bookkeeping. Here the old behavior *is* the default value, and the compiler guarantees no call site changed.

Note the second parameter in that example. `CalendarMath.today()` is a call on a global `object` -- exactly the shape the book handles with *Introduce Instance Delegator*, a technique involving a new instance field and a static-to-instance migration. In Kotlin it is a default parameter. **The technique is obsolete; the problem it solved is not.**

## 4. Extract interface

The standard answer to "this class depends on a concrete class it cannot substitute." It is rung 4 and not rung 1 because it touches a type, which means touching every place that type is named.

The Kotlin caution is the one from [ISP]({{ "/changing_code/m1-isp/" | relative_url }}): mechanically extracting an interface from a class gives you a **header interface** -- every method, same names, one implementation. That is not a seam you wanted, it is the class with an extra file.

Here is what that looks like in practice. An import runner with this signature:

```kotlin
suspend fun run(
    file: PickedFile, wardId: String,
    wardSync: WardSyncStore,      // 1,047 lines, 48 public functions
    ...
)
```

uses exactly **two** of those forty-eight methods. Extracting `WardSyncStore` to an interface would produce a forty-eight-method interface, and testing `run` would still mean building a forty-eight-method fake.

The role-interface version is two methods. The **Kotlin** version is usually neither:

```kotlin
suspend fun run(
    file: PickedFile, wardId: String,
    layoutDetect: suspend (String, String, LoomDate) -> ImportModelResult<RosterLayoutSpec>,
    valueReading: suspend (String, String) -> ImportModelResult<Reading>,
    ...
)

// production call site -- method references, no adapter class
runner.run(file, wardId, wardSync::layoutDetect, wardSync::valueReading, ...)
```

No interface, no file, no fake class. The test passes two lambdas. **Reach for a function type first and an interface only when the collaborator has enough cohesive behavior to deserve a name.**

## 5. Break out method object

The last rung, for the case the others cannot reach: a single enormous method whose locals are effectively the state of a computation.

Move the whole method into a new class -- parameters become constructor parameters, locals become fields, the body becomes `invoke()`. Now the pieces can be split into private methods with real names and tested individually.

This is high risk because you are moving every line. Do it with an IDE's *extract* refactoring rather than by hand, and do it only when you have exhausted 1--4.

The honest signal that you have reached this rung looks like a hundred-line method that mixes fan-out with real logic:

```kotlin
suspend fun pullMemberRoster() = coroutineScope {
    // ... a dozen parallel GETs ...
    val merged = HashMap<LoomDate, String>()
    // Fold in publish order so a later publish wins the same date
    for (snap in archives.values.flatten().sortedBy { it.publishedAt })
        for (c in snap.cells) if (c.nurseId == myId) c.code?.let { merged[c.date] = it }
    // ... twenty more assignments ...
}
```

That fold is a **pure function with real semantics** -- "a later publish wins the same date" is a rule someone decided, and it is currently untestable because it lives inside a method that first makes twelve network calls.

But notice: you do not need rung 5 to fix *that*. Extracting one pure function out of the middle of a method -- `fun mergeByPublishOrder(archives, myId): Map<LoomDate, String>` -- is rung 1 in reverse, and it is the highest-value move available in a method like this. **Look for the pure core before you move the whole thing.**

## The counterargument

The techniques leave the code visibly worse in the short term, and the book does not dwell on the bill.

**Sprouts that are never harvested.** Sprout five times into the same 400-line method and you have a 405-line method plus five small classes. The method is no better; it now also has five outbound dependencies. Sprouting is a **loan against a later extraction**, and loans that are never repaid are how a codebase ends up with both a god class and a scatter of tiny helpers.

The mitigation is a rule, not a resolution: **when the sprouts around one region reach three, the next change to that region extracts it.** Three is arbitrary; having a number is not.

**Wrapping accumulates names nobody chose.** `publishRoster` exists because `publish` was taken. Two or three of those in a file and the reader cannot tell which is the entry point.

**Mechanical translation produces Java-in-Kotlin.** *Expose Static Method*, *Introduce Instance Delegator*, and much of the constructor-parameterization bookkeeping exist because of what Java could not express. Applying them literally in Kotlin gives you ceremony where a default parameter or a method reference would do. The techniques are not sacred -- **the diagnosis is.** "Where is the enabling point, and what is the cheapest way to add one" is the durable part; the catalogue is one language's answers.

**And the whole ladder assumes you can rename and extract safely.** That assumption rests on the IDE, not on discipline. On a codebase where the refactoring tools are unreliable -- unusual generated code, heavy reflection, a language server that gives up -- rungs 2 and 5 are considerably more dangerous than this post makes them sound.

## In the wild

- **Ktor's** `HttpClient(engine)` is rung 3 as a public API: the engine is a constructor parameter with a platform default, which is why `MockEngine` needs no other support.
- **Retrofit's** `CallAdapter.Factory` and OkHttp's `Interceptor` are rung 2 as a product feature -- the library ships the wrap point so you do not have to create one.
- **kotlinx.serialization's** `Json { }` builder is rung 3 for configuration: every call site that does not care gets `Json.Default`.

The pattern: libraries that are pleasant to test are libraries whose authors put rungs 2 and 3 in the public API on purpose.

## When to stop

**Stop when the seam you needed exists.** Not when the class is well-designed. The design work is a separate activity, done later, with tests.

**Do not climb the ladder for a dependency you have not confirmed blocks you.** Write the test first, let it fail to compile or fail non-deterministically, and let the failure name the dependency. Guessing produces seams nobody uses.

**Do not break a dependency in the same commit as a behavior change.** This is the M5 rule arriving early, and it matters most here: a commit that both parameterizes a constructor and changes what the default does is unreviewable and unrevertible.

**If a technique requires making a class `open`, stop and read the next post.** Feathers' catalogue leans on subclass-and-override far more than this ladder does, because in Java everything is virtual by default. In Kotlin that rung has a price, and whether it is ever worth paying is the next session.

## Exercise

Find a file in your project with **zero test references** and more than two hundred lines. Open its largest method.

Do not refactor it. Answer three questions:

1. **What does it construct or call globally inside the body?** Each one is a rung-3 candidate -- write out the parameter list it would have with defaults.
2. **Which parameter is a fat dependency?** A collaborator whose methods you use two of, out of many. Write the function types that would replace it.
3. **Is there a pure function buried in the middle?** A fold, a classification, a derivation that takes values and returns a value. That one is worth extracting before anything else, because it is the part with rules in it.

If all three answers are "nothing," the method is probably fine and the file is long for another reason. That is a useful result too.
