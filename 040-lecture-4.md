---
layout: image-right
image: /images/property.png
---

# Lecture 4 - Property-Based Testing

* Introduction
* [StreamData][]
* Strategies

[StreamData]: https://github.com/whatyouhide/stream_data

<!--

Welcome to Lecture 4. Almost done ...

The last lecture (Randy Pausch anybody) ...

-->

---
layout: image-right
image: /images/pascal.png
---

# Introduction

## Motivation

* Testing: `input -> function -> output`
  * 3 tests: valid, invalid, edge-case
  * Good code coverage
* In 80% of the cases unit-testing will give you 80% of the value and
  that should be good enough!
* But in some cases you can test …
  * `all(valid(input)) -> function -> output`
  * `all(invalid(input)) -> function -> output`
  * ... and check that the output is correct
  * ... and find/test the edge-cases 
* These tests are very valuable and low-maintenance
* Meet **property-based testing**!

<!--

Property-based testing ...

-->

---
layout: image-right
image: /images/pb-testing.png
---

# Introduction

## Definition, History, Concepts, ...

* `generate(input) -> property -> check(output)`
* Property-based testing is "The thing that `QuickCheck` does" :)
  * Haskell, 1999, John Hughes 
* Other implementations ...
  * Scala, Clojure,Rust, Go, ...
* Elixir ...
  * PropCheck/PropEr, StreamData :) ...
* Two main concepts ...
  * Generators 
  * Properties 
* Fun fact (2017) ...
  * `String.replace("", "", "x")`
* Example Based Testing vs. PBT

<!--

Property-based testing ...

-->

---
layout: statement
---

# Generators

<!--

Let's first talk about Generators ...

-->

---

# StreamData - Generators

* This is why `StreamData` is called `StreamData` :)
* Setup (`mix.exs`) ...
```elixir
    defp deps do
      [{:stream_data, ">= 0.0.0", only: [:dev, :test]}] 
    end
```

* Standard/Default generators ...
  * `StreamData.integer(), *.string(), *.float(), *.member_of()`
  * `StreamData.map(StreamData.integer(), &abs/1)`
  * `StreamData.list_of(StreamData.integer())`
  * ... many more

<!--

Generators create a stream of data.

-->

---

# StreamData - Generators ... (cont.)

* Composing generators with `StreamData.bind/2` ...
```elixir
domains = ["gmail.com", "yahoo.com", "icloud.com"]
random_domain_generator = StreamData.member_of(domains)
username_generator = StreamData.string(:alphanumeric, min_length: 1)

random_email_generator =
  StreamData.bind(random_domain_generator, fn domain ->
    StreamData.map(username_generator, fn username ->
      "#{username}@#{domain}"
    end)
  end)
```

* Shrinking ...
  * Finding (new) minimal edge-cases. Find least-complex value that
    makes property fail
    * All odd numbers are prime fails for 95 ... and for ... ???!
  * `StreamData.map() vs. Stream.map()`

<!--

Binding can combine Generators.

You need to use bind to make sure shrinking works (use
`StreamData.map/2` not `Stream.map/2`).

-->

---

# StreamData - Generators ... (cont.)

* Generation size ...
  * A measurement of complexity for the generated values
  * Over time values should get more complex (short simple strings ->
    long complex strings)
  * Static vs. dynamic vs. fixed generations
    * `StreamData.integer() |> StreamData.resize(50)`
    * `StreamData.integer() |> StreamData.scale(fn size -> min(size, _cap = 20) end)`
    * `StreamData.sized(fn size -> if size >= 10, do: Stream.float(), else:
...)`

<!--

Notes ...

-->

---

# StreamData - Generators ... (cont.)

* Combine it with Faker ...
  * ???
  * ???
  * ???
* The `gen_all` macro ...
  * ???
  * ???
  * ???

<!--

Notes ...

-->

---
layout: statement
---

# Properties

<!--

Notes ...

-->

---

# StreamData - Properties

```elixir {all|3|5-6|all}
defmodule FirstPropertySortTest do
  use ExUnit.Case
  use ExUnitProperties
  
  property "Enum.sort/1 sorts lists" do
    check all list <- list_of(integer()) do
      sorted_list = Enum.sort(list)
      assert length(list) == length(sorted_list)
      assert sorted?(sorted_list)
    end
  end
end
```

<!--

Notes ...

-->

---

# StreamData - Properties (cont) ...

* The `check all` macro ...
* Runs 100 times (configurable with `max_runs`)
* Comprehension that supports generators (`<-`), 
  assignments (`=`) and filters (`boolean`)
* Body is doing the checking (with assert)

```elixir
check all list <- list_of(term()),
          list != [],
          member <- member_of(list) do
  assert member in list
end
```

<!--

Notes ...

-->

---
layout: statement
---

# Strategies

## (for finding properties)

<!--

Notes ...

-->

---

# Strategies - Circular/Inverse ...

* Two functions that are the inverse of each other, means a property like ...
  * `inverse(function(value)) == value`
  * ... will always be true

```elixir
property "encoding + decoding is circular" do 
  check all term <- term() do 
    assert term == (term |> JSON.encode() |> JSON.decode()) 
  end 
end
```

<!--

Notes ...

-->

---

# Strategies - Oracle ...

* Very good/effective for migration scenarios, were you have to
  replace an old implementation with a new implementation while
  preserving the old behavior
  * `function(value) == oracle(value)`
* Different can be any of language, service, endpoint, algorithm
  (faster), ...
* Can also cover: New implementation cannot be slower than old one?

```elixir
defmodule ListSortTest do use ExUnit.Case use
ExUnitProperties

  property "quicksort/1 correctly sorts lists" do
    check all list <- list_of(term()) do
      assert ListSort.quicksort(list) == :lists.sort(list)
    end 
  end 
end 
```

<!--

Notes ...

-->

---

# Strategies - Smoke Test ...

* This is not a strategy per-se. More like a use-case
* Normally smoke-tests happen on the system level (during a deploy),
  but you could use PBT to do a unit-test low-level smoke-test by just
  calling the function 100 times with valid inputs and make sure it does
  not crash (in unexpected ways)

```elixir
check all term <- term() do 
  try do
    ListSort.quicksort(term) 
  rescue 
    FunctionClauseError -> :ok 
    other -> raise "raised unexpected exception: #{inspect(other)}" 
  else 
    term -> assert is_list(term)
  end
end
```

<!--

Notes ...

-->

---

# Strategies - Using math :) ...

* In some cases the algorithm or calculation that produces the
  result/output has certain mathematical properties that can be
  tested, e.g. ...
  * `nth_prime(n) < nth_prime(n + 1)`
* These tests can take 2 (or more) values into consideration or just
  focus a property of one value, e.g. ...
  * `(sorted_list |> last()) > (sorted_list |> hd()) # are you still awake?`
* Do not test the algorithm by re-implementing the algorithm :)

<div class="flex justify-center">
  <img src="/images/emc-square.png" class="rounded shadow" />
</div>

<!--

Notes ...

-->

---
layout: image-right
image: /images/andrea.png
---

# Summary

## Main/Key Takeaways ...

* Do not “force” using PBT. It (only) works (well) for a subset of
  problem-domains (e.g. math problems, pure functions, ...)
* Do not look at it as an alternative, but more as a way to make your
  unit-testing more comprehensive, more complete and easier to
  maintain
* Do not “reverse” implement the generators and the properties. Look
  for “real” properties

<!--

Notes ...

-->
