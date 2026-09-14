---
title: "Liskov Substitution"
order: 3
module: "M1"
module_title: "SOLID, Minus the Two You Know"
session: "2-3"
intent: "The only SOLID principle with a formal definition — and the one Kotlin’s variance annotations already encode."
status: "complete"
excerpt_separator: "<!--more-->"
categories:
  - Code You Can Change
tags:
  - "Liskov Substitution"
source: "Liskov"
---

The one principle in SOLID that came from a researcher rather than a consultant, and the only one you can state precisely enough to be wrong about.

<!--more-->

## Skip the squares

Every explanation starts with `Square extends Rectangle`: set the width, the height changes, a caller that expected independent dimensions breaks.

**It is a bad example and it teaches the wrong lesson.** Nobody models geometry that way, and the example makes it look as though the problem is *mutability*, so the takeaway becomes "use immutable types" -- which does not prevent the violation at all.

Here is a real one, from the JDK:

```kotlin
val list: List<String> = Collections.unmodifiableList(mutableListOf("a"))
list.add("b")      // UnsupportedOperationException
```

`unmodifiableList` returns something typed as a `List` that does not behave as a `List`. Java's answer was to declare `add` an *optional operation*, which makes the contract loose enough that this is formally legal -- and leaves every caller unable to rely on it.

**`UnsupportedOperationException` is a language-level announcement that a type is not substitutable for the type it claims to be.** Whenever you see one, you are looking at this principle being paid for.

## The claim

Barbara Liskov, 1987, and it is worth reading once in the original form:

> Let φ(x) be a property provable about objects x of type T. Then φ(y) should be true for objects y of type S where S is a subtype of T.

Anything you could prove about the parent must stay true of the child. The working version is four contract rules:

| Rule | Meaning | Violation looks like |
|---|---|---|
| **Preconditions may not be strengthened** | The subtype cannot demand more | an added `require(...)` in the override |
| **Postconditions may not be weakened** | The subtype must promise at least as much | returning null where the parent never did |
| **Invariants must be preserved** | Whatever was always true stays true | a subtype that lets a field go negative |
| **History constraint** | The subtype cannot allow state changes the parent forbade | adding a setter to something immutable |

And underneath those, **variance**: a substitutable override may accept *wider* parameter types (contravariant) and return *narrower* ones (covariant). Java and Kotlin allow the return half and not the parameter half -- widening a parameter produces an overload, not an override, which is its own source of bugs.

## The objection

**It is not a rule about inheritance.** It is a rule about subtyping, and implementing an interface is subtyping. "We don't use inheritance" does not exempt a codebase: an implementation that throws on a method its interface declares is exactly the `unmodifiableList` problem with a different keyword.

**Without a written contract there is nothing to violate.** The rules are about preconditions and postconditions, and most codebases have never written any down. In practice this means LSP is not used to *find* problems -- it gets invoked afterwards to explain a bug that already happened. That is the honest reason the principle is respected and rarely applied, and it points at the real prerequisite: **you cannot check substitutability until the contract exists**, which is what [characterization tests]({{ "/changing_code/m3-characterization-tests/" | relative_url }}) are for.

**Sometimes the violation is the right call.** `unmodifiableList` is genuinely useful. The JDK chose a loose contract over a larger type hierarchy, and living with `UnsupportedOperationException` was the price. Knowing the principle does not mean never breaking it; it means **breaking it deliberately and writing down what callers lose.**

## The technique

**Inherit the test suite.** This is the practical test for substitutability and it is nearly mechanical: write the tests against the supertype's contract, then run them against every implementation.

```kotlin
abstract class NurseRepositoryContract {
    abstract fun create(): NurseRepository

    @Test fun `saved nurse is retrievable by id`() { /* ... */ }
    @Test fun `byWard returns only that ward`() { /* ... */ }
    @Test fun `saving twice with the same id replaces`() { /* ... */ }
}

class PostgresNurseRepositoryTest : NurseRepositoryContract() {
    override fun create() = PostgresNurseRepository(testDataSource)
}

class InMemoryNurseRepositoryTest : NurseRepositoryContract() {
    override fun create() = InMemoryNurseRepository()
}
```

The suite is the contract, written down and executable. A subtype that cannot pass it is not substitutable, and you find out at build time rather than in an incident.

This has a second payoff that is bigger than the first: **the in-memory fake you use in every other test is now verified against the real implementation.** Fakes drifting from the thing they fake is one of the quieter sources of green tests and broken production.

**Three greps that find violations without any contract at all:**

```bash
grep -rn "UnsupportedOperationException"          # announced non-substitutability
grep -rn -A3 "override fun" | grep -B1 "require(" # strengthened precondition
```

And read every `override` whose return type gained a `?`.

## In Kotlin

This is the principle the language enforces most, which is worth knowing because it means part of the work is already done.

**Variance is in the type system.** `List<out E>` is covariant because it is read-only -- a `List<String>` is safely a `List<Any>` when nothing can be put into it. `MutableList<E>` is invariant for the same reason in reverse. Compare Java's arrays, which are covariant and unsound: `Object[] a = new String[1]; a[0] = 1;` compiles and throws `ArrayStoreException` at runtime. **Java's array covariance is a type system that violates this principle**, and Kotlin's `out`/`in` is the fix.

**Nullability makes postcondition weakening a compile error.** Override `fun find(): Nurse` with `fun find(): Nurse?` and the compiler refuses. In Java the same change compiles and the breakage surfaces at a call site somewhere else.

**`final` by default removes most of the opportunity.** You cannot accidentally violate a contract of a class that cannot be subclassed. This is the same language decision that will make [Feathers' techniques awkward]({{ "/changing_code/m4-kotlin-final/" | relative_url }}) in a later module -- the safety and the inconvenience are the same feature.

What Kotlin does **not** check: preconditions, invariants, and the history constraint. Those still need the inherited test suite.

## In the wild

- **`List` vs `MutableList`** -- Kotlin split the type rather than making mutation an optional operation. That is the design `unmodifiableList` could not have.
- **`Collections.unmodifiableList`** -- the counterexample, still in daily use
- **Java arrays** -- covariance without soundness, preserved for compatibility since 1.0
- **`Iterator.remove`** -- another optional operation, and another reason `ConcurrentModificationException` exists

## When to stop

**Write the contract before judging the violation.** An inherited test suite is worth building; an argument about whether something "violates LSP" without one is not.

**One implementation means no substitution and no problem.** The principle applies where there are two or more.

**A deliberate violation needs documentation, not a refactor.** If `UnsupportedOperationException` is genuinely the right design, say so at the declaration site and move on.

## Exercise

Find an interface in your code with two or more implementations -- a repository with a real and an in-memory version is the usual one.

Write the contract test suite and run it against both. **The question is not whether the fake passes; it is what you had to discover about the real implementation in order to write the assertions.** Each of those discoveries was an undocumented contract, which is the leading indicator of unknown unknowns from [the previous session]({{ "/changing_code/m0-measuring-change/" | relative_url }}).
