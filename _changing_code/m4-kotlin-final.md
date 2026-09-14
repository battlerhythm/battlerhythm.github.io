---
title: "When final Blocks You"
order: 12
module: "M4"
module_title: "Legacy Code II: Breaking Dependencies in Kotlin"
session: "8-9"
intent: "A third of Feathers' catalogue assumes subclass-and-override. Kotlin closed that door on purpose -- and the question is whether the replacement is as good."
status: "complete"
excerpt_separator: "<!--more-->"
categories:
  - Code You Can Change
tags:
  - "When final Blocks You"
source: "Kotlin"
---

The one place where the standard legacy-code technique set and the language's design decisions are in open conflict. Neither side is obviously wrong, which makes it the most interesting session in this track.

<!--more-->

## The block

Open *Working Effectively with Legacy Code* to the technique chapters and count. A substantial share of the catalogue is one move in different clothes:

- **Subclass and Override Method** -- make a testing subclass, override the inconvenient method
- **Extract and Override Call** -- pull a problem call into its own method, override that method in the subclass
- **Extract and Override Factory Method** -- pull a `new` out into a factory method, override it to return a fake
- **Extract and Override Getter** -- same, for a field the class initialises itself

They are the book's workhorses because in Java they are nearly free. Every method is virtual unless someone typed `final`, and almost nobody did. Feathers could assume the door was open, because in Java it always was.

In Kotlin, all four are blocked on arrival. Classes are `final` unless declared `open`, and so are members of an `open` class. The technique requires you to first edit the class you are afraid to edit -- which is exactly the thing the technique existed to avoid.

## Why Kotlin did it

Not an accident, and not a Kotlin invention. The language team cite Bloch directly: *Effective Java*'s

> Design and document for inheritance or else prohibit it.

(Item 17 in the second edition, Item 19 in the third.) Kotlin's move was to make "prohibit it" the default rather than an act of discipline nobody performs.

The underlying problem is the **fragile base class**: a subclass that overrides a method becomes coupled to the base class's *internal call sequence*, not just its public contract. The base class author then changes which internal method calls which -- a change they correctly believe is invisible -- and the subclass breaks. Nothing in Java's type system records that the coupling existed.

There is a second reason that matters more for this track. **An override-based seam is the most implementation-coupled seam available.** It pins not just what the class does but how it does it internally, which is precisely what the [characterization-test]({{ "/changing_code/m3-characterization-tests/" | relative_url }}) brittleness objection warns about. Feathers himself flags the risk. Kotlin makes the cheapest-but-worst seam expensive, and the effect is to push you up the [ladder from the previous post]({{ "/changing_code/m4-breaking-dependencies/" | relative_url }}).

## The counterargument, which is strong

This decision has produced a decade of documented friction, and the honest version of this post has to state it.

**Frameworks needed a compiler plugin.** Spring's proxying, Hibernate's lazy loading, and Mockito's original subclass mock maker all require non-final classes. The answer was `kotlin-allopen` and its `kotlin-spring` preset: a compiler plugin whose entire job is to undo the language default for annotated classes. When a language default requires a first-party plugin to reverse it for the most popular framework on the platform, that is evidence about the default, not just about the framework.

**The tooling had to be rebuilt.** Mocking a final class with Mockito used to require the separate `mockito-inline` artifact; since Mockito 5 the inline mock maker is the default and final classes, static methods and constructors mock out of the box. That is a real fix -- and it arrived by **bypassing** the language's intent with bytecode instrumentation, not by agreeing with it.

**The empirical claim is contested.** The fragile base class problem is real in theory; how often it actually bites, versus how often final-by-default blocks a legitimate extension, is not something either side has measured. "Rarely happens in practice" is a common objection and it has never been refuted with data.

**And the defaults are arguably incoherent.** Kotlin makes members `public` by default -- permissive -- and `final` by default -- restrictive. If the argument is that authors should have to opt into commitments they have not thought about, visibility is the larger commitment of the two.

## The distinction that resolves it

The two positions are not actually arguing about the same object, and noticing that is the point of the session.

**Kotlin's argument is about code you own and are designing.** For that code it is straightforwardly right: an `open` class is a public contract with subclasses you cannot see, and you should not enter one by accident.

**Feathers' problem is code nobody designed for this.** The author never anticipated a test. There is no contract to preserve because none was ever stated. Java's virtual-by-default is bad *design*, and it is an excellent *rescue* property.

Which means the practical shape is an asymmetry:

| | Can you type `open`? | Situation |
|---|---|---|
| **Your own module** | Yes | One word. Cheaper than Feathers' whole technique -- so "can I" is not the question |
| **A dependency you don't control** | No | Genuinely stuck. No technique helps, and this is where the default costs you |

For the first row, the question is never *can I* but *should I* -- and the answer is a rule, below. For the second row, Kotlin's default is a real and unrecoverable cost, paid to library consumers by a decision the library author made implicitly. That cost is the strongest form of the counterargument, and it has no rebuttal other than "wrap it" -- which works, at the price of a file.

## The replacement table

Every override-based technique has a Kotlin equivalent that is cheaper, and the substitutions are mechanical.

| Feathers | Kotlin replacement | Rung |
|---|---|---|
| Subclass and Override Method | Interface + `by` delegation, or a function-type parameter | 2--3 |
| Extract and Override Call | Parameterize with a function type, current call as the default | 3 |
| Extract and Override Factory Method | `create: () -> Thing = { Thing() }` as a parameter | 3 |
| Extract and Override Getter | The value itself as a parameter with a default | 3 |
| Introduce Instance Delegator (for statics) | Function-type parameter, or an extension function | 3 |
| Expose Static Method | Top-level function in the same file | 1 |

