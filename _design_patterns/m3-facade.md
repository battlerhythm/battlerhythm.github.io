---
title: "Facade"
order: 13
module: "M3"
module_title: "Wrappers: The Four Siblings"
session: "6-7"
gof: "Structural"
kotlin: "intact"
kotlin_feature: "No language feature replaces it. It is a design decision, not a syntax problem."
intent: "Put one simple interface in front of a complicated subsystem."
status: "outline"
excerpt_separator: "<!--more-->"
categories:
  - Design Patterns
tags:
  - "Facade"
  - "Structural"
confused_with:
  - "m3-adapter"
redirect_from:
  - "/design_patterns/facade_pattern/"
---

The odd one of the four wrappers: it preserves no existing interface, it invents a simpler one in front of a subsystem. That is also how you tell it from Adapter -- an adapter matches an interface someone else defined, a facade defines its own.

<!--more-->

## Structure

```mermaid
classDiagram
    class Client
    class Facade {
        +simpleOperation()
    }
    class SubsystemA {
        +opA()
    }
    class SubsystemB {
        +opB()
    }
    class SubsystemC {
        +opC()
    }
    Client --> Facade
    Facade --> SubsystemA
    Facade --> SubsystemB
    Facade --> SubsystemC
```
