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



-->

---

# Mocking with Hammox

* Using a Weather app (also for the exerices)
* TODO: Introducing hammox and it's main functions

---

# Mocking with Hammox

## Setup - The mix file

```elixir {11|all}
defmodule Weather.MixProject do
  
  # ... skipping lines ...
  
  # Run "mix help deps" to learn about dependencies.
  defp deps do
    [
      {:httpoison, ">= 0.0.0"},
      {:jason, ">= 0.0.0"},
      # --- test only
      {:hammox, ">= 0.0.0", only: :test}
    ]
  end
  
  # ... skipping lines ...

end
```

---

# Mocking with Hammox

## Setup - The mix file

```elixir {9|18-19|all}
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
  
  # ... skipping lines ...

  defp elixirc_paths(:test), do: ["lib", "test/support"]
  defp elixirc_paths(_env), do: ["lib"]
end
```

---

# Mocking with Hammox

## Setup - The test

```elixir {6,8|all}
defmodule WeatherTest do
  @moduledoc false

  use ExUnit.Case, async: true

  use Hammox.Protect, module: Weather.Api, behaviour: Weather.Api.Behaviour

  import Hammox
  
  # ... missing lines ...

end
```

---

# Mocking with Hammox

## The test

```elixir {7-20|22-23|all}
defmodule WeatherTest do
  
  # ... missing lines ...
  
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

```elixir
defmodule Weather.Api.Behaviour do
  @callback get_forecast(String.t()) ::
              {:ok, map()} | {:error, reason :: term()}
end
```

## The mock

```elixir {1|all}
import Hammox

defmock(Weather.Api.Mock, for: Weather.Api.Behaviour)
```

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
