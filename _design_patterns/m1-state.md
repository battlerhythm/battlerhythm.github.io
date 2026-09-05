---
title: "State"
order: 4
module: "M1"
module_title: "Swapping Algorithms"
session: "2-3"
gof: "Behavioral"
kotlin: "reshaped"
kotlin_feature: "`sealed class` for the states plus a transition function; the GoF form still wins when behavior per state is large."
intent: "Let an object change its behavior wholesale as its internal state changes; the object owns the transitions."
status: "outline"
excerpt_separator: "<!--more-->"
categories:
  - Design Patterns
tags:
  - "State"
  - "Behavioral"
confused_with:
  - "m1-strategy"
---

The same diagram as Strategy, with a different owner. The client never picks a state -- the states hand control to one another. The question to keep asking through this module is \"who knows about the transition?\" If the answer is the object itself, this is State.

<!--more-->

## Structure

```mermaid
classDiagram
    class Context {
        -state: State
        +handle()
    }
    class State {
        <<interface>>
        +handle(ctx) State
    }
    class Idle {
        +handle(ctx) State
    }
    class Running {
        +handle(ctx) State
    }
    Context o--> State
    State <|.. Idle
    State <|.. Running
    Idle --> Running : transitions to
```
