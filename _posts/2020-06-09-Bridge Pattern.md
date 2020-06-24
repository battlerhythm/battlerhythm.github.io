---
title: "Design Patterns - Bridge Pattern"
excerpt_separator: "<!--more-->"
categories:
  - Blog
tags:
  - Design Patterns
  - Bridge Pattern
---

Key words: Cartesian Product

S1 = {A, B}
S2 = {1, 2}
S1xS2 = {(A, 1), (A, 2), (B, 1), (B, 2)}

S1 = {A, B, C}
S2 = {1, 2, 3}
S1xS2 = {(A, 1), (A, 2), (A, 3), (B, 1), (B, 2), (B, 3), (C, 1), (C, 2), (C, 3)}

## UML  

### Bridge Pattern

![Command](http://www.plantuml.com/plantuml/proxy?src=https://raw.githubusercontent.com/battlerhythm/battlerhythm.github.io/master/assets/umls/bridge-pattern.puml)

## Pseudo Code

```java
abstract class View {
  IResource resource;

  public View(IResource r) {
    this.resource = r;
  }

  public String show();
}

class LongForm extends View{
  @Override
  public String show() {
    // ...
    this.resource.snippet()
    // ...
    return html;
  }
}

interface IResource {
  String snippet();
  String title();
  String image();
  String url();
}

class ArtistResource extends IResource {
  Artist artist;

  public ArtistResource(Artist a) {
    this.artist = a;
  }

  public String snippet() {
    return this.artist.bio();
  }

  public String title() {
    return this.artist.fName() + " " + this.artist.lName();
  }

  // ...
}

```

## Reference

- [https://youtu.be/F1YQ7YRjttI](https://youtu.be/F1YQ7YRjttI)
