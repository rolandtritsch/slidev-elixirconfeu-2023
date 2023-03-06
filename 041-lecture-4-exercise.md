---
layout: image-right
image: /images/pascal.png
---

# Lecture 4 - Property-Based Testing

## Exercise

* Pascal's Triangle
* Properties
* ???

<!--

Welcome to the exerise for Lecture 4. Almost done ...

-->

---

# Pascal's Triangle

* Introduce the Pascal’s Triangle
  * Every row has one element more than the previous row
  * Every row is a palindrome
  * The sum of all elements in a given row is double the sum of all
    elements in the previous row
  * Every row starts/ends with a 1 (and there are no other 1s in the
    row)
  * All numbers in the row are >= 1
* Instructions ...
  * Implement tests for all properties (in `pascal_test.exs`)
  * By looking for the `TODOs`
  * Find the bug in the implementation of the triangle and fix it
* Hints ...
  * If you get timeouts, try to limit your generator to integers not bigger
than 100
  * You are testing `nth/1`
  * ???

<!--

Welcome to the exerise for Lecture 4. Almost done ...

-->

---

# Checkpoints and Stretch-Goals

* Checkpoint(s) ...
  * **10 mins** - Do you have one property implemented (and is it failing)
  * **20 mins** - Have you found/fixed the bug and implemented two
    more properties
* Stretch Goal(s) ...
  * Implement a `generator` to generate integers in [0..]
  * ???
  * ???

<!--

Welcome to the exerise for Lecture 4. Almost done ...

-->
