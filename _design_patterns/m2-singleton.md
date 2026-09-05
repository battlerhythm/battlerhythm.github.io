---
title: "Singleton"
order: 9
module: "M2"
module_title: "Separating Object Creation"
session: "4-5"
gof: "Creational"
kotlin: "replaced"
kotlin_feature: "`object` declaration -- thread-safe and lazily initialized, which is exactly what makes it too easy."
intent: "Force a class to have exactly one instance and give it a global access point."
status: "outline"
excerpt_separator: "<!--more-->"
categories:
  - Design Patterns
tags:
  - "Singleton"
  - "Creational"
redirect_from:
  - "/design_patterns/singleton_pattern/"
---

The pattern its own authors came to regret: global state that tests cannot replace. Kotlin's `object` reduces it to a single keyword, which makes it more dangerous, not less. Half of this session is about when to reach for dependency injection instead.

<!--more-->

## Structure

```mermaid
classDiagram
    class Singleton {
        -instance$ Singleton
        -Singleton()
        +getInstance()$ Singleton
        +operation()
    }
    note for Singleton "Global state. Cannot be substituted in tests."
```
