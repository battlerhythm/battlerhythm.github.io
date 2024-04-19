---
title: "Design Patterns - Adapter Pattern"
excerpt_separator: "<!--more-->"
categories:
  - Design Patterns
tags:
  - Adapter Pattern
---

Key words: Not change under laying behavior

## UML  

### Adapter Pattern

![Adapter](http://www.plantuml.com/plantuml/proxy?src=https://raw.githubusercontent.com/battlerhythm/battlerhythm.github.io/master/assets/umls/adapter-pattern.puml)

## Pseudo Code

```java
interface ITarget {
  void request();
}

class Adapter implements ITarget {
  Adaptee adaptee;
  
  public Adapter(Adaptee a) {
    this.adaptee = a;
  }
  
  public void request() {
    this.adaptee.specificRequest();
  }
}

class Adaptee {
  public specificRequest() {
    // Do something
  }
}
```

```java
// From client
ITarget target = new Adapter(new Adaptee());
target.request();
```

## Reference

- [https://youtu.be/2PKQtcJjYvc](https://youtu.be/2PKQtcJjYvc)
