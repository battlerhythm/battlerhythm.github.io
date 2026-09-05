---
title: "Interpreter"
order: 17
module: "M4"
module_title: "Trees and Recursion"
session: "8-9"
gof: "Behavioral"
kotlin: "reshaped"
kotlin_feature: "A sealed AST plus a recursive `eval` function falls out naturally."
intent: "Represent a grammar as a class hierarchy and evaluate sentences written in it."
status: "outline"
excerpt_separator: "<!--more-->"
categories:
  - Design Patterns
tags:
  - "Interpreter"
  - "Behavioral"
confused_with:
  - "m4-composite"
---

Composite applied to a grammar. Build the sentence as a tree of expression nodes and evaluate recursively. Rare in application code and unavoidable the moment you need a filter language, a rules engine, or a query DSL of your own.

<!--more-->

## Structure

```mermaid
classDiagram
    class Expression {
        <<interface>>
        +interpret(ctx)
    }
    class NumberLiteral {
        -value
        +interpret(ctx)
    }
    class Add {
        -left: Expression
        -right: Expression
        +interpret(ctx)
    }
    class And {
        -left: Expression
        -right: Expression
        +interpret(ctx)
    }
    Expression <|.. NumberLiteral
    Expression <|.. Add
    Expression <|.. And
    Add o--> Expression : recurses
```
