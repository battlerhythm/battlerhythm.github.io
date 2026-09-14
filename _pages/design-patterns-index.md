---
title: "Design Patterns, in Learning Order"
permalink: /design_patterns/
layout: single
author_profile: false
toc: false
sidebar:
  nav: "design_patterns"
---

The Gang of Four book files its twenty-three patterns under **Creational / Structural / Behavioral**. That taxonomy is useful for looking something up and terrible for learning, for one reason: the patterns that are easiest to confuse sit in different chapters. Strategy and Bridge draw the same class diagram. So do Strategy and State. Learn them weeks apart and "so what's the difference?" never gets resolved.

So this series is ordered **by which patterns look alike**, and each module ends by naming what separates them. Because what distinguishes two patterns is never the structure -- it is the intent.

Code is Kotlin. Every pattern is tagged with what the language did to it: **replaced** (a language feature does the job), **reshaped** (there is an idiomatic Kotlin form worth knowing), or **as-is** (nothing in the language helps -- write it the way GoF describes).

## The map

{% include series_map.html collection="design_patterns" %}

## Before adopting any of them

Three gates, all of which must pass:

1. **Has the change actually arrived twice?** No predicting. Once is a coincidence; twice is a pattern.
2. **Is there exactly one clear axis of variation?** Two means consider Bridge. Unclear means it is too early.
3. **Does the code get shorter, or at least easier to follow, afterward?** If there are now three more files and the flow is harder to trace, that is a failure.

See also: [Confusing Pairs]({{ "/design_patterns/confusing-pairs/" | relative_url }}) -- the patterns whose structure is identical and whose intent is not.
