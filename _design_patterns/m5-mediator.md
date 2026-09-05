---
title: "Mediator"
order: 19
module: "M5"
module_title: "Communication Between Objects"
session: "10-11"
gof: "Behavioral"
kotlin: "intact"
kotlin_feature: "No language feature replaces it. Watch the hub for God Object growth."
intent: "Replace direct references between objects with a hub that coordinates them."
status: "outline"
excerpt_separator: "<!--more-->"
categories:
  - Design Patterns
tags:
  - "Mediator"
  - "Behavioral"
confused_with:
  - "m5-observer"
---

The other way to cut coupling. Where Observer's publisher knows nothing about its subscribers, a mediator knows all of them and holds the coordination logic. That is its value and its failure mode: every rule you cannot place anywhere else ends up in the hub, and the hub becomes a God Object.

<!--more-->

## Structure

```mermaid
classDiagram
    class Mediator {
        <<interface>>
        +notify(sender, event)
    }
    class DialogMediator {
        +notify(sender, event)
    }
    class Component {
        #mediator: Mediator
    }
    class Button
    class TextField
    class Checkbox
    Mediator <|.. DialogMediator
    Component o--> Mediator : knows only the hub
    Component <|-- Button
    Component <|-- TextField
    Component <|-- Checkbox
```
