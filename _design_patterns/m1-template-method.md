---
title: "Template Method"
order: 3
module: "M1"
module_title: "Swapping Algorithms"
session: "2-3"
gof: "Behavioral"
kotlin: "replaced"
kotlin_feature: "Higher-order functions: `inline fun <T> withRetry(block: () -> T)`."
intent: "Fix the skeleton of an algorithm and let subclasses supply the varying steps."
status: "outline"
excerpt_separator: "<!--more-->"
categories:
  - Design Patterns
tags:
  - "Template Method"
  - "Behavioral"
confused_with:
  - "m1-strategy"
redirect_from:
  - "/design_patterns/template_pattern/"
---

Strategy's inheritance twin. The skeleton lives in the base class and the varying steps are pushed down to subclasses, which fixes the choice at compile time instead of injecting it at runtime. Read it directly after Strategy and the trade-off between composition and inheritance stops being abstract.

<!--more-->

## Structure

```mermaid
classDiagram
    class AbstractClass {
        +templateMethod()
        #stepOne()*
        #stepTwo()*
    }
    class ConcreteA {
        #stepOne()
        #stepTwo()
    }
    class ConcreteB {
        #stepOne()
        #stepTwo()
    }
    AbstractClass <|-- ConcreteA
    AbstractClass <|-- ConcreteB
```
