---
title: "Design Patterns - Structural Patterns (Comparison)"
permalink: /design_patterns/structural_patterns_comparison/
excerpt_separator: "<!--more-->"
toc: true
categories:
  - Design Patterns
tags:
  - Structural Patterns
---

## [Decorator](/design_patterns/decorator_pattern/)

Decorator pattern extends behavior by decorating component.

## [Strategy (Behavioral)](/design_patterns/strategy_pattern/)

The intent of the Strategy pattern is to define a set of algorithms or a family of algorithms where you can exchange one of the algorithms for the other you can vary the algorithms independently from the context of the things that uses the algorithms.

## [Adapter](/design_patterns/adapter_pattern/)

Adapter pattern adapts from one interface to another interface you want to use a particular interface but the thing that you want to use has a different interface you stick and adapter in between so that you can use the thing you want to use.

## [Proxy](/design_patterns/proxy_pattern/)

Proxy controls access while the adapter says I only change the interface, I don't want to change implementation. The proxy says I change the implementation but I don't want to change interface. So, the proxy is a thing but uses the same interfaces so you can place the proxy instead of the thing that you wanted to use because it has the same interface it's interchangeable but then the proxy controls access to that things that you placed it in front of.

## [Facade](/design_patterns/facade_pattern/)

Facade has none of this dependency injection or that stuff. The facade is a term that we use simply when you have a scenario where you have complex subsystem and then you throw a facade in between you say here's the wall use this simpler thing and this simpler thing will interact with all that complexity from the subsystem and again this is not the same as proxy because it doesn't necessarily require the facade to have the same interface as anything in the subsystem and it's not the same as the adapter because it doesn't necessarily adapt from one interface to another it. I mean the facade has a single interface but the subsystem might have a bazillion interfaces. So it's simply a way of hiding some complexity of some subsystem.

## [Bridge](/design_patterns/bridge_pattern/)

The Bridge pattern is perhaps actually most easily thought of as an extension of the strategy pattern. It's like the strategy pattern but where the context that uses the strategy also is an inheritance hierarchy or also is a polymorphic hierarchy so you have two inheritance hierarchy or two polymorphic hierarchies and one of the hierarchies uses things from the other hierarchies so you can take anything from the first hierarchy and pair it up with anything from second hierarchy and that's a valid pair so in some sense it's a way of avoiding duplication of code and increasing reusability.

## Reference

- [https://youtu.be/lPsSL6_7NBg](https://youtu.be/lPsSL6_7NBg)
