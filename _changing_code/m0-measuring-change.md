---
title: "Measuring Changeability"
order: 1
module: "M0"
module_title: "Measuring the Thing"
session: "1"
intent: "Replace “clean” with three things you can actually measure: change amplification, cognitive load, and unknown unknowns."
status: "complete"
excerpt_separator: "<!--more-->"
categories:
  - Code You Can Change
tags:
  - "Measuring Changeability"
source: "Ousterhout"
---

No techniques in this one. First we need something to aim at, because the usual target does not survive contact with a disagreement.

<!--more-->

## The problem with "clean"

Two engineers look at the same function. One says it should be split into four. The other says splitting it would scatter one idea across four places. Both cite *Clean Code*. Neither can be shown wrong, because the claim on the table is **aesthetic** -- and aesthetic claims do not resolve, they alternate with whoever reviewed last.

That is not a failure of taste. It is a failure of target. "Clean" is unfalsifiable, so a team that aims at it argues forever and a solo developer just drifts.

The fix is to aim at something with a truth value. **Not clean -- changeable.** And changeability, unlike cleanliness, leaves evidence.

## The claim

John Ousterhout's *A Philosophy of Software Design* defines complexity by its **symptoms**, which is what makes it usable:

**Change amplification.** A conceptually simple change requires edits in many places. One new shift type; eleven files.

**Cognitive load.** How much you must know to make the change correctly. Not how many lines -- how many facts.

**Unknown unknowns.** You cannot tell *what* you need to know. Somewhere a caller depends on an ordering nobody documented, and you find out in production.

The third is the worst, and the ordering is the useful part of the model. Change amplification is annoying and visible. Cognitive load is expensive and visible. **Unknown unknowns are invisible, which is why they are the only one that produces incidents.**

Two causes underneath all three: **dependencies** (this cannot be understood or changed alone) and **obscurity** (important information is not apparent). Every technique in this series attacks one of those two.

## The objection

Four, and they matter, because a metric adopted uncritically does more damage than no metric.

**Goodhart's law.** When a measure becomes a target, it stops being a good measure. Optimize change amplification and you get an abstraction layer per axis of change; optimize cognitive load and you get a flat codebase with duplication. **The three trade against each other.** They are a diagnostic, not a score.

**"Unknown unknowns" cannot be measured, by definition.** You can only count them after the fact -- incidents, rollbacks, hotfixes for things nobody predicted. That makes the third symptom a *lagging* indicator, and lagging indicators cannot steer a refactoring. What you can do is watch its leading proxy: **undocumented contracts** -- "call this only after that", "this list must stay sorted".

**Changeability is relative to the change.** David Parnas made this point in 1972 and it has never been improved on: you modularize around **the changes you expect**. Pick the wrong axis and the modularization actively obstructs you. This is the same thing the [design patterns series]({{ "/design_patterns/m0-fundamentals/" | relative_url }}) says about OCP -- it is not free, because it requires committing to an axis in advance.

**The Clean Code position.** Martin would say measurement is beside the point: what makes code stay workable is a habit, applied constantly, at small scale -- leave each file better than you found it. And he is not wrong that no measurement has ever cleaned anything by itself.

Take the measurement as evidence and the habit as the mechanism. The measurement's job is to tell you **where** to apply the habit, because a codebase has no uniform quality and attention is finite.

## The technique

Git history already contains the change-amplification data. It costs one command to get at it.

```bash
git log --since="6 months ago" --pretty=format:__C__ --name-only -- '*.kt' > cochange.txt
```

Then group by commit and count pairs. For two files, **co-changes divided by the smaller file's total changes** gives a coupling ratio: *when this file changes, how often does that one change too?*

