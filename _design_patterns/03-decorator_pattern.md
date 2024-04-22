---
title: "Design Patterns - Decorator Pattern"
permalink: /design_patterns/decorator_pattern/
excerpt_separator: "<!--more-->"
toc: true
categories:
  - Design Patterns
tags:
  - Decorator Pattern
---

Key words: Class explosion, Recursive function, Deprecated function

## UML  

### Not Good Design

![Classes]({{ "/assets/images/umls/decorator-pattern1.png" | relative_url }})

### Better Design

![Decorator]({{ "/assets/images/umls/decorator-pattern2.png" | relative_url }})

## Pseudo Code

```java
abstract class Beverage {
  public abstract int cost();
}

abstract class AddonDecorator extends Beverage {
  public abstract int cost();
}

class Espresso extends Beverage {
  public int cost() {
    return 1;
  }
}

class Caramel extends AddonDecorator {
  Beverage beverage;
  public Caramel(Beverage b) {
    this.beverage = b;
  }
  public int cost() {
    return this.beverage.cost() + 2;
  }
} 
```

```java
Beverage b = new Caramel(new Espresso());
b.cost();
```

## Reference

- [https://youtu.be/GCraGHx6gso](https://youtu.be/GCraGHx6gso)
