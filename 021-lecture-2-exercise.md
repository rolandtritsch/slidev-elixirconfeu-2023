---
layout: image-right
image: /images/weather.png
---

# Lecture 2 - Integration-Testing

## Exercise

* The `WeatherApp`
  * Find out, if it is going to rain any time soon (in a given city)
  * Using [OpenWeather][]
* Using [Hammox][]

[Hammox]: https://github.com/msz/hammox
[OpenWeather]: https://openweathermap.org

<!--

Welcome to the exerices for Lecture 2 ...

-->

---

# Checkpoints and Stretch-Goals

* Checkpoint(s) ...
  * **10 mins** - Hammox is up-and-running
  * **20 mins** - The Mocks are up-and-running
* Stretch Goal(s) ...
  * Implement the tests using cassettes ...
* Hints ...
  * Get a/the `credentials.exs` file from `Roland`
  * Put the `mocks.ex` into `test/support`
    * Note: That `test/support` was/is added in `mix.exs`
  * Test `rain?/2` not `rain?/3`
    * This will require that you use the config in `test.env`

<!--

Notes ...

We will start with a working test that is using Fakes
and will port/migrate this test to using Mocks/Hammox.

The stretch goal is to migrate/port the test to using
cassettes.

-->
