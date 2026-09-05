---
title: "Observer"
order: 18
module: "M5"
module_title: "Communication Between Objects"
session: "10-11"
gof: "Behavioral"
kotlin: "reshaped"
kotlin_feature: "`Flow`, `StateFlow`, `SharedFlow`; `Delegates.observable` for a single property."
intent: "Notify a set of subscribers automatically when state changes; the publisher does not know who is listening."
status: "outline"
excerpt_separator: "<!--more-->"
categories:
  - Design Patterns
tags:
  - "Observer"
  - "Behavioral"
confused_with:
  - "m5-mediator"
redirect_from:
  - "/design_patterns/observer_pattern/"
---

Opens the communication module. The publisher not knowing its subscribers is the whole point. Read it against `Flow` and `StateFlow` and ask which of the classic Observer problems -- leaked subscriptions, undefined delivery order, no backpressure -- the coroutine library actually solved, and which it just moved.

<!--more-->

## Structure

```mermaid
classDiagram
    class Subject {
        -observers: List~Observer~
        +attach(o)
        +detach(o)
        +notify(event)
    }
    class Observer {
        <<interface>>
        +update(event)
    }
    class ConcreteObserverA {
        +update(event)
    }
    class ConcreteObserverB {
        +update(event)
    }
    Subject o--> Observer : does not know which
    Observer <|.. ConcreteObserverA
    Observer <|.. ConcreteObserverB
```
