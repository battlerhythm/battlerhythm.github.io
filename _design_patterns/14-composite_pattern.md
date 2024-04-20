---
title: "Design Patterns - Composite Pattern"
permalink: /design_patterns/composite_pattern/
excerpt_separator: "<!--more-->"
toc: true
categories:
  - Design Patterns
tags:
  - Composite Pattern
---

Key words: Tree like hierarchy

## UML  

### Composite Pattern

![Composite1](http://www.plantuml.com/plantuml/proxy?src=https://raw.githubusercontent.com/battlerhythm/battlerhythm.github.io/master/assets/umls/composite-pattern1.puml)

```html
<ul>
  <li>One</li>
  <li>Two</li>
  <li>Three</li>
  <li>Fourth
    <ul>
      <li>Fourth1</li>
      <li>Fourth2</li>
    </ul>
  </ul>
</ul>
```

![Composite2](http://www.plantuml.com/plantuml/proxy?src=https://raw.githubusercontent.com/battlerhythm/battlerhythm.github.io/master/assets/umls/composite-pattern2.puml)

## Pseudo Code

```java
interface TodoList {
  String getHtml();
}

class Todo extends TodoList {
  String text;
  
  public Todo(String text) {
    this.text = text;
  }

  public String getHtml() {
    return this.text;
  }
}

class Project extends TodoList {
  String title;
  List<TodoList> todos;

  public Project(String title, List<TodoList> todos) {
    this.title = title;
    this.todos = todos;
  }

  public String getHtml() {
    String html = "<h1>";
    html += this.title;
    html += "</h1><ul>";
    for(TodoList todo : todos) {
      html += "<li>";
      html += todo.getHtml();
      html += "</li>";
    }
    html += "</ul>";

    return html;
  }
}
```

## Reference

- [https://youtu.be/EWDmWbJ4wRA](https://youtu.be/EWDmWbJ4wRA)
