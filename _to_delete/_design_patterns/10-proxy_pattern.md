---
title: "Design Patterns - Proxy Pattern"
permalink: /design_patterns/proxy_pattern/
excerpt_separator: "<!--more-->"
toc: true
categories:
  - Design Patterns
tags:
  - Proxy Pattern
---

Key words: Remote, Virtual, Protection Proxy, Security, Caching  

Remote - Resource from different server, namespace, project  
Virtual - Cache, Control access to expansive methods  
Protection - Security  

## UML  

### Proxy Pattern

![Proxy]({{ "/assets/images/umls/proxy-pattern.png" | relative_url }})

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
