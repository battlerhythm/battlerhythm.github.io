---
title: "Decorator"
order: 11
module: "M3"
module_title: "Wrappers: The Four Siblings"
session: "6-7"
gof: "Structural"
kotlin: "replaced"
kotlin_feature: "`class Logged(private val inner: Repo) : Repo by inner` -- override what you intercept, delegate the rest."
intent: "Keep the interface and add responsibilities, stackable at runtime."
status: "outline"
excerpt_separator: "<!--more-->"
categories:
  - Design Patterns
tags:
  - "Decorator"
  - "Structural"
confused_with:
  - "m3-proxy"
  - "m3-adapter"
  - "m4-composite"
  - "m5-chain-of-responsibility"
redirect_from:
  - "/design_patterns/decorator_pattern/"
---

Keeps the interface and adds responsibility, and it stacks. Its diagram is identical to Proxy's -- the difference is that a decorator adds behavior while a proxy controls access to the same behavior. Kotlin's interface delegation makes a decorator a one-liner, which is why this is the wrapper you will actually reach for.

<!--more-->

## Structure

```mermaid
classDiagram
    class Component {
        <<interface>>
        +operation()
    }
    class ConcreteComponent {
        +operation()
    }
    class Decorator {
        #inner: Component
        +operation()
    }
    class LoggingDecorator {
        +operation()
    }
    class CachingDecorator {
        +operation()
    }
    Component <|.. ConcreteComponent
    Component <|.. Decorator
    Decorator o--> Component : wraps one
    Decorator <|-- LoggingDecorator
    Decorator <|-- CachingDecorator
```
