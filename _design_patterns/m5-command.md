---
title: "Command"
order: 21
module: "M5"
module_title: "Communication Between Objects"
session: "10-11"
gof: "Behavioral"
kotlin: "reshaped"
kotlin_feature: "A function reference covers the simple case; a `data class` earns its keep once you need an undo stack."
intent: "Turn a request into an object so it can be queued, logged, and undone."
status: "outline"
excerpt_separator: "<!--more-->"
categories:
  - Design Patterns
tags:
  - "Command"
  - "Behavioral"
confused_with:
  - "m1-strategy"
redirect_from:
  - "/design_patterns/command_pattern/"
---

Turn a request into an object and queuing, logging, and undo come free. The comparison to hold is with Strategy: both wrap behavior in an object, but Strategy objectifies *how* something is done while Command objectifies *the request itself* -- which is why only Command gives you an undo stack.

<!--more-->

## Structure

```mermaid
classDiagram
    class Command {
        <<interface>>
        +execute()
        +undo()
    }
    class ConcreteCommand {
        -receiver: Receiver
        -args
        +execute()
        +undo()
    }
    class Invoker {
        -history: List~Command~
        +run(c)
        +undoLast()
    }
    class Receiver {
        +action()
    }
    Command <|.. ConcreteCommand
    Invoker o--> Command : history
    ConcreteCommand o--> Receiver
```
