---
title: "Design Patterns - Command Pattern"
excerpt_separator: "<!--more-->"
categories:
  - Blog
tags:
  - Design Patterns
  - Command Pattern
---

Key words:  

## UML  

### Command Pattern

![Command](http://www.plantuml.com/plantuml/proxy?src=https://raw.githubusercontent.com/battlerhythm/battlerhythm.github.io/master/assets/umls/command-pattern.puml)

## Pseudo Code

```java
class Invoker {
  ICommand on;
  ICommand off;
  ICommand up;
  ICommand down;

  public Invoker(on, off, up, down) {
    this.on = on;
    this.off = off;
    this.up = up;
    this.down = down;
  }

  public void clickOn() {
    this.on.execute()
  }

  public void clickOff() {
    this.off.execute()
  }

  public void clickUp() {
    this.up.execute()
  }

  public void clickDown() {
    this.down.execute()
  }
  ...
}

interface ICommand {
  void execute();
  void unexecute();
}

class LightOnCommand implements ICommand {
  Light light;
  
  public LightOnCommand(Light l) {
    this.light = l;
  }
}
```

```java
Receiver light = new Receiver();
Invoker remoteCtrl = new Invoker(new LightOnCommand(light),
                                 new LightOffCommand(light),
                                 new LightUpCommand(light),
                                 new LightDownCommand(light));

remoteCtrl.clickOn();
```

## Reference

- [https://youtu.be/9qA5kw8dcSU](https://youtu.be/9qA5kw8dcSU)
