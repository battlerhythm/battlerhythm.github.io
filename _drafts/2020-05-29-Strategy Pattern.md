---
title: "Design Patterns - Strategy Pattern"
excerpt_separator: "<!--more-->"
categories:
  - Design Patterns
tags:
  - Strategy Pattern
---

Key words: Dependency injection

## UML

### Not Good Design  

<!-- ```mermaid
---
title: Inheritance
---

classDiagram

Duck <|-- MountainDuck
Duck <|-- CityDuck
Duck <|-- CloudDuck
Duck <|-- WildDuck
Duck <|-- RubberDuck

class Duck {
  <<Class>>
  quack()
  display()
  fly()
}

class CloudDuck {
  <<Class>>
  display()
  fly()
}

class MountainDuck {
  <<Class>>
  display()
  fly()
}

class CityDuck {
  <<Class>>
  display()
}

class WildDuck {
  <<Class>>
  display()
}

class RubberDuck {
  <<Class>>
  display()
  fly()
}
``` -->

![Inheritance]({{ "/assets/images/umls/strategy-pattern1.png" | relative_url }})

### Better Design  

<!-- ```mermaid
---
title: Composite
---

classDiagram

Duck <-- IFlyBehavior
Duck <-- IQuackBehavior
Duck <-- IDisplayBehavior
IFlyBehavior <|-- SimpleFly
IFlyBehavior <|-- JetFly
IFlyBehavior <|-- NoFly
IQuackBehavior <|-- SimpleQuack
IQuackBehavior <|-- NoQuack
IDisplayBehavior <|-- DisplayAsText
IDisplayBehavior <|-- DisplayAsGraphics

class Duck {
  <<Class>>
  quack()
  display()
  fly()
}

class IFlyBehavior {
  <<Interface>>
  fly()
}

class IQuackBehavior {
  <<Interface>>
  quack()
}

class IDisplayBehavior {
  <<Interface>>
  display()
}

class SimpleFly {
  <<Class>>
  fly()
}

class JetFly {
  <<Class>>
  fly()
}

class NoFly {
  <<Class>>
  fly()
}

class DisplayAsText {
  <<Class>>
  display()
}

class DisplayAsGraphics {
  <<Class>>
  display()
}

class SimpleQuack {
  <<Class>>
  quack()
}

class NoQuack {
  <<Class>>
  quack()
}
``` -->

![Composite]({{ "/assets/images/umls/strategy-pattern2.png" | relative_url }})

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
