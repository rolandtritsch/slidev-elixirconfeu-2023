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

We now have a good, initial understanding how we can test units
(functions/modules).

Next level up we need to make sure/verify/test, if the units work
together as expected. In a SaaS/backend/restful-api context you
are now talking about testing your service (and all it's endpoints).

For that to happen we need to find a way to deal with all the external
dependencies (other internal services, infrastructure, ...). Remember:
We need to be able to run these on you laptop. Reliably.

Luckily there are concepts like Fakes, Stubs, Mocks, ... that we will
formally introduce in the Lecture that can help us with this.

Even more there are frameworks like ByPass and Mox and Hammox that can
help us to implement these concepts.

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

Means ... the fact of live is: We have dependencies and need to find a
way to "simulate" them.

Let's talk about dependencies some more ...

-->

---
layout: image-right
image: /images/twins.png
---

# External dependencies

* Try to get away with murder (use the dependency!?!?)
  * Easy/Fast, Brittle, Expensive, ...
* Internal/External Services
  * Use them (Use a/the sandbox)
* Infrastructure (databases, caches, ...)
  * Use them (locally; containers, ...)

... or ...

* Using doubles ...

<!--

Initially this sounds bad. And if you do it wrong it can be, but
... this is why we are here: Let's do it right.

First things first: In some cases you can get away with murder ... 

-->

---
layout: image-right
image: /images/twins.png
---

# Dependency Doubles

* (API) Fakes - using FakeModules
  * Good: Simple
  * Bad: Some (manual) work
  * Ugly: Dependency injection
* Stubs - *Generating* Fakes
  * Good: Tooling available
  * Bad: Not checking number of invokations
  * Ugly: N/A
* Mocks - *Intelligent* Stubs
  * Good: Checking number of invokations
  * Bad: N/A
  * Ugly: N/A

<!--

Now let's talk about the kinds of dependency doubles we can consider.

We already know/understand what a fake is and how it works.

And there is quite a bit of scaffolding that we need to build and
implement.

For stubs and mocks this effort is greatly reduced, because there
is tooling available that will generate a lot of the code for us.

Let's take a look at this ...

-->

---

# Mocking with [Mox][]

* Mox is a library for defining concurrent mocks in Elixir
* It generates mocks based on `behaviors`
* And it articulates what these mocks are suppose to do by
  means of `expectations`
* Most people use `Mox`, but ...

[Mox]: https://github.com/dashbitco/mox 

<!--

THE tool of choice when it comes to mocking an API is Mox.

It generates the mocks based on behaviors (using @callback)
and articulates what the mocks are suppose to do by allowing
to specify so-called expectations.

We will explain in a second how all of that works.

But ...

... we can do event better.

-->

---

# Mocking with [Hammox][]

* Small Problem: Mox does not check the signature of the mocks
  * Means, if the signature changes the test might pass, but the
    application might fail when it runs
* Hammox solves this problem
  * It ensures that both your mocks and implementations fulfill the
    same contract (when/while the tests are running (we will talk
    about this/show this later))
  * To get to that goodness you just need to replace Mox with Hammox
* Hammox automatically checks that mocks stay in sync with the
  behaviours. You can also decorate the api with `Hammox.protect/2`.
  This will ensure that the implementation stays in sync with the
  behaviours
* Using a Weather app (also for the exerices)
  * Using the OpenWeather API
  * `get_forecast/2` for a city and find out, if it is going to `rain?/2`
    any time soon

[Hammox]: https://github.com/msz/hammox

<!--

Hammox is a very thin layer on top of Mox.

But it adds a good capability: It checks if the interface of the
mocks still fit the implementation.

It allows the triangle of behavior, mock and implementation to
stay in sync.

Note: Dialyzer is a static analysis tool; Hammox is a dynamic contract
test provider. They operate differently and one can catch some bugs
when the other doesn't. While it is true that Hammox would be
redundant given a strong, strict, TypeScript-like type system for
Elixir, Dialyzer is far from providing that sort of coverage.

-->

---

# Mocking with Hammox

## Setup - The mix file

```elixir {all|8|all}
defmodule Weather.MixProject do
  # ... 
  defp deps do
    [
      {:httpoison, ">= 0.0.0"},
      {:jason, ">= 0.0.0"},
      # --- test only
      {:hammox, ">= 0.0.0", only: :test}
    ]
  end
  # ...
end
```

---

# Mocking with Hammox

## Setup - The mix file

```elixir {all|9|15-16|all}
defmodule Weather.MixProject do
  use Mix.Project

  def project do
    [
      app: :weather,
      version: "0.1.0",
      elixir: "~> 1.14",
      elixirc_paths: elixirc_paths(Mix.env()),
      start_permanent: Mix.env() == :prod,
      deps: deps()
    ]
  end
  # ...
  defp elixirc_paths(:test), do: ["lib", "test/support"]
  defp elixirc_paths(_env), do: ["lib"]
end
```

---

# Mocking with Hammox

## Setup - The test

```elixir {all|6|all}
defmodule WeatherTest do
  @moduledoc false

  use ExUnit.Case, async: true

  import Hammox
  
  # ...
end
```

---

# Mocking with Hammox

## The test

```elixir {all|5-18|20-21|all}
defmodule WeatherTest do
  # ...
  describe "rain?/2 - using mocks" do
    test "success: gets forecasts, returns true for imminent rain" do
      expect(Weather.Api.Mock, :get_forecast, 1, fn city ->
        assert city == "Dublin"

        response = %{
          "list" => [
            %{
              "dt" => DateTime.to_unix(DateTime.utc_now()) + 60,
              "weather" => [%{"id" => _thunderstorm = 231}]
            }
          ]
        }

        {:ok, response}
      end)

      assert Weather.rain?("Dublin", DateTime.utc_now())
      verify!(Weather.Api.Mock)
    end
  end
end
```

---

# Mocking with Hammox

## The behavior

```elixir {all|3-4|all}
defmodule Weather.Api do
  # ...
  @callback get_forecast(String.t()) ::
          {:ok, map()} | {:error, reason :: term()}
  @spec get_forecast(String.t()) ::
          {:ok, map()} | {:error, reason :: term()}
  def get_forecast(city) do
    # ...
  end
end
```

## The mock

```elixir
import Hammox

defmock(Weather.Api.Mock, for: Weather.Api)
```

---

# Mocking with Hammox

## Breaking the contract

```elixir {all|3-4|all}
defmodule Weather.Api do
  # ...
  @callback get_forecast(String.t()) ::
          map() | {:error, reason :: term()}
  @spec get_forecast(String.t()) ::
          {:ok, map()} | {:error, reason :: term()}
  def get_forecast(city) do
    # ...
  end
end
```

## The error

```text {all|3|all}
  1) test rain?/2 - using mocks success
     test/weather_test.exs:18
     ** (Hammox.TypeMatchError) 
```

---

# Mocking with ByPass

---

# Mocking with Cassettes

---
layout: image-right
image: /images/twins.png
---

# Tools/Techniques

* Hammox
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

---
layout: image-right
image: /images/twins.png
---

# Summary

## Main/Key Takeaways ...

* Pick your battles, but in most cases you want/need to mock
  your external dependencies
* Use Hammox (until you have a reason not to use it; and then
  just use Mox), because it gives you the contract check for
  free
* ???

<!--

Notes ...

-->
