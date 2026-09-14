---
title: "Deep Modules"
order: 6
module: "M2"
module_title: "Names and Boundaries"
session: "4-5"
intent: "Functionality divided by interface is the measure. A shallow module adds a layer without removing anything."
status: "complete"
excerpt_separator: "<!--more-->"
categories:
  - Code You Can Change
tags:
  - "Deep Modules"
source: "Ousterhout"
---

The idea that gives "abstraction" a unit, and the one that most directly contradicts the advice in the previous module.

<!--more-->

## The problem

You extract a layer. It compiles, the tests pass, the diagram looks better. And the caller is no easier to write than before, because the layer exposes exactly what it wraps.

```kotlin
class ScheduleService(private val store: WardStore) {
    suspend fun ward(id: WardId) = store.ward(id)
    suspend fun members(id: WardId) = store.members(id)
    suspend fun addMember(id: WardId, m: Member) = store.addMember(id, m)
    // ...30 more of these
}
```

Every method is a pass-through. The caller must still know every operation the store offers, in the same shape, plus the existence of one more type. **Nothing was hidden, so nothing was gained -- and a file was added.**

This is not a strawman. It is what happens when a layer is created to satisfy a diagram, or extracted mechanically to get a test seam.

## The claim

Ousterhout gives abstraction a unit:

> **The best modules are deep: a simple interface hiding a lot of functionality.**

Think of a module as a rectangle. The **area** is the functionality it provides; the **top edge** is the interface. Depth is area over edge, and the goal is to maximise it.

The canonical deep module is Unix file I/O. Five calls -- `open`, `read`, `write`, `close`, `lseek` -- behind disk scheduling, a buffer cache, permissions, block allocation, and half a dozen filesystem implementations. Decades of change happened under that interface without callers noticing.

His counterexample is in Java's standard library. To read a file with buffering you need to know that `FileInputStream` does not buffer, that `BufferedInputStream` exists, and that you must compose them:

```java
new BufferedInputStream(new FileInputStream(path))
```

Three classes' worth of knowledge for the thing everyone wants. **The interface exposes a decision that should have been a default.**

## The objection

This contradicts the module you just finished, and the contradiction is the point.

**"Deep module" can be a licence for a God object.** Ousterhout is openly skeptical of small-class advice, and the argument reads as permission to keep growing a class. It is not, and the distinction is sharp: **depth is measured by the interface, not by the size.**

A 1,000-line class with 48 public methods is not deep. It is large and its interface is large, which is the worst quadrant. A 1,000-line class with four public methods might be excellent. **Line count says nothing; public surface says everything.**

That resolves the apparent conflict with SRP too. [SRP]({{ "/changing_code/m1-srp/" | relative_url }}) is about *what is inside* -- whether one module serves two actors. Depth is about *what is exposed*. A module can be deep and still serve two actors, in which case it should be split; and splitting it correctly produces two deep modules, not two shallow ones.

**Some shallow layers are correct.** A port in a hexagonal architecture, or an adapter at a module boundary, may be entirely pass-through -- and its value is not hiding functionality but **controlling the direction of the dependency**. That is [DIP]({{ "/design_patterns/m0-fundamentals/" | relative_url }}), and it buys something depth analysis cannot see. Do not delete a layer without asking what it buys; sometimes the answer is an arrow.

## The technique

**Compute the ratio.** Crude and effective:

```bash
# public members vs internal lines, per file
for f in $(find src -name '*.kt'); do
  pub=$(grep -cE '^\s{0,4}(public )?(suspend )?(fun|val|var) ' "$f")
  lines=$(wc -l < "$f")
  [ "$pub" -gt 0 ] && printf "%6.1f  %3s pub  %5s lines  %s\n" \
    "$(echo "$lines/$pub" | bc -l)" "$pub" "$lines" "$f"
done | sort -n | head -20
```

Sorted ascending, the top of that list is your shallow modules: many public members, few lines each. A file with 30 public members and 200 lines is about seven lines of functionality per exposed concept.

**Count pass-throughs.** A method whose body is a single expression forwarding the same arguments to the same-named method one layer down is a pass-through. A handful is fine. A layer that is mostly pass-throughs is not a layer.

**Ask the deletion question.** *If I removed this layer, what would the caller have to learn?* If the answer is "nothing", the layer is pure cost. If it is "the retry policy, the cache key format, and which of three endpoints to call", the layer is doing its job.

That question is more reliable than any ratio, because it measures what the module actually hides rather than how much code it contains.

## In Kotlin

**`internal` is the depth tool.** The single most effective way to deepen a module is not to move code -- it is to stop exporting it. Every declaration you mark `internal` shrinks the interface without touching the functionality, which raises the ratio directly.

This is worth doing deliberately at module boundaries: make everything `internal` by default, then promote only what a caller needs. Kotlin's **explicit API mode** (`explicitApi()` in the Gradle build) enforces the discipline by requiring an explicit visibility and return type on every public declaration -- which turns "is this really part of the interface?" into a question you must answer.

**Default arguments keep interfaces narrow.** The Java overload family -- five `read` methods -- is five interface entries. One function with defaults is one entry that covers the same ground, and the caller who wants the common case writes nothing.

**Extension functions add functionality without widening the type.** `fun Nurse.displayName()` is available at the call site and is not part of `Nurse`. The type's interface stays minimal while the vocabulary grows -- and the extension lives in the module that cares.

**A sealed return type can replace several methods.** `fun import(f: File): ImportResult` where `ImportResult` is sealed exposes one entry point instead of `tryImport` / `importOrNull` / `importWithDiagnostics`.

## In the wild

- **`Flow`** -- the interface is essentially `collect`. Everything else is extension functions, so the type stays one method deep while the library offers dozens of operators. A near-perfect illustration of the Kotlin technique above.
- **Room** -- a DAO interface of the queries you use, hiding SQLite, cursors, threading and migrations
- **`java.io` streams** -- Ousterhout's own counterexample, still in the standard library
- **`Collections.unmodifiableList`** -- shallow *and* [not substitutable]({{ "/changing_code/m1-lsp/" | relative_url }}), which is how one design decision can fail two ways

## When to stop

**Do not delete a layer before asking what it buys.** Direction, testability, and a module boundary are all legitimate purchases that depth analysis does not measure.

**Do not merge modules to raise a ratio.** The metric is a diagnostic; combining two coherent modules to improve a number is Goodhart arriving on schedule.

**Shrink the interface before moving the code.** `internal` is reversible, costs one keyword, and frequently reveals that the module was deep enough and only over-exported.

## Exercise

Run the ratio script and take the shallowest file with more than ten public members.

For each public member, ask the deletion question: *what would a caller have to learn if this were not here?* **Mark the ones where the answer is "nothing" -- those are the interface, not the functionality**, and the count of them is how much of that file is a layer rather than a module.
