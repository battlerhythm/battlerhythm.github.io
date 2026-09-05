---
title: "Chain of Responsibility"
order: 20
module: "M5"
module_title: "Communication Between Objects"
session: "10-11"
gof: "Behavioral"
kotlin: "reshaped"
kotlin_feature: "A list of interceptors walked in order -- the shape OkHttp and Ktor both use."
intent: "Pass a request along a chain until some handler takes it, and can stop it."
status: "outline"
excerpt_separator: "<!--more-->"
categories:
  - Design Patterns
tags:
  - "Chain of Responsibility"
  - "Behavioral"
confused_with:
  - "m3-decorator"
---

Structurally a chain, like Decorator, with one decisive difference: a handler may stop the request instead of passing it on. That single capability is what makes it right for auth, rate limiting, and routing. OkHttp's interceptor stack is this pattern in production, which makes it easy to study from real code.

<!--more-->

## Structure

```mermaid
classDiagram
    class Handler {
        <<interface>>
        +setNext(h) Handler
        +handle(req)
    }
    class AuthHandler {
        +handle(req)
    }
    class RateLimitHandler {
        +handle(req)
    }
    class RouteHandler {
        +handle(req)
    }
    Handler <|.. AuthHandler
    Handler <|.. RateLimitHandler
    Handler <|.. RouteHandler
    Handler o--> Handler : next, may stop here
```
