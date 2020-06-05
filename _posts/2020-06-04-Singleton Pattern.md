---
title: "Design Patterns - Singleton Pattern"
excerpt_separator: "<!--more-->"
categories:
  - Blog
tags:
  - Design Patterns
  - Singleton Pattern
---

Key words: Global access point of the instance

## UML  

### Abstract Factory Pattern

![Composite](http://www.plantuml.com/plantuml/proxy?src=https://raw.githubusercontent.com/battlerhythm/battlerhythm.github.io/master/assets/umls/singleton-pattern.puml)

## Pseudo Code

```java

class Singleton {
  static private Singleton instance;

  private Singleton() {}

  public static Singleton getInstance() {
    if(instance == null) {
      instance = new Singleton();
    }
    return instance;
  }
}

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

- [https://youtu.be/v-GiuMmsXj4](https://youtu.be/v-GiuMmsXj4)
