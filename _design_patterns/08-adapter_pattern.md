---
title: "Design Patterns - Adapter Pattern"
permalink: /design_patterns/adapter_pattern/
excerpt_separator: "<!--more-->"
toc: true
categories:
  - Design Patterns
tags:
  - Adapter Pattern
---

Key words: Not change under laying behavior

## UML  

### Adapter Pattern

![Adapter]({{ "/assets/images/umls/adapter-pattern.png" | relative_url }})

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
