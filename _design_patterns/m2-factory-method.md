---
title: "Factory Method"
order: 5
module: "M2"
module_title: "Separating Object Creation"
session: "4-5"
gof: "Creational"
kotlin: "reshaped"
kotlin_feature: "`companion object` with `operator fun invoke()` reads like a constructor."
intent: "Defer the choice of which concrete class to instantiate to a subclass."
status: "outline"
excerpt_separator: "<!--more-->"
categories:
  - Design Patterns
tags:
  - "Factory Method"
  - "Creational"
confused_with:
  - "m2-abstract-factory"
redirect_from:
  - "/design_patterns/factory_method_pattern/"
---

The first of the creational five. Calling a constructor binds you to a concrete type; Factory Method defers that choice to a subclass. Note that it produces one product -- the moment several products have to match each other, you have crossed into Abstract Factory.

<!--more-->

## Structure

```mermaid
classDiagram
    class Creator {
        +operation()
        #createProduct()* Product
    }
    class ConcreteCreator {
        #createProduct() Product
    }
    class Product {
        <<interface>>
    }
    class ConcreteProduct
    Creator <|-- ConcreteCreator
    Product <|.. ConcreteProduct
    ConcreteCreator ..> ConcreteProduct : creates
```
