---
title: "Memento"
order: 22
module: "M6"
module_title: "State, Resources, Two-Axis Growth"
session: "12"
gof: "Behavioral"
kotlin: "replaced"
kotlin_feature: "An immutable `data class` snapshot makes this nearly free."
intent: "Capture and restore an object's state without breaking encapsulation."
status: "outline"
excerpt_separator: "<!--more-->"
categories:
  - Design Patterns
tags:
  - "Memento"
  - "Behavioral"
---

Save and restore state without exposing the object's internals. Immutable snapshots make the mechanics trivial in Kotlin, so the real work is deciding what belongs in a snapshot -- too much and undo is expensive, too little and it is wrong.

<!--more-->

## Structure

```mermaid
classDiagram
    class Originator {
        -state
        +save() Memento
        +restore(m)
    }
    class Memento {
        -state
        +getState()
    }
    class Caretaker {
        -history: List~Memento~
        +backup()
        +undo()
    }
    Originator ..> Memento : creates
    Caretaker o--> Memento : stores, cannot read
    Caretaker --> Originator
```
