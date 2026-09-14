---
title: "Reading Libraries"
order: 25
module: "M7"
module_title: "Integration"
session: "13"
intent: "A procedure for finding patterns in real source, applied to four libraries you already use."
status: "complete"
excerpt_separator: "<!--more-->"
categories:
  - Design Patterns
tags:
  - "Integration"
---

No new patterns from here on. This session is about the step where they stop being things you know and become things you see -- and the method matters more than the findings, because the method is what transfers to the next library.

<!--more-->

## First: names lie

Before reading anything, calibrate on this.

- **`RecyclerView.Adapter`** is not Adapter. You extend an abstract class and fill in hooks -- that is [Template Method]({{ "/design_patterns/m1-template-method/" | relative_url }}).
- **Retrofit's `CallAdapter`** *is* Adapter. It converts `Call<T>` into `Observable<T>`, `Deferred<T>`, or a suspend return.
- **`JdbcTemplate`** is named after Template Method and is one.
- **Compose's `Modifier`** says nothing, and is a textbook [Decorator]({{ "/design_patterns/m3-decorator/" | relative_url }}) chain.

Roughly half the time the name is right. **Read the structure, not the class name.**

## The procedure

Six passes, in this order. Each one looks for a specific shape.

1. **Start at the entry point.** The first line of the README. A `Builder`, a `create()`, a top-level function. That is the [Facade]({{ "/design_patterns/m3-facade/" | relative_url }}), and the module boundary around it usually marks what is `internal`.
2. **Read the constructor parameters.** [Strategy]({{ "/design_patterns/m1-strategy/" | relative_url }}) and [Abstract Factory]({{ "/design_patterns/m2-abstract-factory/" | relative_url }}) almost always arrive as dependencies. An interface parameter with a default implementation is a strategy nine times out of ten.
3. **Find the list that is iterated in order.** A `List<X>` walked in sequence is either [Decorator]({{ "/design_patterns/m3-decorator/" | relative_url }}) or [Chain of Responsibility]({{ "/design_patterns/m5-chain-of-responsibility/" | relative_url }}). One question separates them: **can an element decline to continue?**
4. **Find the type that contains itself.** `interface X { val children: List<X> }` is [Composite]({{ "/design_patterns/m4-composite/" | relative_url }}); one child instead of many is Decorator or Proxy.
5. **Find the enum or sealed type with transitions.** [State]({{ "/design_patterns/m1-state/" | relative_url }}) -- and check whether the transitions live in the states or in a central function.
6. **Ask what each one bought.** This is the pass people skip and the only one that teaches anything: *what would break if this were a plain function?*

## OkHttp

**The chain.** `RealCall` assembles its interceptors into a list and walks it. The order is the design, and you can read it in one place:

```
client.interceptors          // yours, outermost
RetryAndFollowUpInterceptor  // redirects, retries
BridgeInterceptor            // headers, gzip, cookies
CacheInterceptor             // may return without touching the network
ConnectInterceptor
client.networkInterceptors   // yours, inside the cache
CallServerInterceptor        // terminal
```

Apply pass 3: can an element decline to continue? `CacheInterceptor` can -- a fresh cache hit returns a response without calling `proceed()`. **So the list is Chain of Responsibility.** But `BridgeInterceptor` always proceeds and edits both directions, which makes *it* a Decorator. Same list, both patterns, decided per element.

This is also why the two public lists exist. `addInterceptor` sits outside the cache and sees logical calls; `addNetworkInterceptor` sits inside and sees actual network traffic. **A logging interceptor in the wrong list reports numbers that are quietly wrong** -- and the API split is OkHttp's answer to a problem no ordering convention could solve.

**Pass 2** turns up `Dns`, `Authenticator`, `CookieJar`, `EventListener.Factory` -- interfaces with defaults, injected through the builder. Strategies, every one.

**Pass 6**: what did the chain buy? Retries, caching, redirects and gzip are independent, testable, and reorderable -- and a user can insert their own step into a sequence the library did not anticipate. A single `execute()` function could do all of it and could not be extended from outside.

## Retrofit

**The entry point is the pattern.**

```kotlin
val api = retrofit.create(ScheduleApi::class.java)
```

