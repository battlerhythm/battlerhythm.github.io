---
title: "Design Patterns - Template Pattern"
permalink: /design_patterns/template_pattern/
excerpt_separator: "<!--more-->"
toc: true
categories:
  - Design Patterns
tags:
  - Template Pattern
---

Key words: Hook

## UML  

### Template Pattern

![Template](http://www.plantuml.com/plantuml/proxy?src=https://raw.githubusercontent.com/battlerhythm/battlerhythm.github.io/master/assets/umls/template-pattern.puml)

## Pseudo Code

```java
abstract class Record {
  public void save() {
    this.validate();
    this.beforeSave();
    // DB Query
  }
  abstract void validate();
  abstract void beforeSave();
}

class User extends Record {
  @Override
  public void validate() {
    // Implementation
  }

  @Override
  public void beforeSave() {
    // Implementation
  }
}

User u = new User();
u.save();
```

## Reference

- [https://youtu.be/7ocpwK9uesw](https://youtu.be/7ocpwK9uesw)
