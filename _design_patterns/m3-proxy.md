---
title: "Proxy"
order: 12
module: "M3"
module_title: "Wrappers: The Four Siblings"
session: "6-7"
gof: "Structural"
kotlin: "replaced"
kotlin_feature: "`by lazy` is a virtual proxy built into the language; `by` delegation covers the rest."
intent: "Keep the interface and control access -- lazy loading, caching, permissions, remoting."
status: "outline"
excerpt_separator: "<!--more-->"
categories:
  - Design Patterns
tags:
  - "Proxy"
  - "Structural"
confused_with:
  - "m3-decorator"
  - "m3-adapter"
redirect_from:
  - "/design_patterns/proxy_pattern/"
---

Structurally identical to Decorator, with a different job: the same face, controlled access. If you are adding behavior it is a decorator; if you are deciding whether, when, or for whom the original behavior runs, it is a proxy.

<!--more-->

## Structure

```mermaid
classDiagram
    class Subject {
        <<interface>>
        +request()
    }
    class RealSubject {
        +request()
    }
    class Proxy {
        -real: RealSubject
        +request()
    }
    Subject <|.. RealSubject
    Subject <|.. Proxy
    Proxy o--> RealSubject : controls access to
```
