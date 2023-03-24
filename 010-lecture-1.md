---
layout: image-right
image: /images/unit-testing.png
---

# Lecture 1 - Unit-Testing

## 

* Slide bullet text
* Slide bullet text
* Slide bullet text

<!--

Welcome to Lecture 1. We will talk about ...

-->

---

# Introduction/Motivation

* To have a common understanding of the foundations of testing
  * Identifying scope of your tests through the concept the black box
  * Learning the individual parts of all tests
  * Discover patterns that lend to easier testing and cleaner code design
* To understand the importance and use of test coverage tools

<!--

What is unit-testing? Why do we need it?

-->

---
layout: image-right
image: /images/blackbox.png
---

# The blackbox of testing

* We don’t know what is happening inside
  * The boundaries (the interface) between your test and the code
under test need to be strong
  * The test should exhibit NO knowledge of code on the other side of that boundary
* What’s the boundary?
  * The test knows what information goes in: the arguments 
  * We do know what information comes out: the return values and side effects (calls from the code under test to other code)
    * Commands: e.g. writing to a database
    * Queries: e.g. reading from a database.

<!--

What is the blackbox of testing?

What is the interface for the blackbox of testing?

-->

---
layout: image-right
image: /images/schroedinger.png
---

# What's a "Unit"?

* Unit = Black Box
* Any amount of code in your application that isn’t
  end-to-end
* Typically focuses on the code in one module, but might
  be expanded to contain purely functional code in other modules
* Purely functional code is anything that will ALWAYS return the same
  value for the same input and has no side effects
  * Example: `String.to_atom/1`

<!--

What's a unit ...

-->

---
layout: image-right
image: /images/flower.png
---

# The Anatomy of a Test

* Setup
  * Constructing specific data for your input parameters
  * Staging data for side effects. Example: putting data in your database
* Exercise
  * Call the code you’re testing
* Verify
  * A fancy name for your test assertions
Teardown
  * Cleaning up any lingering impacts of your setup and exercise steps
  * Typically only needed when there were side effects
    * Example: Ecto sandbox clearing a transaction

<!--

The anatomy of a test ...

-->

---

# Isolating Code

* Only needed when there are side effects. Both commands and queries
* Most common solution is dependency injection of test doubles
  * Stubs: Used to replace queries. Tests will not fail if they aren’t called
  * Mocks: Used to replace commands. Tests will fail if they aren’t called
  * Doubles: Generic term that can mean either stubs or mocks
  
<!--

Isolating code ...

-->

---

# Dependency Injection

* Method 1: via the API
  * Might be a function or a module
  * Good: explicit and easy to understand
  * Bad: often requires a new parameter with a default value
  * We will go into more detail in this lecture
* Method 2: via the application environment
  * This is what the Mox and Hammox libraries show in their documentation
  * Good: looks clean, doesn’t require api change.
  * Bad: implicit behavior requires understanding code outside of the module
  * Will be covered in the next lecture

<!--

Dependency Injection ...

-->

---

# Code Under Test (before)

```elixir
defmodule NeedUmbrella do
 alias NeedUmbrella.WeatherAPI
 def need_umbrella_today?(city, datetime) do  
   with {:ok, response} <- WeatherAPI.get_forecast(city),
     {:ok, weather_data} <- NeedUmbrella.WeatherAPI.ResponseParser.parse_response(response) do

     NeedUmbrella.Weather.will_it_rain?(weather_data, datetime)
   end
 end
end
```

<!--

Code under test ...

-->

---

# Find the blackbox?

```elixir
defmodule NeedUmbrella do
 alias NeedUmbrella.WeatherAPI
 def need_umbrella_today?(city, datetime) do  
   with {:ok, response} <- WeatherAPI.get_forecast(city),
     {:ok, weather_data} <- NeedUmbrella.WeatherAPI.ResponseParser.parse_response(response) do

     NeedUmbrella.Weather.will_it_rain?(weather_data, datetime)
   end
 end
end
```

<!--

Find the blackbox ...

-->

---

# Outside the blackbox

```elixir {4|1-3,5-10|all}
defmodule NeedUmbrella do
 alias NeedUmbrella.WeatherAPI
 def need_umbrella_today?(city, datetime) do  
   with {:ok, response} <- WeatherAPI.get_forecast(city),
     {:ok, weather_data} <- NeedUmbrella.WeatherAPI.ResponseParser.parse_response(response) do

     NeedUmbrella.Weather.will_it_rain?(weather_data, datetime)
   end
 end
end
```

`WeatherAPI.get_forecast(city)` is a side effect, specifically a
query. We need to update the code to allow a dependency to be
injected.

<!--

Outside the blackbox ...

-->

---

# Code Under Test (after)

```elixir {3-4|all}
defmodule NeedUmbrella do
 alias NeedUmbrella.WeatherAPI
 def need_umbrella_today?(city, datetime, weather_fn \\ &WeatherAPI.get_forecast/1) do  
   with {:ok, response} <- weather_fn.(city),
     {:ok, weather_data} <- NeedUmbrella.WeatherAPI.ResponseParser.parse_response(response) do

     NeedUmbrella.Weather.will_it_rain?(weather_data, datetime)
   end
 end
end
```
Now we just need to let our test define `weather_fn/1`. This is 
injection through the function’s API.

<!--

Code under test ...

-->

---

# The Test: Setup

```elixir
defmodule NeedUmbrellaTest do
 use ExUnit.Case, async: true

  describe "will_it_rain?/2" do
    test "success: gets forecasts, returns true for imminent rain" do
      # setup
      now = DateTime.utc_now()
      future_unix = DateTime.to_unix(now) + 1
      expected_city = Enum.random(["Denver", "Los Angeles", "New York"])
      test_pid = self()

      weather_fn_double = fn city ->
	# expanded soon in a later slide
      end
```

<!--

The test setup ...

-->

---

# The Test: Exercise and Verify

```elixir
	# exercise… and verify?
      assert NeedUmbrella.need_umbrella_today?(expected_city, now, weather_fn_double) == true

      assert_received(
          {:get_forecast_called, ^expected_city},
          "get_forecast was never called"
        )
# Where is the teardown?
    end
  end
end
```

<!--

The test exercise and verify ...

-->

---
layout: two-cols
---

# The Test

```elixir {2,12|all}
    weather_fn_double = fn city ->
       send(test_pid, {:get_forecast_called, city})
       drizzle_id = 300

       data = [
         %{
           "dt" => future_unix,
           "weather" => [%{"id" => drizzle_id}]
         }
       ]

       {:ok, %{"list" => data}}
     end
```

::right::

* This is how we can assert that the test double was called, testing
  that a side effect (query) happened in our code under
  test. Therefore this double is a mock
  
* This is how we give our code under test a predictable, consistent
  return value from the side effect, isolating our code from outside
  impacts

<!--

The test ...

-->

---

# The Test

```elixir
      # kickoff… and assertion?
      assert NeedUmbrella.need_umbrella_today?(expected_city, now, weather_fn_double) == true

      assert_received(
          {:get_forecast_called, ^expected_city},
          "get_forecast was never called"
        )
      # Where is the teardown?
    end
  end
end
```

<!--

The test ...

-->

---

# Test Coverage

* bullet
* bullet
* bullet

<!--

The test ...

-->

