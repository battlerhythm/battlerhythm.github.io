---
title: "Iterator"
order: 15
module: "M4"
module_title: "Trees and Recursion"
session: "8-9"
gof: "Behavioral"
kotlin: "replaced"
kotlin_feature: "`Iterable` and `Sequence` ship with the standard library; `sequence { yield(x) }` builds a lazy one."
intent: "Walk the elements of a collection without exposing how it is stored."
status: "outline"
excerpt_separator: "<!--more-->"
categories:
  - Design Patterns
tags:
  - "Iterator"
  - "Behavioral"
---

Walking the structure without exposing it. Kotlin ships this as `Iterable` and `Sequence`, so the point of studying it is the contract -- who owns the cursor, what happens when the collection changes mid-walk -- not the implementation.

<!--more-->

## Structure

```mermaid
classDiagram
    class Aggregate {
        <<interface>>
        +iterator() Iterator
    }
    class ConcreteAggregate {
        +iterator() Iterator
    }
    class Iterator {
        <<interface>>
        +hasNext() Boolean
        +next() T
    }
    class ConcreteIterator {
        -cursor
    }
    Aggregate <|.. ConcreteAggregate
    Iterator <|.. ConcreteIterator
    ConcreteAggregate ..> ConcreteIterator : creates
```
