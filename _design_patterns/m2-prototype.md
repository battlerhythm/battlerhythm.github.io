---
title: "Prototype"
order: 8
module: "M2"
module_title: "Separating Object Creation"
session: "4-5"
gof: "Creational"
kotlin: "replaced"
kotlin_feature: "`data class` gives you `copy()` for the shallow case."
intent: "Create new objects by cloning an existing instance."
status: "outline"
excerpt_separator: "<!--more-->"
categories:
  - Design Patterns
tags:
  - "Prototype"
  - "Creational"
---

Copy an existing instance instead of constructing a new one. `data class` and `copy()` cover the shallow case outright, so the interesting question is what happens when the copy has to be deep.

<!--more-->

## Structure

```mermaid
classDiagram
    class Prototype {
        <<interface>>
        +clone() Prototype
    }
    class ConcreteA {
        -field
        +clone() Prototype
    }
    class ConcreteB {
        -field
        +clone() Prototype
    }
    Prototype <|.. ConcreteA
    Prototype <|.. ConcreteB
```
