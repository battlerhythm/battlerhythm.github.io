---
title: "Single Responsibility"
order: 2
module: "M1"
module_title: "SOLID, Minus the Two You Know"
session: "2-3"
intent: "“One reason to change” is the most misread sentence in software. The reason is a person, not a topic."
status: "complete"
excerpt_separator: "<!--more-->"
categories:
  - Code You Can Change
tags:
  - "Single Responsibility"
source: "Martin"
---

The most quoted and least usable of the five -- until you replace "responsibility" with something you can look up.

<!--more-->

## The problem

*A class should have one responsibility.* Now apply it.

Is `NurseRepository` one responsibility? It reads and writes nurses -- sounds like one. Is `save()` one responsibility? It validates, serializes, writes and invalidates a cache -- sounds like four. Is the whole `server` module one responsibility? It serves the API.

Every granularity can be argued to be "one thing", which means the principle decides nothing. **A principle that never rules anything out is not a principle; it is a mood.**

## The claim

Martin has restated it, and the restatement is the usable version. From *Clean Architecture*:

> A module should be responsible to one, and only one, **actor**.

The shift is from *reason* to *person*. "Reason to change" is a judgement call; **"who asks for this change" is a fact you can look up in an org chart.**

His example: an `Employee` class with three methods.

| Method | Who requests changes to it |
|---|---|
| `calculatePay()` | Finance |
| `reportHours()` | Operations |
| `save()` | The DBAs |

Three actors. Put them in one class and a change Finance asked for can break the report Operations depends on -- by way of a shared private helper that seemed harmless to adjust.

**That is the real argument, and it is not about technical coupling.** It is that code serving two actors makes those actors collide, and neither of them knows the other exists.

## The objection

Three, and the third is the one that should change how you apply it.

**A solo developer has one actor.** If you are the founder, the engineer and the support desk, the org chart is a single box and SRP says nothing. The fix is to read *actor* as **source of change**, not as employee: a regulator, a payment provider, a platform review team, a hospital customer. For a scheduling product, labour law and the customer's local policy are two actors even though one person implements both -- because **they change on independent schedules for independent reasons**, which is exactly the property that matters.

**Cohesion is the older and sharper idea.** Constantine and Yourdon graded cohesion on a seven-level scale -- coincidental, logical, temporal, procedural, communicational, sequential, functional -- in the 1970s. SRP is close to a repackaging of the top level. The scale is more useful in review because it gives you names for the *bad* kinds, and "these methods are grouped because they all run at startup" (temporal cohesion) is a diagnosis where "this violates SRP" is only a verdict.

**Taken literally it produces classitis.** Ousterhout's term, and his position is the direct opposite of Martin's: many small classes each make sense alone while the system gets harder, because **every class is an interface you must learn**. Splitting a class is not free -- it converts a private call into a public contract, and public contracts are the things that cannot be changed quietly.

Hold both. **SRP tells you where a split is legitimate. It does not tell you that the split is worth it.** The second question is the [gates]({{ "/design_patterns/m0-fundamentals/" | relative_url }}) again: has the change actually arrived twice, and is the result easier to follow?

## The technique

Three ways to decide, in increasing order of cost and confidence.

**1. List the actors.** For a class you suspect, write down who can ask for each method to change. If the list has one entry, stop -- you are done, and the class is fine. Most classes pass this.

**2. Read the commit messages as evidence.** Actors leave traces. Take a file's history and classify each commit by motive: a bug fix, a compliance change, a design tweak, a performance fix. A file whose history alternates between unrelated motives is serving unrelated actors, and the history says so more honestly than your intuition does.

```bash
git log --oneline --follow -- path/to/File.kt | head -40
```

**3. Check which methods use which fields.** If the methods of a class partition into groups that touch disjoint sets of properties, the class is already two classes that share a file. This is the intuition behind the LCOM metric, and you do not need the metric -- reading the field list next to the method list is enough.

## In Kotlin

Java made this a *class* question because a class was the only container. Kotlin has three more, and two of them are usually the right answer.

**Extension functions move an operation to the actor that owns it.**

```kotlin
// domain module: changes when the hospital's rules change
data class Nurse(val id: NurseId, val name: String, val seniority: Int)

// ui module: changes when the designer changes their mind
fun Nurse.displayName(): String = "$name ${seniority}년차"
```

`Nurse` never acquires a display concern. In Java that string formatting would have to live in `Nurse`, in a static utility, or in a wrapper -- and the first option is what usually happened.

**Top-level functions in a file mean cohesion does not require a class.** A file is a container. `ShiftMath.kt` holding six related pure functions is cohesive without inventing a type, and it avoids the classitis tax.

**`internal` distinguishes a split from an exposure.** Splitting a class into two `internal` classes inside one module costs you nothing publicly. Splitting it into two public ones creates a contract. **When a split is right but the new boundary is not yet proven, make it `internal`** -- you get the separation without committing to the interface.

Worth noticing that `data class` plus extension functions is the idiomatic shape this produces: the data has one home, and each actor's operations live in that actor's module.

## In the wild

- **`kotlinx.serialization`** moves serialization out of the type entirely. `@Serializable` generates a separate serializer, so a type does not gain a persistence concern by being persisted. Compare with the Java tradition of `implements Serializable` and `writeObject`.
- **Room's Entity / DAO split** is the same division: the row shape and the queries change for different reasons and different people.
- **Compose's `Modifier`** keeps layout concerns out of composables -- a component does not grow a padding parameter per caller.

## When to stop

**One actor means do not split.** The most common cost of learning SRP is a codebase of one-method classes that is harder to read than what it replaced.

**Prefer `internal` over public when splitting.** A split you can reverse is cheap; a published interface is not.

**A split that only moves code is not a split.** If the two halves must still change together, you have made two files out of one problem, and your co-change analysis will say so next month.

## Exercise

Take the largest class you own. Write each public method on a line, and next to it the actor who could ask for it to change.

Then count distinct actors. **If it is one, you are done and the class is fine at whatever size it is** -- size was never the criterion. If it is three, you have found the split, and the next question is whether the change has arrived twice.
