---
title: "Design Patterns - Strategy Pattern"
excerpt_separator: "<!--more-->"
categories:
  - Blog
tags:
  - Design Patterns
  - Strategy Pattern
---

Key words: Dependency injection

## UML

### Not Good Design  

![Inheritance](/assets/images/strategy-pattern1.png)

### Better Design  

![Composite](http://www.plantuml.com/plantuml/proxy?src=https://raw.githubusercontent.com/battlerhythm/battlerhythm.github.io/master/assets/umls/strategy-pattern2.puml)

## Code

```java
class Duck {
  IFlyBehavior fb;
  IQuackBehavior qb;
  IDisplayBehavior db;

  public Duck (IFlyBehavior fb, IQuackBehavior fb,
               IDisplayBehavior db) {
    this.fb = fb;
    this.qb = qb;
    this.db = db;
  }

  public void fly() {
    this.fb.fly()
  }

  public void quack() {
    this.qb.fly()
  }

  public void display() {
    this.db.display()
  }
}
```

## Reference

- [https://youtu.be/v9ejT8FO-7I](https://youtu.be/v9ejT8FO-7I)
