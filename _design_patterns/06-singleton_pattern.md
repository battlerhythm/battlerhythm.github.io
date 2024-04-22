---
title: "Design Patterns - Singleton Pattern"
permalink: /design_patterns/singleton_pattern/
excerpt_separator: "<!--more-->"
toc: true
categories:
  - Design Patterns
tags:
  - Singleton Pattern
---

Key words: Global access point of the instance, Code Smell

## UML  

### Singleton Pattern

![Singleton]({{ "/assets/images/umls/singleton-pattern.png" | relative_url }})

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
