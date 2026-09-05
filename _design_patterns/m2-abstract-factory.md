---
title: "Abstract Factory"
order: 6
module: "M2"
module_title: "Separating Object Creation"
session: "4-5"
gof: "Creational"
kotlin: "intact"
kotlin_feature: "No language feature replaces it. Write it as GoF describes."
intent: "Create whole families of related objects that must be used together, without naming their concrete classes."
status: "outline"
excerpt_separator: "<!--more-->"
categories:
  - Design Patterns
tags:
  - "Abstract Factory"
  - "Creational"
confused_with:
  - "m2-factory-method"
  - "m2-builder"
redirect_from:
  - "/design_patterns/abstract_factory_pattern/"
---

Factory Method scaled from one product to a family that has to be used together. This is one of the five patterns Kotlin gives you nothing for: no language feature expresses \"these objects must all come from the same set\".

<!--more-->

## Structure

```mermaid
classDiagram
    class GuiFactory {
        <<interface>>
        +createButton() Button
        +createCheckbox() Checkbox
    }
    class MacFactory
    class WinFactory
    class Button {
        <<interface>>
    }
    class Checkbox {
        <<interface>>
    }
    GuiFactory <|.. MacFactory
    GuiFactory <|.. WinFactory
    MacFactory ..> Button : creates
    MacFactory ..> Checkbox : creates
```