The pattern is worth naming: **the Kotlin answer to "override this method" is almost always "pass that method in."**

```kotlin
// Feathers: extract the call, subclass, override
class Importer {
    fun run(file: File) { ...; val today = CalendarMath.today(); ... }
}
class TestingImporter : Importer() { override fun today() = LoomDate(2026, 1, 1) }   // needs open, open, open

// Kotlin: pass it in
class Importer(private val today: () -> LoomDate = { CalendarMath.today() })
// test: Importer(today = { LoomDate(2026, 1, 1) })
```

The second version is shorter, requires no subclass, leaves no inheritance contract behind, and cannot couple the test to the class's internal call order -- because there is no internal call to couple to.

## The rule for `open`, when you do own the class

**Never make a class `open` for a test.** If the reason is testing, the correct move is one rung lower: a function-type parameter, or an interface with the production class as one implementation.

**If you make a class `open`, you have accepted the LSP obligation** for every override that will ever exist -- preconditions no stronger, postconditions no weaker, invariants preserved, and now with subclasses you cannot see. That is the [contract from M1]({{ "/changing_code/m1-lsp/" | relative_url }}), and `open` is the keyword that signs it.

**`internal open` is the underused middle ground.** Open within the module, closed to every consumer. Test source sets are associated with their module's compilation, so tests can subclass it while the published API stays sealed. If you are convinced you need inheritance for an internal reason, this is the version that does not export the commitment.

**Inheritance in the test source set is fine.** The one thing this post is not arguing against: an `abstract class` that holds shared fixture setup, or the contract-test base class from the LSP post, both live in test code and bind nothing in production. Different decision entirely.

## The natural experiment

Arguments about language defaults usually run on anecdote. Here is a codebase to look at instead -- Kotlin Multiplatform, roughly 50,000 lines, **941 tests**:

| | Count |
|---|---|
| `class` declarations | 173 |
| `open class` | **1** |
| `open` members | **0** |
| `abstract class` | 1 -- and it is `PgTest`, a test fixture base |
| Mocking frameworks | **0** (`kotlin-test` only) |
| Hand-written test doubles | 14 |

Two things stand out.

**The one `open class` is open for a production reason, not a test.** It has exactly one real subclass. And its own dependencies are six function-type constructor parameters -- `read`, `write`, `canWrite`, `locked`, `next`, `prev`. Even the codebase's single use of inheritance does not use inheritance as a seam.

**941 tests, no mocking library.** The doubles are hand-written classes, and their names have already sorted themselves into a taxonomy: `Fake*` (a working implementation), `InMemory*` (a persistence substitute), `Recording*` (captures calls for assertion). That is Meszaros' fake / stub / spy distinction, arrived at by necessity rather than by reading about it.

This is not proof that final-by-default is correct. It is a demonstration that a substantial test suite can be built with **zero** inheritance-based seams and no dynamic mocking -- which is the specific claim the counterargument doubts.

## The multiplatform forcing function

There is a constraint here that makes the choice for you, and it deserves to be named because it is not obvious.

**Dynamic mocking requires runtime bytecode manipulation.** That is a JVM capability. Kotlin/Native and Kotlin/Wasm have no equivalent, so in `commonTest` the mocking libraries are simply not available -- and Mockito's own inline mock maker, the fix for final classes, does not work under GraalVM native images either, which is why `mockito-subclass` still exists.

So a multiplatform module cannot take the shortcut even if it wants to. Every double must be a hand-written class implementing an interface, which means every seam must be an **interface or a function type**, which is rung 2--3 of the ladder.

A constraint that removes the worst option is worth more than advice against it.

## When to stop

**Do not fight `final`.** If you find yourself reaching for `allopen`, a mocking library that instruments bytecode, or reflection to reach a private field, stop -- every one of those is a signal that you skipped a rung. Go back and ask where the enabling point should be.

**Do not `open` a class in a library you publish** for a reason that lives in your test suite. You are exporting a permanent contract to buy a temporary convenience.

**Do accept that some third-party code is genuinely unreachable.** When a dependency's final class is in your way, the answer is a wrapper -- an interface you own, one adapter implementation, and your code depending on your interface. It costs a file. That is the actual price of the language default, and it is worth paying honestly rather than instrumenting around.

**Stop when the seam exists.** Same as every post in this module.

## Module M4: breaking dependencies

- **[The ladder]({{ "/changing_code/m4-breaking-dependencies/" | relative_url }})** -- sprout, wrap, parameterize, extract interface, break out method object, ordered by how much of the frightening code each one makes you touch
- **`final`** -- the rung the book leans on hardest is the one Kotlin removed, and the replacements are lower on the ladder rather than higher

The two combine into a single instruction:

> **Pass the dependency in. If you cannot, wrap it. If you cannot do that either, you are looking at code you do not own, and the answer is an interface you do own.**

Which is a shorter book than Feathers wrote, because a language that expresses functions as values does not need most of the ceremony that a language without them required.

Next is **M5: refactoring** -- what to do once the tests exist, and the one discipline rule that has appeared in every module of this track so far.

## Exercise

Search your codebase for `open ` and count the results. Then, for each one, answer:

1. **Is there an actual subclass in production?** If not, it is a promise nobody collected on -- delete `open` and see if anything breaks.
2. **If the only subclass is in a test source set**, replace it: a function-type parameter or an interface will do the same job without the inheritance contract.
3. **If there is a real production subclass**, check the LSP obligation. What does the base class's contract say, and where is it written down? If the answer is "nowhere," you have an `open` class with an unwritten contract, which is the fragile base class waiting to happen.

Then search for the opposite: a place where you wanted to test something and could not, and blamed `final`. Look at it again with the replacement table. The odds are that the dependency wanted to be a parameter.