There is no class implementing `ScheduleApi` anywhere. `create` calls `Proxy.newProxyInstance` and hands back an object that intercepts every method call. **A remote [Proxy]({{ "/design_patterns/m3-proxy/" | relative_url }}), built at runtime** -- and the purest instance of the pattern you will find in a library you use daily.

**Pass 3, in an unexpected place.** Retrofit holds lists of `Converter.Factory` and `CallAdapter.Factory`. To find a converter it walks the list, asking each factory, and takes the first that returns non-null.

That is **Chain of Responsibility over factories** -- a list where each element may decline. It is why factory registration order matters, and why adding a permissive converter first silently shadows the ones after it.

**Pass 6:** the dynamic proxy buys the annotation-driven API. Without it, every endpoint needs a hand-written implementation. The cost is that errors move from compile time to the first call -- which is why Retrofit validates eagerly when it can.

## Compose

Compose is the densest of the four, and three patterns are structural rather than incidental.

**The node tree is [Composite]({{ "/design_patterns/m4-composite/" | relative_url }}).** Layout nodes hold children of their own type; measurement and drawing are recursive operations over that tree.

**Snapshot state is [Observer]({{ "/design_patterns/m5-observer/" | relative_url }}) with the subscription inferred.** Reading a `State` inside a composable registers that composable as a reader; writing invalidates exactly the readers. Apply pass 6 and the gain is obvious: the hardest question in Observer -- *who is subscribed* -- is answered by the runtime instead of by you remembering to unsubscribe.

**`Modifier` is a Decorator chain.** `Modifier.padding().background()` wraps, and `foldIn`/`foldOut` are the traversal. This is the one place the ordering lesson is unavoidable, because the result is visible on screen.

Worth noting what is *not* here. There is no visitor over the node tree in the public API, no builder, and very little inheritance anywhere. **Compose is composition-over-inheritance carried further than almost any Android API** -- which is itself the second of the three principles, applied at library scale.

## Coroutines

**`CoroutineContext` is Composite, and the cleanest example in the ecosystem.**

```kotlin
public interface CoroutineContext {
    public interface Element : CoroutineContext    // a leaf is also a context
    public operator fun plus(context: CoroutineContext): CoroutineContext
    public fun <R> fold(initial: R, operation: (R, Element) -> R): R
}
```

`Element : CoroutineContext` is the pattern in one line -- leaf and composite share the interface, so `Dispatchers.IO + CoroutineName("x") + job` is a tree you can treat uniformly. `fold` is the traversal, `get` the recursive lookup.

**`Job` is [State]({{ "/design_patterns/m1-state/" | relative_url }}).** New, Active, Completing, Cancelling, Cancelled, Completed, with legal transitions enforced internally and terminal states that refuse further ones.

**`CoroutineDispatcher` is Strategy**, injected through the context.

**`Flow` operators are a Decorator chain.** Each operator returns a new `Flow` wrapping the previous one; nothing runs until `collect`. Apply pass 3: can an operator decline to pass a value downstream? `filter` can. So the chain is Decorator with elements that behave like Chain of Responsibility per value -- the same hybrid as OkHttp, one level down.

## What you should conclude

Three things, none of them "use more patterns".

**The patterns that survive in libraries are the ones that let outsiders extend the library.** Strategy, Chain of Responsibility, Decorator, Abstract Factory. All four exist because a library cannot know what its users will need. That is also why they are over-applied in application code, where you *do* know.

**The patterns that vanish are the ones that only reorganize your own code.** No Visitor, almost no Mediator, no Interpreter in any of the four. Those are shapes you reach for inside a system, not across its boundary.

**Almost every pattern here arrives as a constructor or builder parameter.** Pass 2 finds more than passes 3 through 5 combined. If you remember one heuristic from this session, make it that: **read the parameters first.**

## Exercise

Pick one library you use and have never read -- Coil, Ktor, Room, WorkManager -- and run the six passes on it. Time-box it to an hour; you are not trying to understand the library.

Write down, for each pattern you find, **what it bought.** The list of names is worth little. The list of purchases is a design education, and it is the same list you will consult when deciding whether your own code needs one.

Then keep it, because the next session is the same exercise turned on your own codebase -- in both directions.
