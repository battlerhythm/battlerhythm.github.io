---
title: "Foundations"
order: 1
module: "M0"
module_title: "Foundations"
session: "1"
intent: "What to look at when you look at a pattern -- and how to tell when not to reach for one at all."
status: "complete"
excerpt_separator: "<!--more-->"
categories:
  - Design Patterns
tags:
  - Foundations
  - OCP
---

No patterns in this one. Instead: what to look at when you look at a pattern. Skip this and you can memorize all twenty-three and still misuse them.

<!--more-->

## Why the GoF book reads strangely today

*Design Patterns* was written in 1994, in C++ and Smalltalk. C++ had no lambdas at the time -- they arrived with C++11, seventeen years later -- no garbage collection, no reflection.

That matters more than it sounds, because **a large share of the twenty-three patterns are ways of faking a missing language feature with objects.**

Take Strategy. Its essence is "I want to pass a function as a value." With no first-class functions, you fake it: declare an interface with a single method and pass implementations of it around.

```kotlin
// The GoF form. In 1994 this was the only way.
interface DiscountPolicy {
    fun apply(amount: Money): Money
}

class Percentage(private val rate: Double) : DiscountPolicy {
    override fun apply(amount: Money) = amount * rate
}

// Kotlin. The pattern dissolved into syntax.
val discount: (Money) -> Money = { it * 0.9 }
```

Peter Norvig analyzed this in 1996 and found that **16 of the 23 patterns are invisible or qualitatively simpler in a dynamic language**.

So why learn them at all? Two reasons.

**The problems the patterns solve are all still here.** What the language gave you is syntactic convenience, not design judgment. Whether `(Money) -> Money` is the right seam is still your call.

**Even for the patterns a language absorbed, you need the original shape to know when to go back to it.** The lambda above is only comfortable while the strategy is stateless. The moment a discount policy has to count how often it was applied, or grow a second method like `describe()`, you need the interface back. Without knowing the GoF form, you cling to the lambda and produce something worse than either.

## The three principles

### 1. Program to an interface, not an implementation

"Interface" here does not mean the `interface` keyword. It means **what a type promises**.

```kotlin
// Bound to an implementation
class ReportService {
    private val storage = S3Storage()   // it ends here
    fun save(r: Report) = storage.put(r.id, r.bytes)
}
```

You cannot call S3 from a test, you cannot swap in something else for local development, and moving to GCS means editing this class.

```kotlin
// Bound to an interface
interface BlobStore {
    fun put(key: String, bytes: ByteArray)
}

class ReportService(private val store: BlobStore) {
    fun save(r: Report) = store.put(r.id, r.bytes)
}
```

**Here is where most people go wrong.** Extracting an interface is not the goal. An interface with exactly one implementation, forever, is pure cost -- one more file, one more jump, more to hold in your head, and nothing gained.

The refactoring above is justified by exactly one thing: **a second implementation actually exists** (the test double). Without that, don't.

### 2. Favor object composition over class inheritance

Inheritance has two problems. It is **fixed at compile time**, so nothing can be swapped at runtime; and it **breaks encapsulation**, because a subclass ends up depending on how its parent is built -- the fragile base class problem.

The classic exhibit is Java's `Stack extends Vector`. It is a stack, and yet `insertElementAt(0, x)` works on it. That is what you get when "is-a" is asserted where it isn't true.

Kotlin baked this lesson into the language: **classes are `final` by default.** You have to write `open` to allow inheritance. *Effective Java*'s "design and document for inheritance, or else prohibit it" became a language default. Worth noticing that you are applying principle 2 every time you don't write `open`.

### 3. Encapsulate what varies

The most practical of the three, and the **order** is the whole thing:

1. Find the code you **actually end up editing** when requirements change
2. Separate it from the rest
3. Put an interface in front of the separated part

Step 1 comes **first**. Predicting where change might land and encapsulating it in advance is the definition of over-engineering.

## How SOLID relates

The chronology helps: **GoF came first (1994), SOLID later (2000s).** SOLID reads as principles induced from the patterns rather than the other way round. Two of the five bear directly on patterns.

**OCP (open-closed).** Almost every GoF pattern is a concrete recipe for achieving it. *Twenty-three answers to the question of how to add behavior without editing existing code.* That single sentence runs through this whole series.

**DIP (dependency inversion).** Principle 1 at architecture scale.

And one thing that must be said plainly: **OCP is not free.** To achieve it you have to decide in advance *which axis will vary*, and **if you pick the wrong axis the extension point is just complexity that never pays off** -- you made the policy swappable and what actually changed was the storage.

That cost is exactly what the **Consequences** section of every GoF entry exists to record.

## How to read the catalog

Each GoF entry runs: Intent / Motivation / Applicability / Structure / Participants / **Consequences** / Implementation / Sample Code / Known Uses / Related Patterns.

**Read them in this order: Intent, Applicability, Consequences, Structure.**

Beginners read Structure and copy the UML. That is the route to patternitis. The judgment you need on the job is not "what does this look like" but **"should I use this here"** -- and that answer lives in Applicability and Consequences.

## Seeing coupling in real code

Turning the abstraction into signals you can act on:

| Signal | What it means |
|---|---|
| Count of concrete classes imported by one file | Direct measure of static coupling |
| Dependencies constructed inside the class | Nailed to that type |
| **You can't write a test** | The most honest indicator |
| Files touched by one requirement change | The real indicator -- and measurable |

That last one you can actually pull:

```bash
git log --format=format: --name-only --since="6 months ago" \
  | grep -v '^$' | sort | uniq -c | sort -rn | head -20
```

**Files that change together are coupled.** If that grouping disagrees with your module boundaries, the design is wrong. This beats static analysis because it reflects what actually happened, not what might.

## Three gates before adopting a pattern

All three must pass.

1. **Has the change actually arrived twice?** No predicting. Once is a coincidence; twice is a pattern.
2. **Is there exactly one clear axis of variation?** Two axes means consider Bridge. Unclear means it is too early.
3. **Does the code get shorter, or at minimum easier to follow, after the pattern goes in?**

Gate 3 deserves emphasis. If you introduced a pattern and now there are three more files and the flow is harder to trace, **that is a failure.** "It's the correct design" is not a justification.

## Exercise

Find three places in your own code that violate OCP. In order of smell:

1. **`when` / `if-else` chains that branch on type or kind** -- a new kind means editing here
2. **Boolean flag parameters** -- `fun render(x: X, compact: Boolean)` means two algorithms share one function
3. **Dependencies constructed inside a class** -- `= SomeConcreteThing()`
4. **One enum branched on in three or more scattered `when` blocks** -- adding a value means editing all of them

For each one, answer both questions:

- What change request would force me to edit this code?
- **Has that change actually arrived twice?** If not, leaving it alone is the right answer.

Ending up with cases where the answer to the second question is "no" is normal, and it is half the point of the exercise: separating *can be changed* from *should be changed*.

## Reference

- [Design Patterns in Dynamic Programming -- Peter Norvig, 1996](https://norvig.com/design-patterns/design-patterns.ppt)
- [Design Patterns: Elements of Reusable Object-Oriented Software](https://www.amazon.com/Design-Patterns-Elements-Reusable-Object-Oriented/dp/0201633612)
