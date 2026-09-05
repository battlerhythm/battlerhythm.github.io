---
title: "Bridge"
order: 24
module: "M6"
module_title: "State, Resources, Two-Axis Growth"
session: "12"
gof: "Structural"
kotlin: "intact"
kotlin_feature: "KMP's `expect`/`actual` is this idea at the language level."
intent: "Split abstraction from implementation so the two can vary independently."
status: "outline"
excerpt_separator: "<!--more-->"
categories:
  - Design Patterns
tags:
  - "Bridge"
  - "Structural"
confused_with:
  - "m1-strategy"
  - "m3-adapter"
redirect_from:
  - "/design_patterns/bridge_pattern/"
---

Split abstraction from implementation so both can grow independently. Its diagram matches Strategy's; the difference is that Strategy swaps an algorithm while Bridge separates two axes of variation. Against Adapter the difference is timing -- Bridge is prevention at design time, Adapter is repair after the fact.

<!--more-->

## Structure

```mermaid
classDiagram
    class Abstraction {
        #impl: Implementor
        +operation()
    }
    class RefinedAbstraction {
        +operation()
        +extraOperation()
    }
    class Implementor {
        <<interface>>
        +operationImpl()
    }
    class ConcreteImplA {
        +operationImpl()
    }
    class ConcreteImplB {
        +operationImpl()
    }
    Abstraction <|-- RefinedAbstraction
    Abstraction o--> Implementor : the bridge
    Implementor <|.. ConcreteImplA
    Implementor <|.. ConcreteImplB
```
