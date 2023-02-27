---
layout: image
image: /images/elixirconf.png
---
---
layout: intro
---

# Testing Elixir

ElixirConfEU 2023 - Lisbon/Portugal

<div class="absolute bottom-10">
  <span class="font-700">
    jeffrey@community.com / roland@community.com
  </span>
</div>

<!--
Good morning/Ola.

Welcome to Testing with Elixir.

Thanks for being here. This is going to be a great day.

My name is Jeffrey and this is Roland.

Let me walk you through the agenda ...
-->

---

# Agenda

| When | What | Who |
|---|---|---|
| 09:00 | Welcome & Introduction | Jeffrey/Roland |
| 09:30 | Unit-Testing | Jeffrey |
| 11:00 | Break | |
| 11:30 | Integration-Testing | Roland |
| 13:00 | Lunch | |
| 14:00 | Phoenix & Ecto | Jeffrey |
| 15:30 | Break | |
| 16:00 | Property-based Testing | Roland |
| 17:30 | Wrap-up/Q&A | Jeffrey/Roland |

<!--
We are first going to do a little bit of an introduction.

And then we have 4 lectures. Each lecture is 90 mins
with roughly 45 mins of slides and 45 mins of an
exercise.

We have plenty of breaks to make up for lost time or
to solve problems, if/when we encounter them.

At the end we are going to do a little but of a wrap-up
with a Q&A session. We should be done before 6pm.

The 4 lectures are ...

* Unit-Testing 
* Integration-Testing
* Testing Phoenix and Ecto
* Property-based Testing

Before we start let's do a quick round of intros. Can you
tell us (in 60 secs or less) ...

* Who you are? What you do/your company does?
* How many years of elixir do you have?
* What do you want to get out of the training?
* A little known fact?

Let's go from the back of the room to the front of the room.

Can I ask you (pointing) to start ...

Thanks. Let me introduce myself. I am Jeffrey ...
-->

---

<br><br><br>
<div class="flex justify-center">
  <img src="/images/jeffrey.png" class="rounded shadow" />
</div>

<!--

I am American. I live in Denver/Colorado.

I am doing elixir for 8 years.

I work for Community (more on this in a minute).
-->

---

<div class="flex h-full justify-center">
  <img src="/images/tdd.png" class="rounded shadow" />
</div>

<!--

And I am very passionate about testing.

--> 

---

<div class="flex justify-center">
  <a href="https://pragprog.com/titles/lmelixir/testing-elixir/">
    <img src="/images/book.png" class="rounded shadow"/>
  </a>
</div>

<!--

So much so, that I have written a book about it.

Together with Andrea Leopardi.

Todays training is based on the book.

And if you want to get the book I actually have a 
discount code that you can use to get the book for
10% off.

--> 

---
layout: fact
---

# Discount Code
10% off

---

# [Community][] - Meaningful conversations at scale!

<v-clicks>

* Text/SMS based (A2P **10DLC** standard; no short-codes; nothing to install)
* Enabling meaningful **2-way** conversations (hearing aid vs. megaphone)
* Sports & entertainment, politics, brands, … (**US only**; for now)
* 2019; ~100 employees; ~40 P&E; ~20 elixir engineers; **remote-first**
* 28 million members; 6 billion messages; 95% open rate; **59% click-through**
* ~60 **elixir** micro-services; typescript frontend; iOS app

</v-clicks>

[Community]: https://www.community.com

<!--

Now as I said Roland and myself actually work for Community.

Let me walk you through what Community is ...

Last, but not least ...

-->

---

<div class="flex h-full justify-center">
  <a href="https://youtu.be/uY_SgwM2NRI">
    <img src="/images/msne-fof.png" class="rounded shadow" />
  </a>
</div>

<!--

... if you want to learn more about our tech-stack and our 
architecture, you can watch Roland's talk from last year. 

Roland, do you want to introduce yourself ...

-->

---

<br><br>
<div class="flex h-full justify-center">
  <a href="https://tedn.life">
    <img src="/images/roland.png" class="rounded shadow" />
  </a>
</div>

<!--

Hi, I am Roland.

I am German, but I live in Dublin/Ireland (for more than 
15 years now).

I am doing elixir for 4 years.

And I am very passionate about functional programming in 
general and also about understanding how to build and run
effective, efficient, fun remote-first software-engineering 
organisations.

-->

---
layout: 3-images
imageLeft: /images/tedn-france.png
imageTopRight: /images/van.jpg
imageBottomRight: /images/working.jpg
---

<!--

So much so, that I have build myself a mobile office that
I am using to drive around Europe to work from nice places.

Let's get a little bit of house-keeping out of the way ...

-->

---

<div class="flex h-full justify-center">
  <img src="https://media.giphy.com/media/NV4cSrRYXXwfUcYnua/giphy-downsized.gif" class="rounded shadow" />
</div>

<!--

The fire exits are over there and over here.

The restrooms are down the hallway on the left.

We are doing to do/going to have lunch at 13:00. The options are ...

The wifi is ...

