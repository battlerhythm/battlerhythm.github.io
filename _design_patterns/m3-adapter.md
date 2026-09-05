---
title: "Adapter"
order: 10
module: "M3"
module_title: "Wrappers: The Four Siblings"
session: "6-7"
gof: "Structural"
kotlin: "replaced"
kotlin_feature: "An extension function is often the whole adapter."
intent: "Change an interface so two otherwise incompatible things can work together."
status: "outline"
excerpt_separator: "<!--more-->"
categories:
  - Design Patterns
tags:
  - "Adapter"
  - "Structural"
confused_with:
  - "m3-decorator"
  - "m3-facade"
  - "m6-bridge"
redirect_from:
  - "/design_patterns/adapter_pattern/"
---

First of the four wrappers. All four wrap another object and forward calls; only their intent differs. Adapter is the one that *changes* the interface so two incompatible things can meet. Learn the four together or you will confuse them forever.

<!--more-->

## Structure

```mermaid
classDiagram
    class Client
    class Target {
        <<interface>>
        +request()
    }
    class Adapter {
        +request()
    }
    class Adaptee {
        +specificRequest()
    }
    Client --> Target
    Target <|.. Adapter
    Adapter o--> Adaptee : wraps
```
