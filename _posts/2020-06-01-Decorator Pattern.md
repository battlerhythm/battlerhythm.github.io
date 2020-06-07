---
title: "Design Patterns - Decorator Pattern"
excerpt_separator: "<!--more-->"
categories:
  - Blog
tags:
  - Design Patterns
  - Decorator Pattern
---

Key words: Class explosion, Recursive function, Deprecated function

## UML  

### Not Good Design

![Classes](http://www.plantuml.com/plantuml/proxy?src=https://raw.githubusercontent.com/battlerhythm/battlerhythm.github.io/master/assets/umls/decorator-pattern1.puml)

### Better Design

![Decorator](http://www.plantuml.com/plantuml/proxy?src=https://raw.githubusercontent.com/battlerhythm/battlerhythm.github.io/master/assets/umls/decorator-pattern2.puml)

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
