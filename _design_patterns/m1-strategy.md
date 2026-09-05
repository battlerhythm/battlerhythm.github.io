---
title: "Strategy"
order: 2
module: "M1"
module_title: "Swapping Algorithms"
session: "2-3"
gof: "Behavioral"
kotlin: "replaced"
kotlin_feature: "Function types and lambdas: `val discount: (Money) -> Money`."
intent: "Encapsulate a family of algorithms as objects and swap them at runtime; the client picks."
status: "outline"
excerpt_separator: "<!--more-->"
categories:
  - Design Patterns
tags:
  - "Strategy"
  - "Behavioral"
confused_with:
  - "m1-state"
  - "m1-template-method"
  - "m6-bridge"
  - "m5-command"
---

The first pattern in the curriculum, and the reference point for the next two. Strategy, Template Method and State draw almost the same class diagram; what separates them is who decides. Here the client picks the algorithm and hands it in. Hold onto that fact -- it is the entire difference from State.

<!--more-->

## Structure

```mermaid
classDiagram
    class Context {
        -strategy: Strategy
        +execute(data)
    }
    class Strategy {
        <<interface>>
        +run(data)
    }
    class ConcreteStrategyA {
        +run(data)
    }
    class ConcreteStrategyB {
        +run(data)
    }
    Context o--> Strategy
    Strategy <|.. ConcreteStrategyA
    Strategy <|.. ConcreteStrategyB
```
