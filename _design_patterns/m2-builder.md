---
title: "Builder"
order: 7
module: "M2"
module_title: "Separating Object Creation"
session: "4-5"
gof: "Creational"
kotlin: "replaced"
kotlin_feature: "Named and default arguments cover the common case; `@DslMarker` builders cover the rest."
intent: "Separate the construction of a complex object from its representation."
status: "outline"
excerpt_separator: "<!--more-->"
categories:
  - Design Patterns
tags:
  - "Builder"
  - "Creational"
confused_with:
  - "m2-abstract-factory"
---

Kotlin takes most of this pattern away. Named and default arguments handle the ordinary case; what survives is step-by-step validation, type-safe builder DSLs, and Java interop. Worth studying precisely to see where the language stops helping.

<!--more-->

## Structure

```mermaid
classDiagram
    class Builder {
        <<interface>>
        +setPartA(v) Builder
        +setPartB(v) Builder
        +build() Product
    }
    class ConcreteBuilder
    class Director {
        +construct(b) Product
    }
    class Product
    Builder <|.. ConcreteBuilder
    Director o--> Builder
    ConcreteBuilder ..> Product : builds
```
