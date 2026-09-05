---
title: "Composite"
order: 14
module: "M4"
module_title: "Trees and Recursion"
session: "8-9"
gof: "Structural"
kotlin: "reshaped"
kotlin_feature: "`sealed interface` with a child list; exhaustive `when` for the recursion."
intent: "Treat individual objects and compositions of objects uniformly."
status: "outline"
excerpt_separator: "<!--more-->"
categories:
  - Design Patterns
tags:
  - "Composite"
  - "Structural"
confused_with:
  - "m3-decorator"
redirect_from:
  - "/design_patterns/composite_pattern/"
---

Opens the tree module. Treat a leaf and a branch through the same interface and recursion becomes possible. Everything else in this module operates on the tree that Composite creates -- Iterator walks it, Visitor adds operations to it, Interpreter is the special case where the tree is a grammar.

<!--more-->

## Structure

```mermaid
classDiagram
    class Component {
        <<interface>>
        +operation()
    }
    class Leaf {
        +operation()
    }
    class Composite {
        -children: List~Component~
        +operation()
        +add(c)
        +remove(c)
    }
    Component <|.. Leaf
    Component <|.. Composite
    Composite o--> Component : many children
```