```python
import collections, itertools, io

commits, cur = [], []
for line in io.open('cochange.txt', encoding='utf-8'):
    line = line.rstrip('\n')
    if line == '__C__':
        if cur: commits.append(cur)
        cur = []
    elif line.strip():
        cur.append(line)
if cur: commits.append(cur)

commits = [c for c in commits if 1 < len(set(c)) <= 25]   # drop single-file and sweeping commits

freq, pair = collections.Counter(), collections.Counter()
for c in commits:
    s = sorted(set(c)); freq.update(s)
    for a, b in itertools.combinations(s, 2): pair[(a, b)] += 1

rows = [(n / min(freq[a], freq[b]), n, a, b) for (a, b), n in pair.items() if n >= 5]
for ratio, n, a, b in sorted(rows, reverse=True)[:15]:
    print(f"{ratio:5.0%} ({n:2}x) {a}\n            + {b}")
```

The 25-file ceiling matters. A rename sweep or a dependency bump touches everything and would couple the whole codebase to itself.

### Reading the output

**Most of the top of the list will be a file and its test.** That is the metric working, not a finding -- if a source file and its test changed together every time, your tests are doing their job. Skip those rows.

What you are looking for is three things:

1. **Pairs that cross a module boundary at a high ratio.** A boundary that two files cross every single time is not carrying its weight. Either the split is in the wrong place, or one side is leaking.
2. **A file coupled to something it should not know about.** A screen and a shared design-token file changing together at 100% means tokens is not a shared vocabulary -- it is a place people add things for the screen they are working on.
3. **Files with high frequency that are supposed to be stable.** Configuration, tokens, constants. Churn in a file whose purpose is to stop churn is a contradiction worth looking at.

### Hotspots

The second query is **frequency times size**. A large file nobody touches is not urgent; a small file touched constantly is probably fine; the intersection is where change cost accumulates.

```python
score = changes * lines
```

Crude, and good enough. Replacing `lines` with cognitive complexity refines it, and rarely changes the top five.

### What the history cannot tell you

**Renames break it.** `git log --follow` works for one file at a time and not for this analysis. A package rename splits every file's history in two and quietly halves its apparent frequency, so check for one before trusting the numbers.

**Six months is a choice.** Too short and you measure the current sprint; too long and you measure a codebase that no longer exists. Run it at two windows and compare -- a pair that is coupled in both is structural, one that appears only in the short window is a project.

## What Kotlin does to the three

Worth naming, because the language moves these numbers directly.

**Change amplification: unchanged.** A `sealed` hierarchy does not reduce the number of places that must change when a variant is added.

**Unknown unknowns: converted into known ones.** But the exhaustive `when` *names every one of those places at compile time*. That is the whole value, and it is a different axis from the one people usually claim for sealed classes. It is the same trade the [patterns series]({{ "/design_patterns/m4-visitor/" | relative_url }}) works through on Visitor.

**Cognitive load: reduced by subtraction.** Null safety, immutability by default, and `data class` equality each remove a category of question you would otherwise have to ask about every reference.

**Obscurity: reduced by `internal`.** Anything not visible outside the module is a thing no caller can depend on, which shrinks the surface where undocumented contracts can form.

## In the wild

- **Adam Tornhill's behavioral code analysis** (*Your Code as a Crime Scene*, and the CodeScene tool) is the fullest development of the co-change idea -- hotspots, temporal coupling, and knowledge distribution from the same history.
- **SonarSource's cognitive complexity** is a deliberate replacement for cyclomatic complexity, which counts branches and therefore rates a flat 10-branch `when` as harder than three levels of nesting. It is not.
- **Google's large-scale-change tooling** exists because change amplification at that size is an infrastructure problem rather than a code-review problem.

## When to stop

**Measure once, fix one thing, measure again.** Do not build a dashboard. A number watched continuously becomes a target, and Goodhart takes it from there.

**Only act on files that are actually changing.** A hotspot is frequency times size; a large file with no changes is not a problem you have. The analysis is useful precisely because it refuses to care about code nobody touches.

**One change per finding, shipped.** The failure mode of this whole subject is a six-month refactor with nothing delivered.

## Exercise

Run both queries on a repository you own, over six months.

Then answer one question honestly: **does the top cross-boundary pair surprise you?**

If it does, you have found an unknown unknown -- a dependency that exists and that you did not have a model of. That is the only symptom of the three you cannot find by reading, and it is the reason to run this at all.
