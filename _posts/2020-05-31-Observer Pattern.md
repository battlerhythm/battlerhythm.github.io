---
title: "Design Patterns - Observer Pattern"
excerpt_separator: "<!--more-->"
categories:
  - Blog
tags:
  - Design Patterns
  - Observer Pattern
---

Key words: Push vs Poll

## UML  

### Better Design

![Composite](http://www.plantuml.com/plantuml/proxy?src=https://raw.githubusercontent.com/battlerhythm/battlerhythm.github.io/master/assets/umls/observer-pattern.puml)

## Pseudo Code

```java
interface IObservable {
  public void add(IObserver o);  // Register
  public void remove(IObserver o);  // Deregister
  public void notify();
}

interface IObserver {
  public void update();
}

class WeatherStation implements IObservable {
  public void add(IObserver o) {
    this.observers.add(o);
  }

  public void notify() {
    for(IObserver o : this.observers) {
      o.update();
    }
  }

  public int getTemperature() {
    return this.temperature;
  }
}

class PhoneDisplay implements IObserver {
  WeatherStation station;
  
  // Sort of dependency injection
  public void PhoneDisplay(WeatherStation station) {  
    this.station = station;
  }

  public void update() {
    // Pull data
    int temperature = this.station.getTemperature();
    // Update logic goes below
  }
}
```

```java
WeatherStation station = new WeatherStation();
PhoneDisplay display = new PhoneDisplay(station);
station.add(display);
station.notify();
```

## Reference

- [https://youtu.be/_BpmfnqjgzQ](https://youtu.be/_BpmfnqjgzQ)
