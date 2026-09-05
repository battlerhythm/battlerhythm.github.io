---
title: "Visitor"
order: 16
module: "M4"
module_title: "Trees and Recursion"
session: "8-9"
gof: "Behavioral"
kotlin: "reshaped"
kotlin_feature: "A sealed hierarchy with an exhaustive `when` is the opposite trade-off, not a replacement."
intent: "Add a new operation over an object structure without modifying the structure."
status: "outline"
excerpt_separator: "<!--more-->"
categories:
  - Design Patterns
tags:
  - "Visitor"
  - "Behavioral"
---

Add an operation to a structure you are not allowed to change. This is where the Expression Problem shows up: Visitor makes new *operations* cheap and new *types* expensive, while a sealed class with an exhaustive `when` makes exactly the opposite trade. Neither one replaces the other, and knowing which side you are on is the point of this session.

<!--more-->

## Structure

```mermaid
classDiagram
    class Visitor {
        <<interface>>
        +visitCircle(c)
        +visitSquare(s)
    }
    class AreaVisitor
    class RenderVisitor
    class Shape {
        <<interface>>
        +accept(v)
    }
    class Circle {
        +accept(v)
    }
    class Square {
        +accept(v)
    }
    Visitor <|.. AreaVisitor
    Visitor <|.. RenderVisitor
    Shape <|.. Circle
    Shape <|.. Square
    Circle ..> Visitor : accept(v) calls back
```