Extension cords are ...

-->

---

# Exercises

* 45 to 60 mins with 1-2 checkpoints (every 15 to 20 mins)
* We will ask to get certain things done by a checkpoint and if you
have not reached the checkpoint we will ask somebody who has to pair
with you for a minute to get you there
* And a stretch goal that is optional (just to make sure you do not
get bored) 
* Fork the training repo (with the instructions on how to use
it) 
* Using coveralls to measure impact

<!--

Let's get ready for the exercises ...

Next up ... very important ...

-->

---
layout: statement
---

# The Training Selfie :)

---
layout: image-right
image: /images/covid-testing.png
---

# *-Testing

* Unit-Testing
* Integration-Testing
* System-Testing
* Smoke-Testing
* Scale-Testing
* Stress-Testing
* Chaos-Testing
* ???

Anything that I missed?

What is what?

<!--

Testing is big!!!

There are lot's of different ways you can and should
test your systems. But ... keep 20/80 in mind and ...
use common sense :) ...

Here is a (probably incomplete) list of testing that 
you can do ...

Anything missing? Do you do any testing that is not
on the list here? I can thing at least of one kind 
of testing that we do at community that is not on
the list here (penetration-testing).

Let's find out what is what ...

-->

---
layout: image-right
image: /images/covid-testing.png
---

## Unit-Testing

* What ...
  * The pure/public methods of an interface/module
  * No dependencies/fully mocked
  * Testing for success and failure
  * 80% code-coverage
* Where ...
  * Runs locally on your maschine
* When/How often/How long …
  * Every time/All the time when you change the code (show emacs; two buffers)
  * Runs in 3-10 secs per module
  * (Full) Runs as part of the CI/CD pipeline

<!--

Unit-Testing ...

-->

---
layout: image-right
image: /images/covid-testing.png
---

## Integration-Testing

* What ...
  * Testing the public api of the service
    * Testing the integration of all units
  * No dependencies/fully mocked
  * Testing for success and failure
  * 80% code-coverage
* Where ...
  * Runs locally on your maschine
* When/How often/How long …
  * For every commit/push
  * Runs in 10-30 secs
  * (Full) Runs as part of the CI/CD pipeline

<!--

Integration-Testing ...

-->

---
layout: image-right
image: /images/covid-testing.png
---

## System-Testing (E2E)

* What ...
  * Testing the integration with the rest of the services
  * Using all “internal” infrastructure/services (also testing
    configuration)
  * External dependencies will be mocked (if necessary; money;
    availability; rate-limiting)
  * Testing for success
* Where ...
  * Run in a dev/staging/pre-prod env 
* When/How often/How long ...
  * For every deploy
  * Runs in 1-5 mins 
  * Runs as part of the CI/CD pipeline

<!--

System-Testing ...

... also called End-to-End Testing.

-->

---
layout: image-right
image: /images/covid-testing.png
---

## Smoke-Testing

* What ...
  * Testing a service end-to-end (happy-path)
  * Using all dependencies
  * Testing for success (of the deploy)
  * Testing your monitoring and alerting
    * The absence of an alert is success
* Where ...
  * Runs in dev and prod
* When/How often/How long …
  * For every deploy
  * (Needs to) Runs in 1-2 mins
    * Also used for/in incident-management
  * Runs as part of the CI/CD pipeline

<!--

Smoke-Testing ...

... firing a tracer-bullet down-range.

-->

---
layout: image-right
image: /images/covid-testing.png
---

## Scale-Testing

* What ...
  * Testing the throughput and response-times of services (and/or the
    entire system)
  * Using all dependencies
  * Testing for success (or the absence of regressions)
* Where ...
  * Runs in dev (for relative numbers) and prod (for absolute numbers)
* When/How often/How long ...
  * For every deploy (in dev); Every month (in prod)
  * Runs in ~10 mins (in dev; per service); Runs in ~1-2 hours (in
    prod; entire system

<!--

Scale-Testing ...

-->

---
layout: image-right
image: /images/covid-testing.png
---

## Stress-Testing

* What ...
  * Testing when the system breaks and how it will break (gradual degradation of service; beware of the cliff)
  * Using all dependencies
  * Testing for failure (increase load and length of the test until you run out of “something”)
  * Testing you monitoring and alerting
* Where ...
  * Runs in prod
* When/How often/How long ...
  * Every quarter (in prod)
  * Runs ~2 hours (in prod; entire system)

<!--

Stress-Testing ...

-->

---
layout: image-right
image: /images/covid-testing.png
---

## Chaos-Testing

* What ...
  * Testing for the absence of a single-point-of-failure
  * Randomly kill service instances and infrastructure components
    * Testing availability and redundancy
  * Using all dependencies
  * Testing for success
* Where ...
  * Runs in dev and prod
* When/How often/How long
  * Every day (in dev); Every month (in prod)
  * Runs ~2 hours

<!--

Chaos-Testing ...

-->

---
layout: statement
---

# Let's go ...

<!--

Any questions before we start?

Back to you Jeffrey ...

-->

