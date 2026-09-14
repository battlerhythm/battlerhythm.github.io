---
title: "The One-Page Version"
order: 16
module: "M6"
module_title: "Applying It"
session: "12-13"
intent: "The whole series with the arguments removed -- a procedure short enough to run from memory when you are actually afraid of a file."
status: "complete"
excerpt_separator: "<!--more-->"
categories:
  - Code You Can Change
tags:
  - "The One-Page Version"
source: "Feathers"
---

Everything in this series, with the reasoning stripped out. Fifteen posts exist to justify this page; this page is the only part you need in the moment.

<!--more-->

## Before anything

**Name the change in one sentence, in terms of observable behavior.**

> *"Filing a shift application should stamp the server's time, not the device's."*

If you cannot write that sentence, stop. You are not refactoring -- you have a feeling about a file, and a feeling has no stopping condition.

If you are not afraid of the code, skip this entire page and make the change. **This procedure is for code whose behavior you do not know.** Running it on code you understand is ceremony.

## 1. Verify the finding

Findings are hypotheses with timestamps. Before acting on one:

- **Read the code.** Does it actually do what the note says?
- **Read the callers.** This is where the finding usually is.
- **Look for defensive code at a consumer** -- a clear, a guard, a retry, a null check that should be impossible. That is a bug report written by someone who decided not to file one.

If reading cannot settle it, **the pin settles it.** Skip to step 4 and write the test as a question.

## 2. Measure, do not judge

| Question | Command |
|---|---|
| Which module is legacy? | test lines / source lines, per module |
| Which file in it? | for each file, count test files mentioning it by name |
| What changes together? | co-change frequency from `git log` |
| Is that commit safe to review? | `\|ins − del\| / (ins + del)` — near 0 is mechanical |

```bash
# files that change together, top 20
git log --pretty=format:%H --name-only -- '*.kt' | awk '
  /^[0-9a-f]{40}$/ { for(i=0;i<n;i++) for(j=i+1;j<n;j++) print f[i]" + "f[j]; n=0; next }
  NF && n<40 { f[n++]=$0 }
' | sort | uniq -c | sort -rn | head -20

# refactor commits that modified an existing assertion
git log --pretty='%H %s' | grep ' refactor' | cut -d' ' -f1 | while read h; do
  git show "$h" -- '*Test.kt' | grep -qE '^-\s+assert' && git log -1 --oneline "$h"
done
```

Legacy is a property of a **region**, not a codebase. You are looking for where, not whether.

## 3. Dependency table

One row per collaborator **on the path of this change**. Not the class's dependencies in general.

| Dependency | Enabling point |
|---|---|

Include the invisible ones: **clocks, random, environment, global `object`s, constructor calls inside method bodies.**

For each: *where would I decide, from outside this file, to use something else?*

**Blank rows are the work.** Usually one or two. Usually a clock.

## 4. Cheapest seam

**Rung 0: look for one that already exists.** Another test may have built it months ago for a different feature.

Then, first one that works:

| | Seam | Cost |
|---|---|---|
| 1 | `now: () -> Long = { Clock.System.now()… }` — default function-type parameter | one signature |
| 2 | Interface as a constructor parameter | one type |
| 3 | `internal` + the test source set | no public API change |
| 4 | `expect`/`actual` | enabling point is the build — coarse |
| 5 | `open` + subclass | **don't** — see the `final` post |

**Test of a good seam: zero production call sites change.**

Commit alone, labelled `refactor`. This commit is provable precisely because the change has not happened yet.

## 5. Pin

Characterization tests, in their own commit, passing against **unchanged** code.

1. Assert something you believe is **false**
2. Run it — the failure message is the answer
3. Paste the real value in
4. Repeat for the behaviors this change could break

Rules:

- **Fake at the outermost boundary** (raw HTTP, raw file API), so the test crosses the real production path
- **Fakes, not mocks.** A mock pins *how*; you want *what*
- **Pin both directions** — the return value and what the subject sent
- **Label known-wrong behavior** in the test, with a date and a reason. An unlabelled pin becomes a specification by accident
- **Do not pin** private state, call order, timing, or log output

## 6. Make the change

Now, and in a separate commit labelled `fix` or `feat`.

**The pin updates in this commit**, because the pin's expected value is exactly what you are changing. That is the mechanism: a pin cannot be quietly updated during a structural commit, so the behavior change has to declare itself.

## 7. Stop

- The change you came for is **safe**, not the file **clean**
- The seam you needed **exists**, not the class **well-designed**
- Do not sweep for coverage. Add tests where a change is arriving
- Do not refactor code nobody is about to touch
- Three sprouts into one region — the next change extracts it
- Promote each pin to a real test, or delete it. Neither is how "codifying bugs" comes true

## The commit shape

```
refactor(x): <seam>      -- zero call sites changed, provable
test(x): pin <behavior>  -- passes against unchanged code
fix(x): <the change>     -- behavior and its pin, together
```

Three hats, never two on one head. Reverting the third leaves the seam and the pins. Bisect through them is decisive. **None of that is true of the one commit that would have contained all three.**

## Three questions that compress the series

Ask at every step. Each is the whole of one module.

> **1. Where is the enabling point?** *(M3, M4 — seams and dependency breaking)*
>
> **2. Is this information already somewhere in the code?** *(M2 — names, depth, comments)*
>
> **3. Did behavior change in this commit?** *(M5 — the only rule that does not bend)*

## What this is not

**Not a quality program.** Nothing here improves a codebase in the abstract. Every step is paid for by one pending change.

**Not a rewrite plan.** A rewrite is a behavior change with no pins. If you cannot state what the system does precisely enough to verify a replacement, a rewrite will drop behavior nobody wrote down.

**Not a reason to postpone the change.** If a step is not blocking you, skip it. The seam that already exists, the pin that answers a question you already know the answer to -- skipping those is the procedure working, not being cheated.

## The point of all of it

The output is not a cleaner file.

> **The output is a file where the next change is cheaper than the last one.**

That is measurable, it compounds, and it is the only claim in this series that does not depend on anyone's taste.

---

*Series index: [Code You Can Change]({{ "/changing_code/" | relative_url }}) · [Design Patterns]({{ "/design_patterns/" | relative_url }})*
