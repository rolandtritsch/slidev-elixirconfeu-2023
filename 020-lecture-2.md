---
layout: image-right
image: /images/twins.png
---

# Lecture 2 - Integration-Testing

* Testing the public interface of the service (the endpoints)
* No (???) external dependencies
  * Services & Infrastructure
* Using dependency doubles
  * Fakes, Stubs, Mocks
* Using [Hammox][]

[Hammox]: https://github.com/msz/hammox

<!--

Welcome to Lecture 2 ...

-->

---
layout: image-right
image: /images/twins.png
---

# Introduction

## Motivation

* We now understand how to test units (components/modules) in
  isolation, but ...
* Some of them will have external dependencies (e.g. 3rd party
  systems, internal systems, databases, ...)
* In some cases you can even get away with tests that use these
  dependencies, but ...
* Most of the time you need to find a way to “simulate” these
  dependencies

<!--

Notes ...

-->

---
layout: image-right
image: /images/twins.png
---

# External dependencies

* Try to get away with murder (use the dependency!?!?)
  * Easy/Fast, Brittle, Expensive, …
* Internal/External Services
  * Use them (Use a/the sandbox)
* Infrastructure (databases, caches, …)
  * Use them (locally; containers, …)

... or ...

* Using doubles ...

<!--

Notes ...

-->

---
layout: image-right
image: /images/twins.png
---

# Dependency Doubles

* (API) Fakes - using FakeModules
  * Good: Simple
  * Bad: Some work
  * Ugly: Dependency injection
* Stubs - *Generating* Fakes
  * Good: Tooling available
  * Bad: A little bit more work
  * Ugly: ???
* Mocks - ???
  * Good: ???
  * Bad: ???
  * Ugly: ???

<!--

Notes ...

-->

---

# Mocking with Hammox

---

# Mocking with Plugs

---

# Mocking with ByPass

---

# Mocking with Cassettes

---

# Tools/Techniques

* Mox
  * Good: ???
  * Bad: ???
  * Ugly: ???
* ByPass
  * Good: ???
  * Bad: ???
  * Ugly: ???
* Cassettes
  * Good: ???
  * Bad: ???
  * Ugly: ???
