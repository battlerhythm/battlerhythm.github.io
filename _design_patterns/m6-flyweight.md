---
title: "Flyweight"
order: 23
module: "M6"
module_title: "State, Resources, Two-Axis Growth"
session: "12"
gof: "Structural"
kotlin: "intact"
kotlin_feature: "No language feature replaces it. You build the pool yourself."
intent: "Share common intrinsic state across many objects to cut memory."
status: "outline"
excerpt_separator: "<!--more-->"
categories:
  - Design Patterns
tags:
  - "Flyweight"
  - "Structural"
---

Share intrinsic state so a large number of objects costs less memory. One of the few patterns no Kotlin feature replaces, and one you will not need until you suddenly do -- icon and glyph caches, tile rendering, very large lists. The discipline it teaches is separating state that is intrinsic to the object from state that belongs to the context using it.

<!--more-->

## Structure

```mermaid
classDiagram
    class FlyweightFactory {
        -pool
        +get(key) Flyweight
    }
    class Flyweight {
        -intrinsicState
        +operation(extrinsicState)
    }
    class Client {
        -extrinsicState
    }
    FlyweightFactory o--> Flyweight : shared pool
    Client --> FlyweightFactory
    Client ..> Flyweight : passes extrinsic state in
```
