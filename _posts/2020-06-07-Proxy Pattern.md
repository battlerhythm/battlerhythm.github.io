---
title: "Design Patterns - Proxy Pattern"
excerpt_separator: "<!--more-->"
categories:
  - Blog
tags:
  - Design Patterns
  - Proxy Pattern
---

Key words: Remote, Virtual, Protection Proxy, Security, Caching  

Remote - Resource from different server, namespace, project  
Virtual - Cache, Control access to expansive methods  
Protection - Security  

## UML  

### Proxy Pattern

![Command](http://www.plantuml.com/plantuml/proxy?src=https://raw.githubusercontent.com/battlerhythm/battlerhythm.github.io/master/assets/umls/proxy-pattern.puml)

## Pseudo Code

```java
class BookParser implements IBookParser {
  public BookParser(String book) {
    // Expensive Parsing
  }

  public int getNumPages() {
    // Some Logics
  }
}

interface IBookParser {
  int getNumPages()
}

class LazyBookParserProxy {
  private Book book = null;
  private BookParser parser = null;
  
  public LazyBookParserProxy(String book) {
    this.book = book;
  }

  public int getNumPages() {
    if(parser == null) {
      this.parser = new BookParser(this.book);
    }
    return parser.getNumPages();
  }
}
```
 
## Reference

- [https://youtu.be/F1YQ7YRjttI](https://youtu.be/F1YQ7YRjttI)
