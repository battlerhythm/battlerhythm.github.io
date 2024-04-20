---
title: "Design Patterns - Singleton Pattern"
excerpt_separator: "<!--more-->"
categories:
  - Design Patterns
tags:
  - Singleton Pattern
---

Key words: Global access point of the instance, Code Smell

## UML  

### Singleton Pattern

![Singleton](http://www.plantuml.com/plantuml/proxy?src=https://raw.githubusercontent.com/battlerhythm/battlerhythm.github.io/master/assets/umls/singleton-pattern.puml)

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
```

## Reference

- [https://youtu.be/hUE_j6q0LTQ](https://youtu.be/hUE_j6q0LTQ)
